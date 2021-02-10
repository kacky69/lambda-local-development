# AWS SAM を使ってローカルでLambdaを実行する

## 前提条件
- 本手順はMacでのみ動作確認ずみ
- brewがインストールされていること(本手順は 2.7.2にて動作確認済み)
- Docker Desktop on Mac がインストールされていること(本手順は 3.1.0 にて動作確認済み)
- AWSアカウントをもっていること
- 上記のIAMユーザーがAWSの管理者権限をもっていること(正しくは環境構築にひつような最低限の権限にするべきだが、今回は割愛)
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
- プロジェクトの作成
  - 任意のフォルダを作成する
    ```
    $ mkdir ~/Documents/development/sam-dev
    ```
  - samプロジェクトの作成
    ```
    $ cd ~/Documents/development/sam-dev
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
    ```


## TODO
- AWS認証情報の設定:**TODO**

<img width="404" alt="スクリーンショット 2021-02-10 21 36 16" src="https://user-images.githubusercontent.com/11373018/107511121-653ccc00-6be8-11eb-9a70-f16709290271.png">

