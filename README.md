# AWS SAM を使ってローカルでLambdaを実行する

## 前提条件
- 本手順はMacでのみ動作確認ずみ
- brewがインストールされていること(本手順は 2.7.2にて動作確認済み)
- Docker Desktop on Mac がインストールされていること(本手順は 3.1.0 にて動作確認済み)
- AWSアカウントをもっていること
- 上記のIAMユーザーがAWSの管理者権限をもっていること(正しくは環境構築にひつような最低限の権限にするべきだが、今回は割愛)
- Node.jsの環境を構築
- プロジェクト名は仮当てで[MySamProject]とする
## SAMとは何か？
- SAM => Serverless Application Model
- Lambdaなどのサーバレスアプリケーションを開発するのに便利なコマンド・ツール
- 参考URL https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html

## 設定手順
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


## TODO
- AWS認証情報の設定:**TODO**

<img width="404" alt="スクリーンショット 2021-02-10 21 36 16" src="https://user-images.githubusercontent.com/11373018/107511121-653ccc00-6be8-11eb-9a70-f16709290271.png">

