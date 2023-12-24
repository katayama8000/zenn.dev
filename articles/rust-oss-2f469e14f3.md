---
title: 'Rust で Expo の push通知 SDK を作って公開した'
emoji: '🦍'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [rust, oss, expo, react-native, notification, firebase]
published: false
---

## なぜ作ったのか

[Expo server SDK](https://docs.expo.dev/push-notifications/sending-notifications/#send-push-notifications-using-a-server) は、Node 以外、有志で作られており、Rust は存在しますが、最後のコミットが４年前に止まっています。おそらく、Rust の最新バージョンでは動作しません。本業で、API サーバーを Rust で書いているので、学習がてら、作ってみました。

余談ですが、本業でインフラを gcp に頼っているのですが、ほぼほぼ、Rust サポートがありません。例えば、firebase admin SDK も Rust はありませんので、Rust から firebase を使う場合は、REST API を叩いております。車輪の再発明感がありますが、その過程を楽しみながら、開発しております。

## 作ったもの

https://github.com/katayama8000/expo-push-notification-client-rust

使い方は、README に書いてありますが、以下のような感じです。

```rust
use expo_server_sdk::{CustomError, Expo, ExpoClientOptions, ExpoPushMessage, ExpoPushTicket};
let expo = Expo::new(ExpoClientOptions {
    access_token: Some(access_token),
});
let expo_push_message = ExpoPushMessage::new(expo_push_tokens, title, body);
expo.send_push_notifications(expo_push_message).await;

let expo_push_ids = ExpoPushReceiptId::new(vec!["xxxxx".to_string(), "xxxxx".to_string()]);
expo.get_push_notification_receipts(expo_push_ids).await;
```
