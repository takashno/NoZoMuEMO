---
layout: post
title: SpringBoot Devtools を IntelliJ IDEA で利用する
categories: [springboot]
tags: [SpringBoot,Devtools,Thymeleaf]
author: takashno
fullview: false
comments: true
img:  # Add image post (optional)
---

SpringBootのDevtoolsを利用していい感じで開発効率上げたい！と考えていますが、\\
なかなかうまく機能せず、困ったためメモエントリーになります。\\
以下からは全て、SpringBootとDevtoolsをIntelliJ IDEAで利用してThymeleafのテンプレートを
キャッシュさせることなく変更をリアルタイムに見る設定方法を記載します。

<!-- 目次 -->
<hr/>
* TOC
{:toc}
<hr/>


## 設定内容

### Gradleの設定
build.gradleのdependenciesに以下を記載。
```groovy
dependencies {
  runtime('org.springframework.boot:spring-boot-devtools')
}
```

### application.properteisの設定
application.properteisに以下を設定。（YAMLの場合は適宜読み替えて下さい。）

```properties
spring.devtools.restart.enabled=true
spring.devtools.restart.exclude=META-INF/maven/**,META-INF/resources/**,resources/**,static/**,public/**,templates/**,**/*Test.class,**/*Tests.class,git.properties,META-INF/build-info.properties
spring.devtools.restart.log-condition-evaluation-delta=true
spring.devtools.restart.poll-interval=1s
spring.devtools.restart.quiet-period=400ms

```

### IntelliJ IDEAの設定
結局はここからが重要になります。

#### 自動ビルド設定
Preference > Build,Execution,Deployment > Compiler\\
この設定の「Build project automatically」にチェックを付与する。
  
![idea]({{ site.url }}/assets/resources/images/2018-09-01-springboot-devtools-withidea/2018-09-01-001.png)

#### レジストリ設定
通常では、自動ビルド設定を行なっていても、既にアプリケーションが実行状態である場合には機能しません。\\
Devtoolsはクラスファイルやリソースの変更を検知して再起動してくれるものだと思いますので、この状態では意味がありません。\\
実行状態でも自動ビルドが走るように設定します。\\
\\
IntellJ IDEAを起動した状態で「Ctrl + Shift + A」を押します。（Cmd + Shift + A）\\
ここに対して「Registry」をタイプします。

![idea]({{ site.url }}/assets/resources/images/2018-09-01-springboot-devtools-withidea/2018-09-01-002.png)

Registryの設定にて、「compiler.automake.allow.when.app.running」のチェックをONにします。\\
この設定を行うことによって、アプリケーションが実行中であったとしても自動ビルドが走るようになります。

![idea]({{ site.url }}/assets/resources/images/2018-09-01-springboot-devtools-withidea/2018-09-01-003.png)

### SpringBootApplicationの起動
通常、Gradleのタスクから「bootRun」を行なっていますが、\\
DevtoolsによるrestartをIntelliJ IDEAにおいて利用する場合は、Gradleタスクは利用できません。\\
理由としては、ビルドした際の出力パスの違いです。\\
Gradleでビルドした場合は、「$projectDir/build」ですが、IntellJ IDEAでは「projectDir/out」になります。\\
Gradleのタスクから「bootRun」を行うとDevtoolsの監視ディレクトリは「$projectDir/build」になるようです。\\
これに対して、SpringBootApplicationクラス（mainクラス）をIntellJ IDEAから直接実行した場合にはDevtoolsの監視ディレクトリが「project/out」となるようです。\\
従って、Devtoolsによるリソース監視＋再起動（restart）を行いたい場合はSpringBootApplicationクラスを直接実行するようにしましょう。\\

#### Profileの指定をどうするのか
SpringBootApplicationクラスを直接実行する際に、SpringBootのProfileを指定したい場合は多々あるかと思います。\\
そう言った場合には、Run&Configurationにて「--spring.profies.active」を直接指定します。\\
以下設定イメージになります。

![idea]({{ site.url }}/assets/resources/images/2018-09-01-springboot-devtools-withidea/2018-09-01-004.png)

#### ThymeleafのテンプレートキャッシュをOFFにしたい
Thymeleafのキャッシュを「false」にしても、Devtools入れても動的に変更できずもどかしくなったのが調査のきっかけだったのですが、\\
結局はビルド時の出力ディレクトリの違いが原因でした。\\
きちんと確認をしているわけではありませんが、Devtoolsを導入すると勝手にcacheが「false」になるようです。\\
そうであったとしてもDevtoolsの参照スコープが「Runtime」なので問題ないでしょう。
