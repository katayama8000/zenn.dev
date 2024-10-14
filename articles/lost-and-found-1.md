---
title: "靴をなくしたので、アプリを作ることにした vol1"
emoji: "🦕"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["ReactNative", "Expo", "Deno", "TypeScript","Firebase"]
published: false
---

## 靴をなくした
今日、靴をなくした。靴をなくした人が身近にいる人がどのくらいいるだろうか。靴がなくなったことよりも、なくしてしまった事実にがっかりしている。海外に３回行ったことがあるが、全てパスポートを無くしている。嫁から記念日にもらったネックレスも無くした。ネックレスを無くした時に、結婚指輪を外して、嫁に預けた。絶対に無くすと思ったから。先週の東京出張で、嫁が遠出する時には指輪をつけていって欲しがるのでつけていった。帰りの新幹線でなんか窮屈に感じて鞄にしまった。それを今思い出して、指輪がなくなっていないことに心底ほっとしている。

くらいに僕は物忘れがひどい。自身を ADHD という人が最近多くて、そんなに軽々しくいうもんじゃねえぞと思っているが、僕はそうかもしれない。ちなみに、 ADHD 診断をしてもらうために病院を予約したこともあるが、それも忘れてすっぽかしてしまった。僕自身は ADHD だと思っていないが、物忘れがひどいのは事実だ。

そして、僕は 11 月にバリ島に行く。しかも一人で。前回タイに行った時は財布とパスポートを無くした。つまり今回も何か無くす可能性がとても高い。というか、無くす。絶対に。

## そんなわけで、アプリを作ることにしたよ
僕が欲しいのは、スマートフォンから、定期的にものを無くしていないか通知してくれるアプリ。通知だけでいい。それだけで大きく違う。僕の経験上、無くしてから時間が経つほど、見つけるのが困難になる。むしろ自分が歩いた道とか思い出せない。無くすのを防ぐのではなくて、無くした時に見つかる可能性を高めるためのアプリである。

ちなみにタイで財布を無くした時は、無くしてから半日以上経っていたので、自分の移動した場所を全て探すのは不可能だった。奇跡的に見つかったが...。
https://note.com/katayama8000/n/n7508eb27b8a8

ちなみに無くすのを防ぐ方法も考えたが、結構厳しい。僕が何か脳のトレーニングをして、物忘れを改善するくらいしかない。ソフトウェアではどうにもならない。同じような方法で、AirTag を全てのものにつけるとかやりようはあるが、めんどくさい上に料金も嵩む。

## なぜ記事を書くのか
個人開発する上で、一番避けなければならないのは、途中で辞めてしまうことである。せっかくプライベートな時間を使っているので勿体ない。その辺をまとめた記事を過去に書いたことがある。

https://zenn.dev/doctormate/articles/app-for-wife34ixb5s

見てくれる人がいるかわからないが、進捗を公開することで、挫折する確率を少しでも下げたい。なぜ Zenn で書くのかというと、技術的なことも書くので。あと執筆もめんどくさいので、ある程度 github copilot の力も借りたかったから、今 VSCode から書いてるってわけ。スクラップを垂れ流すような記事になるかもしれないが、そこはご容赦を。

あとは、割と過程を見るのが好き。「こんなアプリを作りました！」という記事よりも、「こんなん作ってます〜」という記事の方がそそられる。

全 5 回くらいで終わったらいいな。

## 今回の目標: Push通知定期実行の技術調査
調査が必要なところは定期実行の間隔をユーザーに決めさせる方法である。Push 通知の方法や、ネイティブアプリの作り方は問題ない。過去に実装したことがある。

通常、定期実行は cron を使えば良い。しかし、この定期実行の間隔をユーザーに決めさせたい。重要なものは 1 時間おきに。そうれほどでもないものは 3 時間おきになど。あとは夜に通知しないようにしたい。イメージは旅行中にユーザーが設定した間隔で、「◯◯さん、財布無くしていないですか」と通知が来る感じだ。もちろん旅行中だけ有効にしたい。

