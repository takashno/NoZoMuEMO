---
layout: post
title: Jekyllでのブログの書き方
categories: [blog]
tags: [blog,jekyll]
author: alp-takashno
fullview: false
comments: true
img:  # Add image post (optional)
---
{% assign author = site.data.people[page.author] %}
こんばんは、{{ author.name }}です。\\
少しきちんとJekyllのブログの書き方を紹介したいと思います。

<!-- 目次 -->
<hr/>
* TOC
{:toc}
<hr/>

## 投稿の書き方
JekyllはMarkdown記法による記述を行えば、HTML変換するときに綺麗に出力してくれるので、\\
小さい労力で綺麗なページが書けます。このあたりはGitHubのREADME.mdなんかと同じですね。\\
\\
ですが、Markdownにも割と派生がありまして選択するMarkdonwのエンジンによって少し記法が変わっていたりもするようです。\\
またAsciiDoc等のドキュメントビルダーとも少し違うので覚えるのが少しだるいかもしれません。。。\\
この辺りは世界的に１つにまとまるととても嬉しいですが、そんなことはありえません。

### Jekyllで利用するMarkdownのエンジン
_config.ymlで定義を行います。
例は以下の通り。
{% highlight yaml %}
markdown: kramdown
{% endhighlight %}

このブログで採用しているMarkdown記法は「kramdown」というもので特に思い入れもなく使ってます。\\
詳細に関しては以下を参照してください。

公式ページ\\
[https://kramdown.gettalong.org/](https://kramdown.gettalong.org/)\\
\\
クイックリファレンス（案外見辛い）\\
[https://kramdown.gettalong.org/quickref.html](https://kramdown.gettalong.org/quickref.html)\\

### 下書きから公開まで
Jekyllの公式ページをみると書いてあるんですが、投稿の下書きと公開をきちんと管理できます。\\
Jekyllのディレクトリ構成は以下のようになっているのですが（公式ページリンクのみ）\\
\\
[Directory structure](https://jekyllrb.com/docs/structure/)\\
\\
プロジェクト直下に、「_posts」と「_drafts」というディレクトリがあります。\\
単純ではありますが、「_posts」が公開投稿となり、「_drafts」が下書きの投稿となります。\\
なのでまだ公開したくないが書き溜めて置きたいんだ！といった内容は「_drafts」に配置すれば良いわけです。

#### 下書きのプレビュー方法
次にきになるのが当然ですが、下書きのプレビューです。\\
ローカルで簡単に起動して投稿の確認ができるJekyllですが、どうしたら下書きの確認ができるかを説明します。\\
\\
Jekyllを普通にローカルで起動するには以下のコマンドを実行します。\\
起動後に[localhost:4000](http://localhost:4000)へアクセスしてみてください。
{% highlight bash %}
$ jekyll s
{% endhighlight %}

Jekyllを下書きモードでローカルで起動するには「--draft」オプションを追加して以下のコマンドを実行します。\\
起動後に[localhost:4000](http://localhost:4000)へアクセスしてみてください。
{% highlight bash %}
$ jekyll s --draft
{% endhighlight %}

こうすることで下書きディレクトリ配下に存在する投稿も全てローカルで確認することができます。

#### 公開する方法
単純ですが、「_posts」ディレクトリへ移動するのみです。

#### 下書きから公開するまでの運用？文化
複数人でブログを運営する場合で承認が必要なようなお堅いブログであれば、\\
GitHubのブランチ運用を行なってプルリクエストで運用すべきでしょうから「_drafts」ディレクトリというよりは\\
ブランチを切って「_posts」ディレクトリのみを更新対象とする方がよいでしょう。\\
\\
このブログは適当でいいのでmasterブランチをそのまま直接いじってます。。。\\
masterを直接いじるのであれば割と「_drafts」ディレクトリは有効利用できるのではないでしょうか。

### 画像の埋め込み
画像の埋め込み方法はいくつかありますが、決めの問題だと思いますので自分なりに考えた方法を説明します。\\
公式ページには、「assets」とか「downloads」とか勝手に使えばいいんじゃない？という感じでしたが、\\
以下のディレクトリを使おうと思います。
```
{projectDir}/assets/resoureces/images/{投稿ページの拡張子を抜いたもの}/{任意のファイル}
```
参照するときには、以下のように記載します。
{% highlight md %}
![alt属性]({{"{{ site.url " }}}}/assets/resoureces/images/{投稿ページの拡張子を抜いたもの}/{ファイル名}.jpg){:height="(縦)px" width="(横)px"}
{% endhighlight %}
実際に使ってみるとこんな感じです。\\
\\
![takashno]({{ site.url }}/assets/resources/images/2017-08-23-howto-writepage/takashno.jpg){:height="200px" width="160px"}

### テンプレート
下書きページに、テンプレートを入れておいてみんなで育てて使いやすくしていくのがいいかもしれません。\\
そうすることで誰もが手っ取り早く馴染むことができて更新頻度も上がるかもしれません。\\
\\
\\
以上でした。
