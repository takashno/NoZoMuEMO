---
layout: post
title: SpringMVCのController,Service,Repositoryについて
categories: [blog]
tags: [blog]
author: takahashi0813
fullview: false
comments: true
---
{% assign author = site.data.people[page.author] %}
こんばんは、{{ author.github }}です。\\
SpringMVCのController,Service,Repositoryとそれぞれの繋がりについて教えてもらったので書いておきます。

![2017-08-27blog](assets/media/Untitled Diagram.png)

たとえば入社年と連番を入力すると社員番号を作成し、その社員番号の人の名前、社員番号、部署名などを表示させる
社員検索システムがあったとします。\\
入社年、連番を入力し決定ボタンが押され、リクエストが送られます。\\
まずそのリクエストを受け取るのがControllerです。\\
Controllerではリクエストを受け取る以外にも、連番で数字が入力されているかなどの入力チェックも行います。\\
そして、Controllerはドメイン層のService（ビジネスロジック）を呼び出します。\\
Serviceでは業務処理のみ行います。\\
業務処理とはここでいう、入社年と連番から社員番号を作成する処理です。（例：入社年2017年　連番63番＝社員番号217063）\\
Serviceが社員番号を作成したら、社員番号を元にデータベースから社員の名前、部署などの情報を持ってくる必要があります。\\
それを行うのがRepositoryです。\\
Repositoryは情報を持ってくるだけではなく作成、編集、削除などのCURD処理も行うことができます。\\
\\
下にそれぞれの役割をまとめておきます。\\
\\
## Controllerの役割
リクエストを受け取る\\
入力チェック\\
ドメイン層のService（ビジネスロジック）の呼び出し\\

### Controllerの入力チェック
入力チェックを厳しくしすぎるとServiceの実装は楽だがユーザーにとって使いにくいものとなる可能性がある。\\
入力チェックを甘くするとその逆。\\
という問題がある。

## Serviceの役割
業務処理のみ\

### ServiceとRepositoryを分ける理由
一緒でもできる？が、コードが長くなる、DBの変更があった場合全て修正しなければならない。\\
テスト時に、一番みたいところはビジネスロジック部分なのでDBアクセスする部分は邪魔になる。\\
なので、DBアクセスはRepositoryにまとめておく。

## Repositoryの役割
DBアクセス（作成、更新、読み込み、削除といったCRUD処理）
