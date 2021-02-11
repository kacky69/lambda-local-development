## 前提条件
- 本手順はMacでのみ動作確認ずみ
- brewがインストールされていること(本手順は 2.7.2にて動作確認済み)
- Docker Desktop on Mac がインストールされていること(本手順は 3.1.0 にて動作確認済み)
- AWSアカウントをもっていること
- 上記のIAMユーザーがAWSの管理者権限をもっていること(正しくは環境構築にひつような最低限の権限にするべきだが、今回は割愛)
- `~/.aws/credentials`と`~/.aws/config`が正しく設定されていること
- Node.jsの環境を構築
- プロジェクト名は仮当てで[MySamProject]とする

## SAMとは何か？
- SAM => Serverless Application Model
- Lambdaなどのサーバレスアプリケーションを開発するのに便利なコマンド・ツール


## 参考URL
  - https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html
  - https://dev.classmethod.jp/articles/lambda-container-pkgfmt-with-sam-cli/

## 1. SAMのインストールとプロジェクトの作成
- AWS SAM CLIをホスト上にインストールし

  ```
  $ brew tap aws/tap
  $ brew install aws-sam-cli
  $ sam --version # インストールされたことと、バージョンを確認
  ```

- SAMプロジェクトの作成

- プロジェクトを作成したい任意のフォルダに移動する
    ```
    $ cd ~/Documents/development
    ```
    
  - samプロジェクトの作成
    ```
    $ sam init
    ```
    
  - sam init での質問に対して以下のように回答
    ```
    Which template source would you like to use?
      1 - AWS Quick Start Templates
      2 - Custom Template Location
    Choice: 1

    What package type would you like to use?
      1 - Zip (artifact is a zip uploaded to S3)
      2 - Image (artifact is an image uploaded to an ECR image repository)
    Package type: 2
    
    Which base image would you like to use?
      1 - amazon/nodejs14.x-base
      2 - amazon/nodejs12.x-base
      3 - amazon/nodejs10.x-base
      4 - amazon/python3.8-base
      5 - amazon/python3.7-base
      6 - amazon/python3.6-base
      7 - amazon/python2.7-base
      8 - amazon/ruby2.7-base
      9 - amazon/ruby2.5-base
      10 - amazon/go1.x-base
      11 - amazon/java11-base
      12 - amazon/java8.al2-base
      13 - amazon/java8-base
      14 - amazon/dotnet5.0-base
      15 - amazon/dotnetcore3.1-base
      16 - amazon/dotnetcore2.1-base
    Base image: 1
    
    Project name [sam-app]: MySamProject
    ```
    
  - MySamProject が作成されていることを確認
    ```
    $ ls -al
    ```

  - MySamProject ディレクトリに移動し、フォルダ構成を確認
    ```
    $ cd MySamProject
    $ tree
    ```

## 2. 主要なディレクトリ・ファイル

