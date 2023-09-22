---
title: "rustのイテレーター"
emoji: "🦍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rust]
published: false
---

# イテレーターとは
配列やベクターで繰り返し処理を行うときに使う。
```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
for val in v1_iter {
    println!("Got: {}", val);
}
```
これで`for`と同じような動きになる。

### Iteratorトレイト
全てのイテレータは、標準ライブラリで定義されている`Iterator`というトレイトを実装している。
```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // デフォルト実装のあるメソッドは省略
    // methods with default implementations elided
}
```
見てわかるように、`next`が生えているので使える。
```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```
`assert_eq!`はテストの時に使うもので、左と右がイコールであることを確かめている。

# 消費
https://doc.rust-jp.rs/book-ja/ch13-02-iterators.html


## iter()
## iter_mut()
## into_iter()