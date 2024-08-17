---
title: "Rust で Client(API Wrapper) を作る手順"
emoji: "🦍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust", "API"]
published: false
---

## 初めに
Rust で API クライアントを作る手順をまとめました。
Rust はまだまだ、プロダクションで使われることが少ないので、それに伴って、ライブラリも少ないです。
逆に言えば、ライブラリを作るチャンスが多いということです。
自作OSSを作ったことない人でも、とっつきやすいと思います。

# APIを選ぶ
まずは、API を選びます。
公開 API がまとまっている[サイト](https://apidog.com/apihub/)とかあるので、そこから選ぶといいかもしれません。
今回私は、[gyazo api](https://gyazo.com/api/docs/image)を選びました。
理由は、Scrapbox が好きなこと(関係ない)と書き込み API が存在したためです。

## ライブラリを作る
`Gyazo API` には、公開 API のエンドポイントが 5 つあリます。
今回は、`Image` と `Upload` の 2 つを実装します。
### Client を作る
まずは、Client を作ります。
私は、`GyazoClient`と命名しました。

```rust
pub struct GyazoClient {
    client: reqwest::Client,
    access_token: String,
}
```
つぎに、このクライアントに対して、`new`メソッドを実装します。

```rust
impl GyazoClient {
    pub fn new(access_token: String) -> Self {
        Self {
            client: reqwest::Client::new(),
            access_token,
        }
    }
}
```
他の言語でいうところのコンストラクタです。
あとは、`Image` と `Upload` のメソッドを仮で実装します。

```rust
impl GyazoClient {
    pub async fn get_image(&self) -> Result<(), ()> {
        todo!()
    }

    pub async fn upload_image(&self) -> Result<(), ()> {
        todo!()
    }
}
```
これで、枠組みは完成です。

### Input の定義をする
次に、Input の定義をします。
API にリクエストする際に、必要なパラメータを定義します。
#### Image
まずは、`Image` のリクエストをする際に必要なパラメータを定義します。
Image はとても簡単です。API 仕様書を見てみましょう
```bash
GET https://api.gyazo.com/api/images
```
| Key | Type | Required |
| --- | --- | --- |
| access_token | String | true |
| image_id | String | true |

access_token は `GyazoClient` で定義しているので、`image_id` だけを定義します。

```rust 
#[derive(Debug, Serialize)]
pub struct ImageRequest {
    image_id: String,
}
```
これだけであれば、わざわざ struct を定義する必要はないので、`get_image` メソッドの引数に直接書きます。

```rust
pub async fn get_image(&self, image_id: &str) -> Result<(), ()> {
    todo!()
}
```

#### Upload
次に、`Upload` のリクエストをする際に必要なパラメータを定義します。
同じように、API 仕様書を見てみましょう
```bash
POST https://upload.gyazo.com/api/upload
```
| Key | Type | Required |
| --- | --- | --- |
| access_token | String | true |
| imagedata | Binary | true |
| access_policy | String | false |
| metadata_is_public | String | false |
| referer_url | String | false |
| app | String | false |
| title | String | false |
| desc | String | false |
| created_at | Float | false |
| collection_id | String | false |

かなり多いですね。
単純に定義すると以下のようになります。

```rust
#[derive(Debug)]
pub struct UploadParams {
    pub imagedata: Vec<u8>,
    pub access_policy: Option<String>,
    pub metadata_is_public: Option<String>,
    pub referer_url: Option<String>,
    pub app: Option<String>,
    pub title: Option<String>,
    pub desc: Option<String>,
    pub created_at: Option<String>,
    pub collection_id: Option<String>,
}
```
これをメソッドの引数にしても、問題ないですが、引数が多くて、使う側は必要のない引数も書かないといけません。
これを解決するために、Builderパターン を使います。
イメージとしては、必要なパラメータだけを書いて、ビルドすると、`UploadParams` が生成されるというものです。

```rust
#[derive(Debug)]
pub struct UploadParamsBuilder {
    imagedata: Vec<u8>,
    access_policy: Option<String>,
    metadata_is_public: Option<String>,
    referer_url: Option<String>,
    app: Option<String>,
    title: Option<String>,
    desc: Option<String>,
    created_at: Option<String>,
    collection_id: Option<String>,
}

impl UploadParamsBuilder {
    pub fn new(imagedata: Vec<u8>) -> Self {
        Self {
            imagedata,
            access_policy: None,
            metadata_is_public: None,
            referer_url: None,
            app: None,
            title: None,
            desc: None,
            created_at: None,
            collection_id: None,
        }
    }

    pub fn access_policy(mut self, access_policy: impl Into<String>) -> Result<Self, GyazoError> {
        let access_policy = access_policy.into();
        if access_policy != "anyone" && access_policy != "only_me" {
            return Err(GyazoError::InvalidInput(
                "access_policy must be 'anyone' or 'only_me'".to_string(),
            ));
        }
        self.access_policy = Some(access_policy);
        Ok(self)
    }

    pub fn metadata_is_public(
        mut self,
        metadata_is_public: impl Into<String>,
    ) -> Result<Self, GyazoError> {
        let metadata_is_public = metadata_is_public.into();
        if metadata_is_public != "true" && metadata_is_public != "false" {
            return Err(GyazoError::InvalidInput(
                "metadata_is_public must be 'true' or 'false'".to_string(),
            ));
        }
        self.metadata_is_public = Some(metadata_is_public);
        Ok(self)
    }

    pub fn referer_url(mut self, referer_url: impl Into<String>) -> Self {
        self.referer_url = Some(referer_url.into());
        self
    }

    pub fn app(mut self, app: impl Into<String>) -> Self {
        self.app = Some(app.into());
        self
    }

    pub fn title(mut self, title: impl Into<String>) -> Self {
        self.title = Some(title.into());
        self
    }

    pub fn desc(mut self, desc: impl Into<String>) -> Self {
        self.desc = Some(desc.into());
        self
    }

    pub fn created_at(mut self, created_at: impl Into<String>) -> Self {
        self.created_at = Some(created_at.into());
        self
    }

    pub fn collection_id(mut self, collection_id: impl Into<String>) -> Self {
        self.collection_id = Some(collection_id.into());
        self
    }

    pub fn build(self) -> Result<UploadParams, GyazoError> {
        Ok(UploadParams {
            imagedata: self.imagedata,
            access_policy: self.access_policy,
            metadata_is_public: self.metadata_is_public,
            referer_url: self.referer_url,
            app: self.app,
            title: self.title,
            desc: self.desc,
            created_at: self.created_at,
            collection_id: self.collection_id,
        })
    }
}
```
`imagedata` 以外は、オプショナルなので、初期値に `None` を入れています。
各メソッドが呼び出されたタイミングで必要なのもは、検証を行い、`self` に値を入れています。
さいごに、`build` メソッドで、`UploadParams` を生成しています。
※ `GyazoError` は、エラーの定義をする際に使います。後述します。
使い方は以下のようになります。

```rust
let params = UploadParamsBuilder::new(imagedata)
    .access_policy("anyone")
    .metadata_is_public("true")
    .build()?

let gyazo_client = GyazoClient::new("your_access_token".to_string());
gyazo_client.upload_image(params).await?;
```

##### nits
rust では引数にトレイトを使うことができます。
例えば以下の関数では、`Summary` トレイトを実装しているものを引数に取ることができます。
```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```
```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct NewsArticle {
    headline: String,
    location: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.headline, self.location)
    }
}

struct Tweet {
    username: String,
    content: String,
}

pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
};

let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
};

// Summary トレイトを実装しているので、引数に渡すことができる
notify(&article);
// Summary トレイトを実装していないので、コンパイルエラー
notify(&tweet);
```
上の例でわかるように、`Into<String>` トレイトを実装しているものを引数に取ることができます。
なので、String に変換できるものを引数に取ることができます。
`String` と `&str` は `Into<String>` トレイトを実装しているので、引数に取ることができます。
少し大袈裟に書くと、
```rust
trait Into<T> {
    fn into(self) -> T;
}

struct FirstName {
    name: String,
}

struct LastName {
    name: String,
}

impl Into<String> for FirstName {
    fn into(self) -> String {
        self.name
    }
}

fn print_name(name: impl Into<String>) {
    println!("{}", name.into());
}

let first_name = FirstName {
    name: String::from("John"),
};

let last_name = LastName {
    name: String::from("Doe"),
};

// FirstName は Into<String> トレイトを実装しているので、引数に取ることができる
print_name(first_name);
// LastName は Into<String> トレイトを実装していないので、コンパイルエラー
print_name(last_name);
```
実際には、`Into` トレイトは `Rust` の標準ライブラリにあるので、少し形が違いますが、説明のために簡略化しています。

### Output の定義をする
あまり説明することがないので、`Image` のみを定義します。
```rust
#[derive(Debug, Deserialize)]
pub struct GyazoImageResponse {
    pub image_id: String,
    pub permalink_url: Option<String>,
    pub thumb_url: Option<String>,
    #[serde(rename = "type")]
    pub image_type: String,
    pub created_at: String,
    pub metadata: ImageMetadata,
    pub ocr: Option<ImageOcr>,
}
```
`serde` を使って、API のレスポンスをデシリアライズしています。
`json` 形式から、`GyazoImageResponse` に変換しています。
`#[serde(rename = "type")]` は、`type` というフィールド名が予約語なので、`image_type` に変換しています。

このレスポンスを使って、`get_image` メソッドの戻り値にします。

```rust
pub async fn get_image(&self, image_id: &str) -> Result<GyazoImageResponse, ()> {
    todo!()
}
```

### エラーの定義をする
### テストを書く

## まとめ
Rust で API クライアントを作る手順をまとめました。

