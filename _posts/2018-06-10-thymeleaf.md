---
layout: post
title: Thymeleafを画面テンプレートエンジンとして利用する①
categories: [Thymeleaf,Spring]
tags: [Spring,SpringBoot,Thymeleaf]
author: takashno
fullview: false
comments: true
---

[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)とは、JavaのテンプレートエンジンでXML/XHTML/HTML5からアプリケーションのデータを作成できるものですが、\\
主な用途は画面（HTML）のテンプレートエンジンでしょう。\\
[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)は公式の日本語ドキュメントが非常に充実しているため、あえて説明することもあまりありません。\\
このブログでは紹介や自分自身の考え方のメモとして記事にすることにしました。

<!-- 目次 -->
<hr/>
* TOC
{:toc}
<hr/>

### デザインとロジックの共存が可能なテンプレート
[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)の強みはHTMLとの相性がとても良いことです。\\
また、デザイナーさんとの共同作業に強いことです。\\
JSPなどは、デザイナーさんがモックアップを作成してそれをエンジニアがJSPへ移行する等の作業が入りますが、\\
[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)を利用するとその手間が少しなくなります。\\
また、画面に対する仕様変更が入った際に一番効果的ですがエンジニアが作成したThymeleafのテンプレートを\\
そのままデザイナーさんへ引き継ぐことが可能です。\\
どういうことかというと、[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)のテンプレートを見ながら説明します。


```html
<table>
  <tr>
    <th>NAME</th>
    <th>PRICE</th>
    <th>IN STOCK</th>
    <th>COMMENTS</th>
  </tr>
  <tr th:each="prod : ${prods}" th:class="${prodStat.odd}? 'odd'"> <!-- (1) -->
    <td th:text="${prod.name}">Onions</td> <!-- (2) -->
    <td th:text="${prod.price}">2.41</td>
    <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    <td>
      <span th:text="${#lists.size(prod.comments)}">2</span> comment/s
      <a href="comments.html" 
         th:href="@{/product/comments(prodId=${prod.id})}" 
         th:unless="${#lists.isEmpty(prod.comments)}">view</a>
    </td>
  </tr>
  <tr class="odd" th:remove="all"> <!-- (3) -->
    <td>Blue Lettuce</td>
    <td>9.55</td>
    <td>no</td>
    <td>
      <span>0</span> comment/s
    </td>
  </tr>
  <tr th:remove="all">
    <td>Mild Cinnamon</td>
    <td>1.99</td>
    <td>yes</td>
    <td>
      <span>3</span> comment/s
      <a href="comments.html">view</a>
    </td>
  </tr>
</table>

```

1. [Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)はテンプレートエンジンとしての処理指示をHTMLの属性で記述することが可能です。\\
これは単純なHTMLとしてブラウザ表示した場合には無視されます。\\
thというネームスペースがThymeleaf用の属性を定義するために存在します。\\
これは別途htmlエレメントに定義する必要があります。

1. th:textはいわゆるDOMのテキストノードを置換するための記述です。\\
ブラウザで見たときには「Onions」というHTMLに記述されている値が採用されますが、\\
[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)のテンプレートエンジンを利用してコンバートした後のHTMLとしては\\
`th:text="${prod.name}"`として動的に解決した値が採用されます。

1. モックアップの時点ではTableタグの行（trタグ）として複数のものが存在していないと\\
お客様にイメージをきちんと把握してもらえないためダミーの行を作成するでしょう。\\
ですがテンプレートとしては不要です。\\
JSP等と一緒で１つの行に対してfor文のような繰り返し処理を行って画面を作成するからです。\\
[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)ではこれらの`モックアップには必要だがテンプレートエンジンには不要`といったHTMLの要素を消し込む機能があります。
それが`th:remove`です。


### システムインテグレータの内部としても利用する価値がある
前述では、あくまでもデザイナーさんとの共同開発を主な目線として記載していますが、\\
システムインテグレータ＝エンジニアのみで利用する場合も有効であると自分は考えています。\\
\\
基本的にウォーターフォール開発をする場合には、工程が要件定義→基本設計→詳細設計のように進んでいくことが多いでしょう。\\
画面モックアップが活躍するのは、要件定義〜基本設計だと思います。\\
このモックアップをたとえエンジニアが作成していても後々の製造工程等において\\
そのまま活用するために[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)を利用するのはアリだと思います。\\
\\
仕様変更がないシステム開発は効率的で理想的ですが、現実的に多くはないでしょう。（あるのかな・・・）\\
その際に仕様変更が画面デザインも含めて変わるような内容の場合はお客様に確認する術として\\
視覚に訴えかけることが必要となるでしょう。\\
\\
そのような時に、[Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)であれば別途モックアップの最新化やメンテナンス作業を別途行うことなく\\
モックアップ修正＝製造工程修正となるため効率が良いです。\\
（このような利用をするためには、元からそういうつもりでテンプレートを組まなければなりませんので少し注意）


### 今回のまとめ
- [Thymeleaf](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)の紹介
- ちょっとしたテンプレートの紹介
- 実際の開発での利用についての自論

### 参考
[Thymeleaf公式ドキュメント](https://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf_ja.html)