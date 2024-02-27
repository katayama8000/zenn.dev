---
title: 'Rust 製 Expo SDK を作ったら、Expo の公式ドキュメントに掲載された'
emoji: '🦍'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [rust, oss, expo, reactnative, firebase]
published: true
published_at: 2024-02-28 12:30
publication_name: 'doctormate'
---

## 作成の経緯

[Expo Server SDK](https://docs.expo.dev/push-notifications/sending-notifications/#send-push-notifications-using-a-server) は、Node.js 以外、有志で作られており、Rust は存在しますが、最後のコミットが 5 年前に止まっています。しかも、Rust の最新バージョンでは動作しません。業務で、API サーバーを Rust で書いているので、学習がてら作ってみました。

## 作ったもの

https://github.com/katayama8000/expo-push-notification-client-rust

簡単に説明すると、Expo で Push 通知を送るための Client です。
Expo が [API](https://docs.expo.dev/push-notifications/sending-notifications/#http2-api) を提供しているので、それをラップする形で作っています。
主に、パラメータのバリデーション、戻り値の型、エラー処理を提供しています。

### 使い方

[README](https://github.com/katayama8000/expo-push-notification-client-rust?tab=readme-ov-file#server-rust) に書いてありますが、以下のような感じです。
client を作成して、push 通知を送るメソッドと、push 通知のステータスを取得するメソッドを提供しています。これは、Expo がこのような API を提供しているためです。

```rust
use expo_push_notification_client::{Expo, ExpoClientOptions, ExpoPushMessage, GetPushNotificationReceiptsRequest};

let expo = Expo::new(ExpoClientOptions {
    access_token: Some(access_token),
});

let expo_push_tokens = ["ExponentPushToken[xxxxxxxxxxxxxxxxxxxxxx]"];
let expo_push_message = ExpoPushMessage::builder(expo_push_tokens).build()?;
expo.send_push_notifications(expo_push_message).await;

let expo_push_ids = GetPushNotificationReceiptsRequest::new(vec!["xxxxx".to_string(), "xxxxx".to_string()]);
expo.get_push_notification_receipts(expo_push_ids).await;
```

送信時のパラメーターも設定できます。
例えば、以下のように、タイトル、本文、を設定したい場合、以下のようになります。

```rust
let expo_push_message = ExpoPushMessage::builder(expo_push_tokens)
    .title("Hello")
    .body("World")
    .build()?;
```

クライアント側の設定 (ReactNative with Expo) が割と面倒です。Android は Expo が内部で FCM (Firebase Cloud Messaging) を使っているので、GCP の設定が必要です。iOS は、端末がなかったので、試していませんが、有料 の Apple Developer Program に加入している必要があるようです。アカウントさえあれば、設定は簡単そうです。くわしくは、[Expo の公式ドキュメント](https://docs.expo.dev/push-notifications/push-notifications-setup/)を参照してください。

## どうやって作ったか

色々、回り道をしましたが、最終的には [expo-server-sdk-node](https://github.com/expo/expo-server-sdk-node) を参考に作りました。一部たりていない部分 ([chunk](https://github.com/katayama8000/expo-push-notification-client-rust/issues/91) 周り) がありますが、基本的な機能は実装しています。

## 社内に公開してみた

開発を決めて、二週間くらいで、できたものを社内に公開しました。ver 0.1.0 です。Rust に知見の深い方が社内に在籍しているので、レビューしていただきました。ここから、１週間以上毎日 PR を出していただき、完成度が上がりました。

日本語が話せる同士ですが、なぜか英語でやりとりしていました。OSS 活動っぽいですね。
![PR](/images/rust-oss/PR.png)

## Expo の公式ドキュメントに掲載されるまで

この crate を作成中、密かに、Expo の公式ドキュメントに載せてくれないかと考えていました。ほとんどメンテナンスされてないものが掲載されていたので、チャンスがあるかもしれないと。反対に、僕が作成したことによって、メンテナンスし出すのではないかと思いましたが、それはそれで、嬉しいことです。

まずは、Discord に参加しました。最近の IT 業界のコミュニティは、Discord が主になっているようです。
「#expo-notification」というチャンネルがあったので、いきなり宣伝してみました。
![discord](/images/rust-oss/discord1.png)
ハートが一つだけつきましたが、ほぼフルシカトでした。
これは、Discord に参加して、いきなり宣伝するんじゃないぞという洗礼かと思い、「#expo-notification」内に上がってくる質問にしばらく答えてました。crate の開発中、幾度となくドキュメントを読んでいたので、知見が溜まったのは、良かったです。この行為自体は、おそらく意味なかったです。

次に、Expo 社内の人らしき人が、Discord にいたので、フレンド申請をして、「便利なものを作りました」と報告してみました。今のところ、友達になれていません。

こういう時は、どこに相談すればいいものかと思っていたところ、「#docs」というチャンネルを見つけました。ここでは、宣伝ではなく、ドキュメントに載っているものが動かないので、私が作ったものを載せてくれないかというお願いを正式にしました。
![discord](/images/rust-oss/discord2.png)

すると、Expo 社内の人が返事をくれました。
![discord](/images/rust-oss/discord3.png)
レビューして、OK が出たら、公式ドキュメントに載せてくれるとのことでした。
2、3 週間かかると言っていたので、README が丁寧に書いていないとか、テストが足りないとか、レビューが山ほど来るかと思いましたが、割とすんなり OK が出ました。その間、３日でした。

![discord](/images/rust-oss/discord4.png)

[Jon Samp (Expo の人)](https://jonsamp.dev/) さんが２つ PR を作成してくれました。一つは、既存の [expo-server-sdk-rust](https://github.com/expo/expo-server-sdk-rust) をもうメンテナンスせずに、私が作成した、crate を使うように促す PR 。
[もう一つ](https://github.com/expo/expo/pull/27116)は、公式ドキュメントに私が作成した crate を載せる PR でした。

両方とも、無事にマージされ、[公式ドキュメント](https://docs.expo.dev/push-notifications/sending-notifications/#send-push-notifications-using-a-server)に載せてもらえることになりました。
![expo](/images/rust-oss/expo-doc.png)

## まとめ

初めて、作成した crate が Expo の公式ドキュメントに載せていただき、とても良い経験になりました。
web 開発における Rust はまだまだ揃ってない部分があり、反対に言えば、自分で作るチャンスが多いです。これを機に OSS 活動をもっとしていきたいと思いました。

リポジトリへの issue、PR、Star など、いただけると嬉しいです。