今回は、cron を使って、Push通知を送る方法までを調査する。

### 今回の技術スタック
調査の前に、今考えている技術スタックを書いておく。
- FE: `ReactNative` + `Expo`
- BE: `Deno` が有力
- DB: 未定 (Deno と相性が良さそうなもの)

通知には Expo が API を提供しているので、それを使う。BE は何を使っても良いので、本当は Rust を使いたいが、デプロイが面倒。Vercel にデプロイしたことあるが、cron が free plan だと使い物にならない。ログが 1 時間しか残らない上に、一日一回しか実行できない。Cloud Run + Cloud Scheduler だとほぼ無料で使えるが、めんどくさい。もっと楽に設定したい。ってことで、Deno を使ってみる。できなかったら、Google Cloud に頼る。

ちなみに僕は、Rust の `expo-push-notification-client-rust` の作者なので、気軽に Contribute してね。(唐突な宣伝)
https://github.com/katayama8000/expo-push-notification-client-rust 

### Deno で cron を使う方法の調査
公式ドキュメントを見ながら進めていく。
OMG、とりあえず、web server を立てて、Deploy してから、cron jobs でそのエンドポイントにリクエストを送ろうと思ったけど、必要ないらしい。
>On Deno Deploy, our multi-tenant distributed serverless JavaScript platform, Deno.cron() is automatically detected and managed so you don’t need to worry about anything.

> You can run cron jobs without a web server or even consistent incoming requests to keep your isolate alive. That’s because whenever your project is deployed, Deno Deploy automatically detects your cron jobs and evaluates them. When its time for your handler to run, Deno Deploy automatically spins up an isolate on-demand to run them.

JSR にパッケージを作った時も思ったが、Deno すごくないかい。Node の終焉も近いかも。

```typescript
Deno.cron("Log a message", "* * * * *", () => {
    console.log("This will print once a minute.");
});
```
これで、デプロイしたら、動いてしまった。これが調査と言えるがわからんが、Deno で cron を使えることはわかった。

順調に進みすぎてしまったので、Deno の cron で Push 通知を送ってみる。

### Deno で Push 通知を送る方法の調査
まあ、調査といっても、API を叩くだけである。クライアント側はまだ作ってないので、token がないが、僕が妻のために作ったアプリがあるので、それで試してみる。

```typescript
Deno.cron("Push notification", "* * * * *", async () => {
    console.log("This will push a message to your device every minute.");
    const pushMessage = {
        to: "ExponentPushToken[xxxxxxxxxxxxxxxxxxxxxx]",
        title: "hello",
        body: "world",
    };

    try {
        const response = await fetch("https://exp.host/--/api/v2/push/send", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify(pushMessage),
        });

        if (!response.ok) {
            console.error(
                "Failed to send push notification:",
                response.statusText,
            );
        } else {
            console.log("Push notification sent successfully!");
        }
    } catch (error) {
        console.error("Error sending push notification:", error);
    }
});
```
問題なく動いた。これで十分だが、Expo Server SDK があるので、npm package を Deno から使ってみる。

```typescript
import { Expo } from "expo-server-sdk";

const expo = new Expo();

Deno.cron("Push notification", "* * * * *", async () => {
    const pushToken = "ExponentPushToken[xxxxxxxxxxxxxxxxxxxxxx]";

    if (!Expo.isExpoPushToken(pushToken)) {
        return;
    }

    const messages = [{
        to: pushToken,
        title: "hello",
        body: "world",
    }];

    try {
        const chunks = expo.chunkPushNotifications(messages);
        for (const chunk of chunks) {
            await expo.sendPushNotificationsAsync(chunk);
        }
    } catch (error) {
        console.error(error);
    }
});
```
順調にいきすぎている。もう半分くらい終わった気分だ。
次回は、DB or ユーザーが設定した間隔で通知を送る方法を調査する。

また次回 :)

## References
https://deno.com/blog/cron
https://vercel.com/docs/cron-jobs/usage-and-pricing
https://github.com/expo/expo-server-sdk-node




