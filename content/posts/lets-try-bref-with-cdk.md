+++
date = '2025-12-04T16:28:57-08:00'
draft = true
title = 'brefを使って、Laravelをサーバレスで動かす'
+++

本記事は『[AWS CDK Advent Calendar 2025](https://qiita.com/advent-calendar/2025/aws-cdk)』の6日目の記事です。

## はじめに

こんにちは、AWS CDKとサーバレス全般が好きな**よ〜ん**です。

初アドカレをAWS CDKで書けることを嬉しく思っています。

## bref/constructsとは

brefについて

## 

## 書きたいこと

業務でAWS CDKとLaravelに触れて2年ぐらい経ちました

個人的な開発でもLaravelでサクッと書くことが多くなったが、どこでホストするか問題

Lightsailで動かしていたりしたけど、勿体無い感（ケチすぎる）
もっとモダンに、無料枠で収まるようにしたいな

Bref CDK コンストラクトを使用すると、AWS CDK を使用して AWS Lambda にサーバーレス PHP アプリケーションをデプロイできた

Lambda web adapterでいいんじゃない？それはそうかもだけど、簡単にできた気がするなあ

### はまりどころ

マイグレ

バッチとかどうするの

### 発見したこと

al2を使ってる

ここを変えても無駄、
runtime 

v3のリリースを待ちましょう

起動は遅いが、一回起動して仕舞えばしばらく早い
