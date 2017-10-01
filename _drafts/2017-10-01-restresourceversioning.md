---
layout: post
title: REST-APIにおけるリソースのバージョニング
categories: [memo]
tags: [REST-API,SpringBoot,SpringMVC]
author: takashno
fullview: false
comments: true
---

{% assign author = site.data.people[page.author] %}
{{ author.name }}です。\\
SpringBootを利用してREST-APIを作成していますが、\\
リソースのバージョニングをどうやるべきなのかに迷い出しています。\\
\\
調べて結果としては、ETag/If-Match/If-None-Matchによる制御が一番しっくりきそうです。

// 書きかけ。。。