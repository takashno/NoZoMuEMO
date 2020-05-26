---
layout: post
title: REST-APIにおける条件付き処理
categories: [tech]
tags: [SpringMVC,REST-API]
author: takashno
fullview: false
comments: true
img:  # Add image post (optional)
fig-caption: # Add figcaption (optional)
---

{% assign author = site.data.people[page.author] %}
こんばんは、{{ author.name }}です。\\
\\
SpringBoot/SpringMVCを利用してREST-APIを組んでいますが、\\
REST-APIアプリケーションの排他制御を考えた際に、極力サーバーサイドアプリケーションまたはデータベースサーバーの負荷をかけない方法を考えたいと思いました。
RESTにおけるリソースの条件付き処理（取得/更新）の方法について調べましたのでメモとして残します。\\
\\
REST-APIのみならずHTTP（HTTP1.1）には元々ブラウザキャッシュを有効に利用するために、
現在見ているコンテンツとサーバー側に存在するコンテンツの新旧比較を行い制御する方法が策定されています。\\
具体的には以下のHTTPヘッダとなります。

|ヘッダ|Request / Response|用途|利用されるメソッド|
|:---|:--|:---|:---|
|If-Match|Request|コンテンツのバージョンが指定した値と`合致している場合`に処理を行うことを指定するヘッダ。|GET/HEAD
|If-None-Match|Request|コンテンツのバージョンが指定した値と`合致していない場合`に処理を行うことを指定するヘッダ。|
|ETag|Response|コンテンツに対するバージョンを通知するためのヘッダ。|GET/HEAD

上記はサーバーサイドにコンテンツのバージョンを示す値を保有している場合に、
リクエストでその値と合致するか合致しないかで振る舞いを変えたい場合に利用する。

|ヘッダ|Request / Response|用途|利用されるメソッド|
|:---|:--|:---|:---|
|If-Modified-Since|Request|コンテンツの最終更新日時が指定した値と`合致していない場合`に処理を行うことを指定するヘッダ。|GET/HEAD
|If-Unmodified-Since|Request|コンテンツの最終更新日時が指定した値と`合致している場合`に処理を行うことを指定するヘッダ。|POST/PUT/DELETE
|Last-Modified|Response|コンテンツに対する最終更新日時を通知するためのヘッダ。|GET/HEAD POST/PUT/DELETE

上記はサーバーサイドにコンテンツの最終更新日時を示す値を保有している場合に、
リクエストで指定されるヘッダの値と比較し、合致するか合致しないかで振る舞いを変えたい場合に利用する。

\\
ここまでは調査内容です。\\
SpringMVCでは [WebRequest](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/request/WebRequest.html)
というインタエースが用意されており、\\
この実装クラスで`checkNotModified(String etag, long lastModifiedTimestamp)`あたりのメソッドを利用すると上記のヘッダを利用した処理実行制御を実現することができました。\\
実際に利用したクラスは [ServletWebRequest](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/request/ServletWebRequest.html)
になります。\\
\\
まずは実装方法から、GET(HEAD) / POST / PUT / DELETE で試してみました。
```java
@GetMapping("{id}")
public ResponseEntity<Sample> getCheckModifiedEtag(WebRequest webRequest, @PathVariable("id") String id) throws ParseException {

    // XXX:本来業務処理で取得
    String etag = "1";

    // XXX:本来業務処理で取得
    SimpleDateFormat sdf = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss zzz", Locale.US);
    sdf.setTimeZone(GMT);
    long lastModified = sdf.parse("Sun, 01 Oct 2017 11:11:11 GMT").getTime();

    // ここでチェックを行う
    if (webRequest.checkNotModified(etag,lastModified)) {
        return null;
    }

    // 本来行うべき業務処理
    return new ResponseEntity<Sample>(
            Sample.builder().id(id).message("sample").build()
            , HttpStatus.OK);
}

```
メソッドが異なるが、同じ実装で試しているためMappingアノテーションのみの差分となります。\\
実際に試した結果は文章よりも表で示した方がわかりやすいので表にします。

|メソッド|ヘッダ|リクエストヘッダ値|実態値|ステータスコード|レスポンスヘッダ|
|:---|:-|:-|:-|:-|:---|
|GET|If-Match|0|1|200|ETag & Last-Modified
|GET|If-Match|1|1|200|ETag & Last-Modified
|GET|If-Match|2|1|200|ETag & Last-Modified
|GET|If-None-Match|0|1|200|ETag & Last-Modified
|GET|If-None-Match|1|1|304|ETag & Last-Modified
|GET|If-None-Match|2|1|200|ETag & Last-Modified
|POST/PUT/DELETE|If-Match|0|1|200|-
|POST/PUT/DELETE|If-Match|1|1|200|-
|POST/PUT/DELETE|If-Match|2|1|200|-
|POST/PUT/DELETE|If-None-Match|0|1|200|-
|POST/PUT/DELETE|If-None-Match|1|1|412|-
|POST/PUT/DELETE|If-None-Match|2|1|200|-

結果として、GETメソッド時の「If-None-Match」のみ有効に利用できそうです。\\
ソースコードまで追ったところ、「If-Match」に関する処理は実装されていませんでした。\\
次にModified系の検証結果になります。

|メソッド|ヘッダ|リクエストヘッダ値|実態値|ステータスコード|レスポンスヘッダ|
|:---|:-|:-|:-|:-|:---|
|GET|If-Modified-Since|0|1|200|ETag & Last-Modified
|GET|If-Modified-Since|1|1|304|ETag & Last-Modified
|GET|If-Modified-Since|2|1|304|ETag & Last-Modified
|GET|If-Unmodified-Since|0|1|412|-
|GET|If-Unmodified-Since|1|1|200|-
|GET|If-Unmodified-Since|2|1|200|-
|POST/PUT/DELETE|If-Modified-Since|0|1|200|-
|POST/PUT/DELETE|If-Modified-Since|1|1|412|-
|POST/PUT/DELETE|If-Modified-Since|2|1|412|-
|POST/PUT/DELETE|If-Unmodified-Since|0|1|412|-
|POST/PUT/DELETE|If-Unmodified-Since|1|1|200|-
|POST/PUT/DELETE|If-Unmodified-Since|2|1|200|-

結果として、GETメソッド時の「If-Modified-Since」は有効に利用できそうです。\\
POST/PUT/DELETE に関しては、ステータスコードが412になった時点で GET からやり直す振る舞いを
しなければならないということでしょう。
つまりリソースのバージョンもしくは最終更新日時を取得するには情報のリフレッシュが必要だということです。
通常、バージョンもしくは最終更新日時のみが変化していくことは考えづらいため他の情報も更新されていることでしょう。\\
\\
以上のようにチェック自体は簡単に実装できるのですが、肝心の現在の最新バージョン、最終更新日時を取得するには
データベースに情報を永続化しているアプリケーションでは結局データベースへ問い合わせにいくしかないはずなので処理コストは変化しないというよりは
下手すると増えそうです。こうなるとバージョンまたは最終更新日時のキャッシュも視野に入れないといけないのでしょうか・・・\\
\\
悩みましたが、答えがでませんでした。。。\\
何か答えやヒントをくださる方がおられたらコメントいただけると幸いです。
