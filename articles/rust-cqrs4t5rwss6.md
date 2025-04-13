---
title: 'RustとDDDとCQRSとEvent-SourcingでAPIサーバーを構築する'
emoji: '🦍'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [Rust, axum, ddd, cqrs, event-sourcing]
published: false
---

## はじめに

### 対象読者

- CQRS とは何かを知りたい人
- イベントソーシング とは何かを知りたい人
- Rust で CQRS とイベントソーシングを実装したい人

### 説明しないこと

- Rust の基本的な文法
- DDD の基本的な考え方
- CQRS の基本的な考え方
- イベントソーシングの基本的な考え方

以前、[Rust と DDD で API サーバーを構築する](https://zenn.dev/doctormate/articles/rust-ddd-7353b79179) 記事を書いたので、DDD を使った API サーバーの構築方法を知りたい方は、そちらを参考にしてください。

https://github.com/katayama8000/axum-ddd-rust

:::message
あくまでも _Rust_ で CQRS やイベントソーシングを実装することが目的です。
基本的な考え方は公式ドキュメントや書籍を参考にすると良いでしょう。
:::

## CQRS とイベントソーシング

CQRS (Command Query Responsibility Segregation) は、コマンドとクエリの責任を分離するアーキテクチャスタイルです。

![cqrs](/images/rust-cqrs/image1.png)

書き込み用の DB と読み取り用の DB を分け、何かしらの方法で同期します。
CQRS の利点は、書き込みと読み取りの責任を分離することで、システムのスケーラビリティとパフォーマンスを向上させることができる点です。

また、今回は、イベントソーシングを併用します。
イベントソーシングは、状態の変更をイベントとして保存し、そのイベントを元に状態を再構築するアーキテクチャスタイルです。

後程コード例と共に説明します。

## 作成するシステム

前回と同様、大学がサークルを管理するシステムを作成します。
簡単のため、今回のシステムは、サークル名とサークルの許容人数を管理するシステムとします。

##
