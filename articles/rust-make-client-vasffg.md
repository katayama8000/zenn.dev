---
title: "Rust で Client(API Wrapper) を作る手順"
emoji: "🦍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust", "API"]
published: false
---

## 初めに
Rust で API クライアントを作る手順をまとめました。
Rust はまだまだ、プロダクションで使われることが少ないので、それに伴って、ライブラリも少ないです。
逆に言えば、ライブラリを作るチャンスが多いということです。
自作ライブラリを作ったことない人でも、とっつきやすいと思います。

# APIを選ぶ
まずは、API を選びます。
公開 API がまとまっている[サイト](https://apidog.com/apihub/)とかあるので、そこから選ぶといいかもしれません。
今回私は、[gyazo api](https://gyazo.com/api/docs/image)を選びました。
理由は、Scrapbox が好きなこと(関係ない)と書き込みAPIが存在したためです。

## ライブラリを作る
### Client を作る
### Input の定義をする
### Output の定義をする
### エラーの定義をする
### テストを書く

## まとめ
Rust で API クライアントを作る手順をまとめました。

