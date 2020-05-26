---
layout: post
title: Serverless勉強
categories: [Serverless, Node.js]
tags: [Serverless, Node.js]
author: takashno
fullview: false
comments: true
img:  # Add image post (optional)
---

# Serverless学習

{% assign author = site.data.people[page.author] %}
こんばんは、{{ author.name }} です。  
今日は、Serverless Framework のチュートリアルをやってみました.

<!-- 目次 -->
<hr/>
* TOC
{:toc}
<hr/>


## 概要
ServerlessFrameworkを学んでみる.  
開発する言語はは`Node.js`.  
PaaSには`AWS`を選択してみる.

## インストール

### Node
Node.jsはもともとインストールしていたため割愛
```bash
$ node -v
v11.4.0
```

### Serverless Framework
npmでServerless Frameworkをインストール.

```bash
$ npm install -g serverless
$ serverless -v
1.35.1
$ sls -v
1.35.1
```

## プロジェクト作成

```bash
$ sls create --template aws-nodejs --path {projectName}
Serverless: Generating boilerplate...
Serverless: Generating boilerplate in "/Users/takashimanozomu/work/30_pgworkspaces/vscode/learning-serverless/out"
 _______                             __
|   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
|   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
|____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
|   |   |             The Serverless Application Framework
|       |                           serverless.com, v1.35.1
 -------'

Serverless: Successfully generated boilerplate for template: "aws-nodejs"
```

ディレクトリ構成としては以下のようなファイル群が作成された.

```
projectRoot
  |
  |--.gitignore
  |--handler.js
  `--serverless.yml
```

## ファイル編集

serverless.ymlを一部書き換える必要があるそうだ.  

```yaml
provider:
  name: aws
  runtime: nodejs8.10
  region: ap-northeast-1  # この行を追加
```

## IAMの作成

ServerlessFrameworkで利用するIAMをAWSのマネジメントコンソールから作成した.  
アクセスキーおよびシークレットキーを取得.  
ポリシーは `AdministratorAccess` を与えた.

## デプロイ

デプロイの前に、作成したIAMの情報を環境変数で定義.  

```bash
$ export AWS_ACCESS_KEY_ID={アクセスキー}
$ export AWS_SECRET_ACCESS_KEY={シークレットキー}
```

この他にも方法はあるようだ.  
[公式ドキュメント](https://serverless.com/framework/docs/providers/aws/guide/credentials/)に書いてあった.

デプロイを実施する.
```bash
$ serverless deploy -v
Serverless: Packaging service...
Serverless: Excluding development dependencies...
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
CloudFormation - CREATE_IN_PROGRESS - AWS::CloudFormation::Stack - out-dev
CloudFormation - CREATE_IN_PROGRESS - AWS::S3::Bucket - ServerlessDeploymentBucket
CloudFormation - CREATE_IN_PROGRESS - AWS::S3::Bucket - ServerlessDeploymentBucket
CloudFormation - CREATE_COMPLETE - AWS::S3::Bucket - ServerlessDeploymentBucket
CloudFormation - CREATE_COMPLETE - AWS::CloudFormation::Stack - out-dev
Serverless: Stack create finished...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading artifacts...
Serverless: Uploading service .zip file to S3 (1.27 KB)...
Serverless: Validating template...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
CloudFormation - UPDATE_IN_PROGRESS - AWS::CloudFormation::Stack - out-dev
CloudFormation - CREATE_IN_PROGRESS - AWS::Logs::LogGroup - HelloLogGroup
CloudFormation - CREATE_IN_PROGRESS - AWS::IAM::Role - IamRoleLambdaExecution
CloudFormation - CREATE_IN_PROGRESS - AWS::Logs::LogGroup - HelloLogGroup
CloudFormation - CREATE_COMPLETE - AWS::Logs::LogGroup - HelloLogGroup
CloudFormation - CREATE_IN_PROGRESS - AWS::IAM::Role - IamRoleLambdaExecution
CloudFormation - CREATE_COMPLETE - AWS::IAM::Role - IamRoleLambdaExecution
CloudFormation - CREATE_IN_PROGRESS - AWS::Lambda::Function - HelloLambdaFunction
CloudFormation - CREATE_IN_PROGRESS - AWS::Lambda::Function - HelloLambdaFunction
CloudFormation - CREATE_COMPLETE - AWS::Lambda::Function - HelloLambdaFunction
CloudFormation - CREATE_IN_PROGRESS - AWS::Lambda::Version - HelloLambdaVersionB7WY0OSKPIHFjeHZtSsqgSXLgHFcfJalbqn07jxo8
CloudFormation - CREATE_IN_PROGRESS - AWS::Lambda::Version - HelloLambdaVersionB7WY0OSKPIHFjeHZtSsqgSXLgHFcfJalbqn07jxo8
CloudFormation - CREATE_COMPLETE - AWS::Lambda::Version - HelloLambdaVersionB7WY0OSKPIHFjeHZtSsqgSXLgHFcfJalbqn07jxo8
CloudFormation - UPDATE_COMPLETE_CLEANUP_IN_PROGRESS - AWS::CloudFormation::Stack - out-dev
CloudFormation - UPDATE_COMPLETE - AWS::CloudFormation::Stack - out-dev
Serverless: Stack update finished...
Service Information
service: out
stage: dev
region: ap-northeast-1
stack: out-dev
api keys:
  None
