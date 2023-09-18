---
title: "rustのエラーハンドリング"
emoji: "🦍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rust"]
published: false
---

# Result型とは
https://doc.rust-jp.rs/rust-by-example-ja/error/result.html
> Resultは、リッチなバージョンのOption型で, 値の不在の可能性の代わりにエラーの可能性を示します。
> つまり、Result<T, E>は以下の２つの結果を持ちます。
> Ok<T>: 要素Tが見つかった場合
> Err<E>: 要素Eとともにエラーが見つかった場合
> 慣例により、Okが期待される結果であり、Errは期待されない結果です。
## 実際にResultを返す関数を作ってみる
読むだけでは分かりにくいので、実際に書いてみます。
今回はフルネームを返す関数を作成し、戻り値を`Result`にしてみます。

```rust
fn build_full_name(last_name: &str, first_name: &str) -> Result<String, String> {
    if last_name.is_empty() || first_name.is_empty() {
        return Err("空文字です".to_string());
    }

    let full_name = format!("{} {}", last_name, first_name);
    Ok(full_name)
}
```
失敗時は`空文字です`、成功時は`full_name`を返し、中身は`Result`のジェネリクスに書いているように`String`を返します。(`format!`の戻り値は`String`です)

## 呼び出してみる
```rust
let last_name = "yamada";
let first_name = "";

match build_full_name(last_name, first_name) {
    Ok(full_name) => {
        println!("Full Name: {}", full_name);
    }
    Err(error) => {
        println!("Error: {}", error);
    }
}
```
`first_name`が空文字ですので、`空文字です`と出力されるはずです。
`match`を使って、エラーハンドリングをしておりますが、rustにはさまざまな方法でエラーハンドリングができるので、試していこうと思います。

# Result型のエラーハンドリング

## match
上に記述してある方法です。
基本的には`Ok`または`Err`で場合わけをして、その中の処理を書いていきます。

## let if
`build_full_name`の戻り値をif文を用いて判定しています。
`else`に`unwrap`が入っていますが、後述します。

```rust
let last_name = "yamada";
let first_name = "";

if let Err(error) = build_full_name(last_name, first_name) {
    println!("Error: {}", error);
} else {
    println!("Full Name: {}", build_full_name(last_name, first_name).unwrap());
}
```

## unwrap
`unwrap`は`Err`時に`Panic`マクロをを呼び出し、メッセージを表示してプログラムを終了させます。

```rust
    let last_name = "yamada";
    let first_name = "taro";
    let full_name = build_full_name(last_name, first_name).unwrap();
```
成功時はOkの中身を表示します。

```rust
let last_name = "yamada";
let first_name = "taro";
let full_name = build_full_name(last_name, first_name).unwrap();
```
失敗時は、これ以降の処理は何も実行されません。なので`unwrap`の使用には注意する必要があります。

unwrapの中をmatchで書き直すと下記のようになります。
`match`を短くしただけです。
```rust
result.unwrap()
↓
match result {
    Ok(v) => v,
    Err(e) => panic!(...),
}
```

## expect 
`expect`も`unwrap`と同じくエラー時に`panic`させます。
`unwrap`と違うのは、引数にエラーメッセージを渡せます。

```rust
let last_name = "yamada";
let first_name = "";
let full_name = build_full_name(last_name, first_name).expect("名前の作成に失敗しました");
```

## unwrap_or_else
エラー時にはクロージャーを呼び出します。

```rust
let last_name = "yamada";
let first_name = "";

let ret = build_full_name(last_name, first_name).unwrap_or_else(|error| {
    println!("エラー: {}", error);
    String::from("名前の作成に失敗しました")
});

println!("{}", ret);
```
エラー時の処理を自由に書けるので、使い勝手が良さそうです。

## and_then
こちらはOk時にクロージャーを呼び出します。

```rust
let last_name = "yamada";
let first_name = "taro";
let ret = build_full_name(last_name, first_name)
    .and_then(|full_name| {
        println!("Full Name: {}", full_name);
        Ok(full_name)
    })
    .unwrap();
println!("{}", ret);
```
`and_then`は`Result`を返すので、Stringにするために`.unwrap()`しています。


## ?
`?`はエラーの伝搬をします。

```rust
fn function() {
    let last_name = "yamada";
    let first_name = "";
    let full_name = build_full_name(last_name, first_name)?;
    println!("Full Name: {}", full_name);
}
```

Resultの中身を取り出せるのですが、
```bash
the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
```
`result`、`option`を返す関数でしか使えないというエラーが出ます。

つまり以下のようにする必要があります。
```rust
fn function() -> Result<String, String> {
    let last_name = "yamada";
    let first_name = "";
    let full_name = build_full_name(last_name, first_name)?;
    Ok(full_name)
}
```

これを実行させてみます。
```rust
fn main() {
    let ret = function().unwrap();
    println!("{}", ret);
}

fn function() -> Result<String, String> {
    let last_name = "yamada";
    let first_name = "";
    let full_name = build_full_name(last_name, first_name)?;
    Ok(full_name)
}
```
今回はエラーを返しているので、`unwrap`でパニックになります。