![image.png](https://image.docbase.io/uploads/d2ea987d-a94a-4d8a-9ca1-c0e18403cdd6.png =WxH)

- README.md
    - 有益な情報が記載されているので、目を通しておいたほうがいい
    - 全体像を把握してから見たほうが分かりやすいので、本手順を試したあとに読むといいかも

- [hello_world]ディレクトリ
  - ソース(今回の場合は`app.js`)やpackage.jsonやDockerfileを配置する主要ディレクトリ
  - このディレクトリにて開発を行う
  - 各ファイルについての知識がある前提で詳細は割愛

- [template.yaml]
  - CloudFormation の設定ファイル
  - 詳細は割愛

## 3. コンテナイメージのビルド

- 以下のコマンドにてコンテナのイメージを作成する
  ```
  $ sam build
  ```

- イメージが作成されたことを確認する
  ```
  $ docker images
  ```

## 4. ローカルで動作確認を行う

### パターン1. Dockerを使ってローカルにAPI Gatewayを立ち上げてLamdbaを呼び出す方法

- 以下のコマンドで擬似的な API Gateway を立ち上げる
  ```
  $ sam local start-api
  ```
  > [http://127.0.0.1:3000/] にて起動する
  
  > デフォルトでは [http://127.0.0.1:3000/hello] を Lambda へマッピングする
  
  > どのパスをどのLambda関数にマッピングするかは[template.yaml]の[Resources]にて指定する
  
  > マッピングを変えた場合は[コンテナイメージのビルド]をやり直す必要がある(ハマりポイント！)

- Lambdaの呼び出し
  - curlを利用する `curl http://127.0.0.1:3000/hello`
      - 結果は標準出力に表示される
  - ブラウザから呼び出す
    - ブラウザから [http://127.0.0.1:3000/hello] にアクセスする
    - 結果はブラウザに表示される
- Lambdaへのパラメータの渡し方
  - URLにQueryStringをつけて呼び出す
      - 例) [http://127.0.0.1:3000/hello?hoge=fuga]
  - POSTなどで値を渡す
      - 例) 検証用のフォームがあるHTMLなどを作成しておいて呼び出す

- 擬似的な API Gateway の停止
  - [Ctr+C]で止める

### パターン2. json形式で用意したテストデータでLamdbaを呼び出して検証する用法

- パターン1の`$ sam local start-api`コマンド実行は不要
-  以下のコマンドで呼び出す
-  結果は標準出力に表示される
-  詳細は[このあたり](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-invoke.html)を参考に。

    ```
    sam local invoke --event [json形式で用意したテストデータのPATH]
    ```
> 画像のリサイズなどのブラウザで確認する必要がある処理はパターン1があいそう。jsonを返すAPIや内部データを変更するような処理やS3に画像を吐き出すような処理はパターン２があいそう

## 5. AWSにLamdbaをデプロイする

### アーキテクチャ
    - 以下の順番で動作する
        - SAMで作成されたイメージをECRリポジトリにアップロードする
        - 上記が自動的にLamdbaへ反映される

### 手順
- ECRリポジトリを準備する
    - AWSコンソール上で上記のイメージ用のECRリポジトリを作成する
    - 作成するリージョンをしっかり確認しておくこと！
- samコマンドを使ってERRリポジトリにアップロードする

    ```
    sam deploy --guided
    ```
- sam deploy --guided での質問に対して以下のように回答

    ```
    Stack Name [sam-app-nodejs14]: sam-app-nodejs14
    
    AWS Region [ap-northeast-1]: ap-northeast-1
    
    # 以下はERCレポジトリのURIを入力
    Image Repository for HelloWorldFunction: XXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/hello_world_function_repo

    Confirm changes before deploy [Y/n]: y
    
    Allow SAM CLI IAM role creation [Y/n]: y
    
    HelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
    
    Save arguments to configuration file [Y/n]: y
    
    SAM configuration file [samconfig.toml]: samconfig.toml
    
    SAM configuration environment [default]: default
    
    # ここで1分くらいかかる
    
    Deploy this changeset? [y/N]: y
    ```

- sam deploy --guided を実施すると`samconfig.toml`ファイルが作成される。ここにECRリポジトリの情報などが保存されるため、次回以降は`sam deploy` コマンドのみでデプロイ出来るようになる。保存されている情報を変更したい場合は、再度[--guided] オプションをつけて実施する。

- AWSコンソール上でLambdaの画面を開き、デプロイされたことを確認する

## 6. 開発サイクル
- ローカルでソースを修正
- `sam build`で上記修正を反映したイメージを作成
- [ローカルで動作確認を行う]の手順で動作確認
- [AWSにLamdbaをデプロイする]の手順でデプロイ

## メモとかTODO

- 以下、正しいかどうかは不明です。(どこかのタイミングで調べたい or 詳しい方、教えて下さい！)
- `$ sam build`にてイメージが作成されると同時に、コンテナのステータスが[created]のままになっている。(`docker ps -a`で確認できる) おそらく、実行時に[up]に変わり、処理が終了すると[created]に戻っている？こうすることで、リソースの消費を抑えつつ、必要なときに素早く起動するようにしているのかなぁ？
- Lamdbaのデプロイ方法は、今回記載しているコンテナ方式とzip方式の２種類がある。zip方式の方が起動までの時間が短いという情報をよく見かけるので、本番にデプロイするときはzipにしてアップロードするシェルをつくるなどした方がよいかも？
- `$ sam local start-api`にてローカルAPI Gatewayを立てて動作確認する方式では、ソースを修正するとhot reloadされるらしい([参照](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-start-api.html))が、うまく行かない。ソースを修正するたびに` $ sam build`を実行すれば反映されるが、めんどくさい。ここは解決したい。
- 文字ばっかりでアーキテクチャが伝わりづらいので、図解したい！
