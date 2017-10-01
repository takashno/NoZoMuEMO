---
layout: post
title: Jekyllでのブログ構築（ローカル環境構築編）
categories: [blog]
tags: [blog,jekyll]
author: alp-takashno
fullview: false
comments: true
---
{% assign author = site.data.people[page.author] %}
こんばんは、{{ author.github }}です。\\
\\
このブログ自体は[Jekyll](https://jekyllrb-ja.github.io/)というもので構築されています。\\
記事自体はMarkdownで書かれておりシンタックスハイライトももちろん対応しています。\\
普段は[Atom](https://atom.io/)で記事を記載しており、執筆環境としてはとても楽ですし続けやすいなと思います。\\
\\
一番の気に入っているのは「[GitHub Page](https://pages.github.com/)」も対応しているので、自分でビルドとかすることなく簡単に公開ができるところです。\\
ローカルもコマンド一発で立ち上がるのでGitHubへプッシュする前に最終的な見え方も簡単に確認ができます。

## ローカル環境の整備
{:toc}
Jekyllを用いたブログの記事を書いて、実際に動いている画面を見ることはローカルでも可能です。\\
そのためにRubyの環境の整備が必要となります。\\
自分が必要となったものは以下です。

- Ruby 2.4.0
- rbenv 1.1.1
- bundler 1.15.3
- nokogiri 1.8.0
- github-pages 155
\\
割と最初に考えているよりスムーズに終わらず、イライラしました。

### Mac環境でのJekyll環境の構築
自分の環境はMacであるため、Macの手順を記載する。（今Winが家にないっす。VirtualBoxでまで頑張れません）\\
そろそろ文章能力も落ちてくる頃なので、コマンドラインのメモで伝えることにします。

#### Rubyおよびrbenvのインストール
{% highlight bash %}
$ brew update # 更新しとく
$ brew install rbenv # rbenvをインストール
$ rbenv -v

rbenv 1.1.1

$ ruby -v # バージョンを確認。ここが「2.4.0」なら問題ない

ここで1.X系のRubyが出てくる…

$ rbenv install -l # インストールしたいバージョンがあるか確かめる。今回は「2.4.0」

普通にあるはず

$ rbenv install 2.4.0

ここはそんなに落ちないはず

$ rbenv global 2.4.0 # Rubyは「2.4.0」を利用するように指定する。
$ ruby -v
ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-darwin16]

{% endhighlight %}

ここまでで、うまくいくのであれば幸せかと。\\
自分はRubyのバージョンが切り替わらず、少し苦戦しました。\\
結果として、以下の対応でうまくいきました。

{% highlight bash %}
$ cat ~/.bash_profile

#!/bin/bash

# Ruby Version Controll  # 追記
eval "$(rbenv init -)"   # 追記

#THIS MUST BE AT THE END OF THE FILE FOR SDKMAN TO WORK!!!
export SDKMAN_DIR="/Users/********/.sdkman"
[[ -s "/Users/********/.sdkman/bin/sdkman-init.sh" ]] && source "/Users/********/.sdkman/bin/sdkman-init.sh"

{% endhighlight %}

#### Bundlerのインストール
単純に以下のコマンドでインストールする。
{% highlight bash %}
$ gem install bundler
$ bundler -v
Bundler version 1.15.3
{% endhighlight %}

#### nokogiriのインストール
単純に以下のコマンドでインストールする。
{% highlight bash %}
$ gem install nokogiri
$ nokogiri -v
# Nokogiri (1.8.0)
    ---
    warnings: []
    nokogiri: 1.8.0
    ruby:
      version: 2.4.0
      platform: x86_64-darwin16
      description: ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-darwin16]
      engine: ruby
    libxml:
      binding: extension
      source: system
      compiled: 2.9.4
      loaded: 2.9.43
{% endhighlight %}

#### github-pagesのインストール
単純に以下のコマンドでインストールする。
{% highlight bash %}
$ gem install github-pages
$ github-pages -v
github-pages 155
{% endhighlight %}


すげー簡単なように書いているが、いろいろエラーを踏んだ上でこの順序じゃなきゃダメだったから気をつけてください。