endpoints:
  None
functions:
  hello: out-dev-hello
layers:
  None

Stack Outputs
HelloLambdaFunctionQualifiedArn: arn:aws:lambda:ap-northeast-1:************:function:out-dev-hello:1
ServerlessDeploymentBucketName: out-dev-serverlessdeploymentbucket-************
```

よくわからないが、デプロイは完了したような雰囲気だ.  
雰囲気、LamndaとS3に何かしたのだろうというのは標準出力からわかる.  
とりあえず、マネジメントコンソールで何が起きたのかは後ほど確認してみる.  

チュートリアルに存在する、以下のコマンドは簡単に更新するためのもの.

```bash
$ serverless deploy function -f hello
Serverless: Packaging function: hello...
Serverless: Excluding development dependencies...
Serverless: Uploading function: hello (476.58 KB)...
Serverless: Successfully deployed function: hello
Serverless: Successfully updated function: hello
```

## デプロイ結果

### Lambda

Lambda Functionが作成されていた.  
内容は流石に、`handler.js` のようだ.

![Lambda]({{ site.url }}/assets/resources/images/2019-01-06-serverless-framework-01/lambda001.png)

ただし、トリガーが定義されていない.  
`API Gateway` 等の設定を行っていないからだろうが、どうやって実行するのか.  

![Lambda]({{ site.url }}/assets/resources/images/2019-01-06-serverless-framework-01/lambda002.png)

この他、S3にバケットが新たに作成されており、CloudFormationも作成されていた.  


## 実行

実行もServerless Frameworkから行えるようだ.

```
$ serverless invoke -f hello -l
{
    "statusCode": 200,
    "body": "{\"message\":\"Go Serverless v1.0! Your function executed successfully!\",\"input\":{}}"
}
--------------------------------------------------------------------
START RequestId: d061003a-11c8-11e9-bc43-c9296002e575 Version: $LATEST
END RequestId: d061003a-11c8-11e9-bc43-c9296002e575
REPORT RequestId: d061003a-11c8-11e9-bc43-c9296002e575  Duration: 5.34 ms       Billed Duration: 100 ms         Memory Size: 1024 MB    Max Memory Used: 45 MB

```

実行してみたところ正常に返却されているようだ.  
ただし、AWSのマネジメントコンソールからみても１度実行されてる以外よくわからない.  

![Lambda]({{ site.url }}/assets/resources/images/2019-01-06-serverless-framework-01/lambda003.png)

## ログ確認

ログ確認用のコマンドがあるようなので、試してみる.  
よくわからんが、めちゃくちゃコマンドが返ってくるまでに時間がかかりすぎるので諦めた・・・


## 削除

デプロイしたものを削除するコマンドがあるようだ.  
無料枠で頑張る自分は忘れず削除・・・  

```
$ serverless remove
Serverless: Getting all objects in S3 bucket...
Serverless: Removing objects in S3 bucket...
Serverless: Removing Stack...
Serverless: Checking Stack removal progress...
....
Serverless: Stack removal finished...
```

AWSのマネジメントコンソールからも確認を行なったが、  
作成されたものが綺麗さっぱり掃除されていた.  


## まとめ

Serverless Frameworkのチュートリアル的なものをなぞってみたが、  
ハマりどころがなくスムーズにことが進むので驚いた.  
Serverless Architectureはこれから無視できない技術領域になるため、  
もう少ししっかりとしたキャッチアップを進めていきたいと思う.  