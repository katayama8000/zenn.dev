---
title: "身近な人やパートナーにアプリを作るすゝめ"
emoji: "🦍"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["ReactNative", "Expo", "TypeScript", "Rust", "Vercel"]
published: true
publication_name: 'doctormate'
---
:::message
この記事は、弊社ドクターメイトの社内LTで発表したものです。
:::
## 生きてるレポジトリ持っていますか？

### ここで言う生きてるレポジトリとは
- 定期的に更新されていること
- 利用者がいること（自分も含む）

### 調査結果
- 持っている ... 5%
- 持っていない ... 95% （katayama調べ）

### なぜ生きているレポジトリを持つべきか
- 自分の試したいことが試せる
    - サンドボックス環境で試すのとは大きな違いがある
- 愛着が湧く
- 名刺代わりになる

### 生きているレポジトリを作るためには
#### 個人でサービスを作る
- 使ってもらうハードルが高い
- 挫折率が高い
#### ライブラリ等を作る
- 開発ハードルが高い
    - 良い題材があればあり
- ものにもよるが、一回できてしまったら、機能追加などする必要があまりない
#### 個人ブログを作る
- 開発ハードルが低い (使用する技術にもよる)
- 使ってもらうハードルが低い (自分が使えば良いので)
- こだわれば、いろいろな機能を追加できる
- 数多にブログサービスがあるから途中でやる気がなくなる
    - 実際に僕もRustで個人用のブログを作ろうとしたが、途中でやる気がなくなった
    - https://github.com/katayama8000/blog-rust
#### パートナーにアプリを作る
- 使ってもらうハードルが低い（ほぼなし）
- 自分のためだけではないので、やる気が出る
- 機能追加しやすい

### 表にしてみる
| プロジェクト                       | 開発ハードルの低さ | 使ってもらうハードルの低さ | やる気の出やすさ |
|--------------------------------|--------------|--------------------|--------------|
| 個人でサービス開発                | 4            | 2                  | 6            |
| ライブラリ等の開発                   | 2            | 3                  | 5            |
| 個人ブログ開発                     | 7            | 9                  | 2            |
| パートナーにアプリ開発             | 6            | 8                  | 9            |

## どんなアプリを作るか
- 使った金額をパートナー間で請求できるアプリ

### なぜ作ったか
- 僕たちは結婚してからも別々でお金を管理している
- 普段、僕は財布を持ち歩かないので（過去に何度も無くしているから）、いくら使ったか記録する必要がある
- 買い物を頼むことがよくある（夫だけAmazonプライム会員だったり、嫁は楽天で買い物をすることが多いのでポイントを貯めていたりする）
- Notion で管理していたが、Notion のモバイルUIが使いにくい
- Expo の機能を試してみたかった
- [自作OSS](https://github.com/katayama8000/expo-push-notification-client-rust) を使いたかった

### 実際のUI
|![app1](/images/app-for-wife/app1.jpg =250x)|![app2](/images/app-for-wife/app2.jpg =250x)|![app3](/images/app-for-wife/app3.jpg =250x)|
|---|---|--|

## アプリの構成
- Frontend
    - React Native
    - Expo
- Backend
    - Firebase (FCM)
    - Supabase (Auth, Datastore)
    - Vercel (Serverless Functions)
        - Push 通知用 Rust 製 API を Vercel にデプロイ
        - https://github.com/katayama8000/expo-push-notification-api-rust
- CI/CD
    - GitHub Actions

## 作った感想
- ネイティブアプリを作るのが大変だった
    - 体感でWebの3倍くらい
    - Expoの無料プランが辛い
        - ビルド時間が長いし、ビルド回数に 30/Month 制限がある
- 使ってもらう対象を一人に絞ると、機能が絞られるので、リリースまで行きやすい
    - 例えば、自分のアプリには、新規登録機能がない
    - 固定請求項目はユーザーが選択できず、固定のクエリを発行している
- フィードバックもらえるのが楽しい
    - 個人開発でフィードバックをもらった試しがないので、これは楽しい
    - アプリ内にGitHubのIssueリンクを貼っており、そこに記載してもらっている
        ![issue](/images/app-for-wife/issue.png)
- 継続利用してもらえる人がいる
    - 少なくとも、嫁は使ってくれている
    - 身近な人へ需要があるということは、僕と似たような環境の人にも需要がある可能性が高い
    - 嫁の友人カップルも使っている
- デザインの細かなこだわりや軽微な修正はほぼ無
    - 画面遷移や Pull to Refresh の実装などのUXの改善は効果がある
- 使ってもらうことに意味がある
    - どんなに流行りの技術でいいアプリを作っても、使ってもらえなければ意味がない

## まとめ
これらを踏まえて、生きてるレポジトリを持つためには、身近な人やパートナーにアプリを作るのがいいと思いました。皆さんも次アプリを作る際、toB、toC、いや、toPアプリを作ってみてはいかがでしょうか？

https://github.com/katayama8000/household-account-book