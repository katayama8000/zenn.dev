---
title: "靴をなくしたので、アプリを作ることにした vol2"
emoji: "🐈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["ReactNative", "Expo", "Deno", "TypeScript"]
published: false
---

## それ、LINEでできるじゃん
嫁が、旅行から帰ってきて、アプリを作ることを伝えた。「それ良いけど、LINEのリマインくんでできるよ」と言われた。
...
...
...
まじか。
確かに、旅行中に物をなくさないことに特化したアプリはないかインターネッツで探してみたが、もっと汎用的なアプリでできることは考えていなかった。嫁氏、さすがである。早速使ってみたが、僕が求めていることはできなかったし、UX も僕が求めているものとは程遠い。一安心(？)である。

しかし、LINEではなくても、チャットアプリの API をつかえば、僕が求めているものは作れることを気づいてしまった。
この手のアプリの API は Discord が一番使われている印象がある。Slack は仕事感があるので使いたくない。

しかしよく考えると、Discord でできるとして、そんなに工数が変わるのか？ほぼ調査が終わって実装するだけ、しかもやったことある内容だ。Discord API を調べて、Discord のイベント発火時に DB にデータを書き込む方法とか調べていたら、もうそっちの方が時間がかかる。
明らかなメリットは Firebase を使わなくても良くなることだ。Deno の KV と cron あとは Discord だけで完結させることができる。
デメリットは UX が悪い。PC から操作すること前提であれば、まだ許せそうだが、スマホから操作するとなると、おそらく使わなくなる。やっぱりアプリを作ろう。

## アプリの名前を考えた
ズバリ、「Lost and Found」だ。靴を無くしたことがきっかけで作るアプリとは思えないクールな名前だが、結構気に入っている。ポイントは無くすことを防ぐのではなく、無くした時に見つかる可能性を高めることだ。そのことが名前にも表れていてとても良い。

## 今回の目標: 通知を受け取る & デザインの仮作成
### 通知を受け取る
### デザインの仮作成