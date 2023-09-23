---
title: "rustのイテレータ"
emoji: "🦍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rust]
published: true
published_at: 2023-09-25 12:30 # 未来の日時を指定する

---

# イテレータとは
配列やベクター,HashMapで繰り返し処理を行うときに使う。
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
`next`を呼ぶとイテレータは消費される。
上の例では、4回目の`v1_iter.next()`で`None`と比較している。
これがイテレーターが消費されているのを表している。
出力してみると、もっとわかりやすい。

```rust
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];
    let mut v1_iter = v1.iter();
    v1_iter.next()
}
```
一つめの要素が消費されて、`Iter([2, 3])`と出力されたのが確認できる。
`next`を呼び出して、イテレータを消費するメソッドもある。
sumメソッドは、イテレータの所有権を奪い、nextを繰り返し呼び出すことで要素を繰り返し、 故にイテレータを消費する。

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();
    let total: i32 = v1_iter.sum();
    assert_eq!(total, 6);
}
```

このほかにも他のイテレータを作成するメソッドなどもあるので、documentを貼っておく。
https://doc.rust-jp.rs/book-ja/ch13-02-iterators.html

# 様々なイテレータ
## iter()
`iter()`は不変な参照を作成します。
```rust
let v = vec![1, 2, 3];
let v_iter = v.iter();
```

## iter_mut()
`iter_mut()`は可変な参照を作成します。
```rust
let mut v = vec![1, 2, 3];
let v_mut_iter = v.iter_mut();
```
可変参照なので、元の配列の値を書き換えることもできます。
```rust
let mut v = vec![1, 2, 3];
for num in v.iter_mut() {
    *num *= 2;
}
```

## into_iter()
`into_iter()`は所有権を奪います。
```rust
let v = vec![1, 2, 3];
let v_into_iter = v.into_iter();
```
