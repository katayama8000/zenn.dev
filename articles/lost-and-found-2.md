---
title: "靴をなくしたので、アプリを作ることにした vol2"
emoji: "🦕"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["ReactNative", "Expo", "Deno", "TypeScript","Firebase"]
published: false
---

## 靴
は、まだ見つかっていない。ジムでなくした可能性が高いので、電話番号を伝えてきた。あと、安い靴を注文した。

話は変わるけど、X で Youtube で配信しながら開発することを、配信駆動開発と名付けていた方がいた。であれば、記事を書きながら開発するのは、記事執筆駆動開発とでも読んだらいいのだろうか。

## 今回の目標: DBを決める & DB設計

### DBを決める
前回、DB は Deno と相性のいいものを使うと書いた。Deno Deploy を触っていたら、Deno KV なるものの存在に気づいた。KV とは Key-Value の略だろう。cron と同じで、beta 版なので、安定しているかわからない。これを使ってみたいのだが、そもそも　Deno から、DB に書き込んだりする必要がないじゃないか。このアプリにわざわざ、API 経由で書き込むのは、必要ない。

Firestore を使おう。Android の Push 通知にもどうせ Firebase の設定が必要である。従量課金なので、まず僕しか使わない段階では無料で済む。

### DB設計
まず、通知の間隔をユーザーに決めてもらうための設計を考える。

1. 最小の通知間隔で定期実行 (おそらく 1 時間)
2. ユーザーに紐づく情報を Firestore から取得
3. 通知間隔を確認
4. 最後の通知時間を確認
5. 現在の時間 - 最後の通知時間 > 通知間隔 なら通知
6. 通知完了後、通知時間を書き込む

効率が悪そうであるが、これ以外に方法が思いつかない。

Firestore のデータ構造を考える。TypeScript の型でデータ構造を表す。

```typescript
type User = {
  tokens: string[];
}

type Trip = {
  destination: string;
  startDate: Timestamp;
  endDate: Timestamp;
}

type Item = {
  name: string;
  notificationInterval: number; // 通知間隔（分）
  lastNotifiedAt: Timestamp; // 最後の通知日時
}
```

各コレクションは、親子関係なので、互いの ID を持つ必要は特にない。

`users/{userId}/trips/{tripId}/items/{itemId}`

通知のオンオフとか付け足したくなるかもしれないが、とりあえずこれで。

## クライアント側の環境構築
あとは、クライアント側の環境構築だけしておこう。
https://docs.expo.dev/tutorial/create-your-first-app/

Expo チームはアップデート頻度が本当に高いな。ドキュメントがどんどん更新されている。
環境構築 80% くらい完了。プロジェクト作成して、Biome を入れて、Firestore と通信も確認。
あとは、Expo に サーブスアカウントをアップロードなどして、プッシュ通知を受け取れることを確認する。
https://github.com/katayama8000/lost-and-found-client

また、次回 :)

## 前回の記事
https://zenn.dev/doctormate/articles/lost-and-found-1




