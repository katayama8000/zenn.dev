---
title: "靴をなくしたので、アプリを作ることにした vol3 それって、作る必要ある？"
emoji: "🐈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["ReactNative", "Expo", "Deno", "TypeScript"]
published: true
---

## 前回の記事
前回の記事はこちら。
https://zenn.dev/tattu/articles/lost-and-found-2

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
公式ドキュメントに沿って進めていくだけだ。
割とめんどくさい。

簡単に言うと、
1. push token を取得する
2. その token を Expo の Server に登録する
3. その token を API のリクエストに乗せて送る
4. Android は Firebase の FCM 経由で通知が送られる

という流れだ。
おそらく、Expo がネイティブの機能をラップしてくれているのだが、それでもめんどくさい。
作業するだけなので、特に問題なく完了。
https://github.com/katayama8000/lost-and-found-client/pull/3

### デザインの仮作成
画面はとりあえず２つだけでいい。ホーム画面と登録画面のみ。
わざわざ Figma で作るのも面倒なので、コードで書く。

|![app1](/images/lost-and-found/UI1.png =250x)|![app2](/images/lost-and-found/UI2.png =250x)|
|---|---|

とっても雑だけど、とりあえずこれでいい。念の為に伝えておくが、AIにコードを書かせた。
僕はもう少しだけデザインセンスがある。

あとは、Push 通知がを受け取った時のモーダルである。

モーダルのイメージとしては、通知されたものがリストで表示され、「ある」「ない」「わからない」の選択をさせる。Push 通知のリクエストにアイテムの id を含めて、クライアント側では、その id で Firestore からデータを取得して表示する。

バックエンドからは、下記のような感じで、Push 通知を送る。
```typescript
async function sendPushNotification(expoPushToken: string) {
	const message = {
		to: expoPushToken,
		title: "パスポート",
		body: "パスポートを持っていますか？",
		data: { ids: ids },
	};

	await fetch("https://exp.host/--/api/v2/push/send", {
		method: "POST",
		headers: {
			Accept: "application/json",
			"Accept-encoding": "gzip, deflate",
			"Content-Type": "application/json",
		},
		body: JSON.stringify(message),
	});
}
```

上記の例ではパスポートの１アイテムだけだが、複数のアイテムが通知されることもあるので、その場合は、`data: { ids: ids }` の ids に複数の id を含める。

モーダルのイメージは下記のような感じ。

![modal](/images/lost-and-found/UI3.jpg =250x)

通知が送られてきたら、このモーダルが表示される。僕はこの画面を見たら、大事なものを無くしていないか、確認することになるだろう。実質このアプリの役割はここにある。

**僕にパスポートの存在を思い出させるのだ。無くしたことにきづかせるのだ。**

2日前に無くなったものなんてまず見つからない。1時間前に無くしたものなら、まだ見つかる可能性がある。このアプリは、その可能性を高めるためのものだ。

これでクライアント側の実装は、ほぼ完了。

え、ログイン画面は？更新機能は？と思ったあなた。
記事をちゃんと読んでくれていてありがとうございます。

今回は僕しか使わないので、実装しなくていいものはとことん落とす。ログインもいらないし、DB から取得すべきところもハードコードしていたりする。
更新機能も、今回はいらない。いや、流石に欲しいけど、もう面倒くさい。

Firestore のコンソールから、通知したくない項目の設定を変えればいいじゃないか。列記とした DB のクライアントである。
セキュリティルールも全て通す。

僕には旅行まで一週間しかないのだ。平日は仕事なのであまり作業できないし、休日も予定があるので、実質あと数時間しかない。

そう、記事を書いている場合ではないのである。

今回はここまで。次回はサーバー側の実装を行なって完成だ。

またね:)

## References
https://docs.expo.dev/push-notifications/push-notifications-setup/
