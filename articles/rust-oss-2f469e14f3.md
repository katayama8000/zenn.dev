---
title: 'Rust で Expo の push通知 SDK を作って公開した'
emoji: '🦍'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [rust, oss, expo, react-native, notification, firebase]
published: false
---

## なぜ作ったのか

[Expo server SDK](https://docs.expo.dev/push-notifications/sending-notifications/#send-push-notifications-using-a-server) は、Node 以外、有志で作られており、Rust は存在しますが、最後のコミットが４年前に止まっています。おそらく、Rust の最新バージョンでは動作しません。業務で、API サーバーを Rust で書いているので、学習がてら、作ってみました。

余談ですが、業務ではインフラを gcp に頼っているのですが、ほぼほぼ、Rust サポートがありません。例えば、firebase admin SDK も Rust はありませんので、Rust から firebase を使う場合は、REST API を叩いております。車輪の再発明感がありますが、その過程を楽しみながら、開発しております。

## 作ったもの

https://github.com/katayama8000/expo-push-notification-client-rust

### 使い方

README に書いてありますが、以下のような感じです。

```rust
use expo_push_notification_client::{CustomError, Expo, ExpoClientOptions, ExpoPushMessage, ExpoPushTicket};
let expo = Expo::new(ExpoClientOptions {
    access_token: Some(access_token),
});
let expo_push_message = ExpoPushMessage::new(expo_push_tokens, title, body);
expo.send_push_notifications(expo_push_message).await;

let expo_push_ids = ExpoPushReceiptId::new(vec!["xxxxx".to_string(), "xxxxx".to_string()]);
expo.get_push_notification_receipts(expo_push_ids).await;
```

クライアント側の設定(ReactNative with Expo)が割と面倒です。Android は Expo が内部で FCM(Firebase Cloud Messaging) を使っているので、gcp の設定が必要です。iOS は、端末がなかったので、試していませんが、有料 の Apple Developer Program に加入している必要があるようです。アカウントさえあれば、設定は簡単そうです。くわしくは、[Expo の公式ドキュメント](https://docs.expo.dev/push-notifications/push-notifications-setup/)を参照してください。

## どうやって作ったか

色々、回り道をしましたが、最終的には [expo-server-sdk-node](https://github.com/expo/expo-server-sdk-node) を参考に作りました。node ver ほど作り込まれていませんが、`Done is better than perfect` ということで、とりあえず、動くものを作り、公開しました。

## crate を作成した感想

(Rust の言語についての感想ではなく、crate を作成した感想です。)

1. 一番に驚いたことは、作成中の crate を他のプロジェクトで、デバックしながら、開発できます。

   ```toml
   [dependencies]
   expo_push_notification_client = {git = "https://github.com/katayama8000/expo-push-notification-client-rust", branch="main" commit="xxxxx"}
   ```

   cargo.toml に上記のように記述すると、git から、直接、crate を取得してきてくれます。調べたわけではありませんが、`npm package` では、こんなに簡単に、開発版を使うことができません。(多分)。

2. Rust では、crate 内でのみ使用する関数を、`pub` にすると、外部からも使用できてしまうので、`pub(crate)` という記述をすることで、crate 内でのみ使用できるようにできます。公開範囲を簡単に制限できて、使い勝手が良いです。

3. Rust 色々揃っていないなと感じました。使われる、OSS は開発者であれば、誰しも作ってみたいと思うはず...!!。Rust チャンスありです。

## 最後に

初めて、crate を作成しました。PR、issue、star など、いただけると、とんで喜びます！！
