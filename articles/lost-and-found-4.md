---
title: "靴をなくしたので、アプリを作ることにした vol4 そして伝説へ"
emoji: "🐈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["ReactNative", "Expo", "Deno", "TypeScript"]
published: false
---

## 伝説でもなんでもないが
タイトルを考えていたら、僕が好きなドラクエのテーマ曲を思い出した。小学生の低学年の頃、ドラゴンクエストジョーカーをやっていて、友達より先にバベルボブルをゲットして、自慢していた。人生のピークだったなぁ。。。

3週間連続で旅行に行ったので、全然開発ができなかった。だけど旅行はいいね。人の温かさを感じられて、僕も人に優しくしようと思える。

アプリともプログラミングとも全く関係のないものになってしまったので、前置きはこれくらいにして。

## 今回の目標: 完成
### Deno と Firestore と接続
公式ドキュメントにも、Firebase のセットアップ方法は記載があった。
```js
import firebase from "https://esm.sh/firebase@8.7.0/app";
import "https://esm.sh/firebase@8.7.0/auth";
import "https://esm.sh/firebase@8.7.0/firestore";
```
どうやら、esm.sh というものがあるらしい。
```
Create modern(es2015+) web apps easily with NPM packages in browser/deno.
No build tools needed!
```
npm packages をブラウザや Deno でしかも es2015 以降のバージョンで使えるようにするものらしい。便利だね。

CLI からも使えるらしい。
```
# Adding packages
deno task esm:add react react-dom     # add multiple packages
deno task esm:add react@17.0.2        # specify version
deno task esm:add react:preact/compat # using alias

# Updating packages
deno task esm:update react react-dom  # update specific packages
deno task esm:update                  # update all packages

# Removing packages
deno task esm:remove react react-dom
```
node だとパッケージマネージャーがたくさんあるけど、Deno だと統一されていていいね。

ちなみに Firebase は、Google の CDN から読み込むこともできるらしい。
```js
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.1/firebase-app.js";
import {
    collection,
    doc,
    getDoc,
    getDocs,
    getFirestore,
    query,
    setDoc,
    where,
} from "https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore.js";
```

これで動作した。
Firebase の JS SDK なのですが、サーバーサイドでは Admin SDK しか動かないと思っていたゾ...?。
Deno だと JS SDK でも動くの不思議だな。

動作は確認できたので、あとはロジックを書くだけだ。
通知判定でみるべきフィールドは3つ。`isNotifyEnabled` `lastNotifiedAt` `reminderInterval` だ。

- `isNotifyEnabled` は、通知が有効かどうかを判定する。
- `lastNotifiedAt` は、最後に通知した日時を記録する。
- `reminderInterval` は、通知の間隔を設定する。

ロジックは難しくない。`isNotifyEnabled` が `true` かつ、`現在日時` -`lastNotifiedAt` が `reminderInterval` より大きい場合に通知を送る。

```typescript
export const shouldNotify = (
    lastNotifiedAt: string | null,
    reminderInterval: number,
): boolean => {
    if (!lastNotifiedAt) return true;
    const now = Date.now();
    const lastNotifiedTime = new Date(lastNotifiedAt).getTime();
    return now - lastNotifiedTime >= reminderInterval;
};
```

## 動作確認




## Referece
https://docs.deno.com/deploy/tutorials/tutorial-firebase/
https://esm.sh/
https://firebase.google.com/docs/web/alt-setup?hl=ja

