---
title: "Rust で Client(API Wrapper) を作る手順"
emoji: "🦍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rust", "API"]
published: false
---

## 初めに
`Rust SDK` がないサービスは結構あるので、`Rust` を採用した場合に、自作することが多いです。有名なもので言うと、`Firebase` や `Supabase` も 公式の　`Rust SDK` がないです。
この記事では、`Rust` で API クライアントを作る手順を紹介します。

## APIを選ぶ
まずは、API を選びます。
公開 API がまとまっている[サイト](https://apidog.com/apihub/)とかあるので、そこから選ぶといいかもしれません。
`Rust SDK` があまりないと上で述べましたが、とても有名なサービスで API を公開しているものは、有志で作っていることが結構あります。
[notion](https://github.com/jakeswenson/notion) とか [spotify](https://github.com/ramsayleung/rspotify) とかは、有志で作られています。

今回私は、[gyazo](https://gyazo.com/api/docs/image)を選びました。
理由は、Scrapbox が好きなこと(関係ない)と書き込み API が存在したためです。

## ライブラリ(クレート)を作る
`Gyazo API` には、公開 API のエンドポイントが複数あります。
今回は、`Image` と `Upload` の 2 つを実装します。

### Client を作る
まずは、`Client` を作ります。
私は、`GyazoClient`と命名しました。

```rust
pub struct GyazoClient {
    client: reqwest::Client,
    access_token: String,
}
```
つぎに、このクライアントに対して、`new` メソッドを実装します。

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
次に、`Input` の定義をします。
API にリクエストする際に、必要なパラメータを定義します。

#### Image
まずは、`Image` のリクエストをする際に必要なパラメータを定義します。
`Image` はとても簡単です。API 仕様書を見てみましょう

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

これだけであれば、わざわざ `struct` を定義する必要はないので、`get_image` メソッドの引数に直接書きます。

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
最後、`build` メソッドで、`UploadParams` を生成しています。
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

##### 補足：トレイトについて
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
次に、エラーの定義をします。
| code | Description |
| --- | --- |
| 200 | Success |
| 400 | リクエストパラメータが不正なときにこの値が返ります。 |
| 401 | ユーザーの認証が必要なときにこの値が返ります。 |
| 403 | アクセスする権限がないときにこの値が返ります。 |
| 404 | Not found |
| 422 | リクエストパラメータが文法的には正しいがサーバーで処理できないときにこの値が返ります。 |
| 429 | Rate limiting |
| 500 | Unexpected internal error |

```rust
#[derive(Error, Debug)]
pub enum GyazoError {
    #[error("HTTP request failed: {0}")]
    RequestFailed(#[from] reqwest::Error),
    #[error("Failed to parse JSON: {0}")]
    JsonParseError(#[from] serde_json::Error),
    #[error("Bad Request: Invalid request parameters")]
    BadRequest,
    #[error("Unauthorized: Authentication required")]
    Unauthorized,
    #[error("Forbidden: Access denied")]
    Forbidden,
    #[error("Not Found")]
    NotFound,
    #[error("Unprocessable Entity: Server cannot process the request")]
    UnprocessableEntity,
    #[error("Too Many Requests: Rate limit exceeded")]
    RateLimitExceeded,
    #[error("Internal Server Error: Unexpected error occurred")]
    InternalServerError,
    #[error("API error: {status}, message: {message}")]
    ApiError { status: StatusCode, message: String },
    #[error("Unexpected error: {0}")]
    Other(String),
    #[error("Invalid input: {0}")]
    InvalidInput(String),
    #[error("Invalid url: {0}")]
    InvalidUrl(String),
}
```
`GyazoError` というエラーを定義しています。
`thiserror` というクレートを使っていますが、ここでは、使い方は省略します。
全てのエラーを `GyazoError` にまとめていますが、分けてもいいかもしれません。

### API にリクエストを送る
いよいよ、API にリクエストを送ります。
#### Image
`Image` のリクエストを送る際には、`GET` メソッドを使います。
`reqwest` クレートを使って、リクエストを送ります。

```rust
pub async fn get_image(&self, image_id: &str) -> Result<GyazoImageResponse, GyazoError> {
    let url = format!("https://api.gyazo.com/api/images/{}", image_id);
    let response = self
        .client
        .get(&url)
        .header("Authorization", format!("Bearer {}", self.access_token))
        .send()
        .await?;

    match response.status() {
        StatusCode::OK => {
            let response = response.json::<GyazoImageResponse>().await?;
            Ok(response)
        }
        StatusCode::BAD_REQUEST => Err(GyazoError::BadRequest),
        StatusCode::UNAUTHORIZED => Err(GyazoError::Unauthorized),
        StatusCode::FORBIDDEN => Err(GyazoError::Forbidden),
        StatusCode::NOT_FOUND => Err(GyazoError::NotFound),
        StatusCode::UNPROCESSABLE_ENTITY => Err(GyazoError::UnprocessableEntity),
        StatusCode::TOO_MANY_REQUESTS => Err(GyazoError::RateLimitExceeded),
        StatusCode::INTERNAL_SERVER_ERROR => Err(GyazoError::InternalServerError),
        status => Err(GyazoError::Other(format!("Unexpected status code: {}", status))),
    }
}
```
戻り値には、上で定義した `GyazoImageResponse` と `GyazoError` を使っています。
`reqwest` でリクエストを送り、`response.status()` でステータスコードを取得しています。
ステータスコードによって、エラーを返すか、成功した場合は、`GyazoImageResponse` を返します。
`serde` を使って、`json` 形式から `GyazoImageResponse` に変換しています。
`Image` は結構シンプルですね。

#### Upload
`Upload` のリクエストを送る際には、リクエストボディに必要なパラメータを入れて、`POST` メソッドを使います。
`Content-Type` は `multipart/form-data` です。
当然、`reqwest` も `multipart/form-data` に対応しています。
https://docs.rs/reqwest/0.11.27/reqwest/blocking/multipart/index.html

まずは `UploadParams` に `From` トレイトを実装します。

```rust
impl From<UploadParams> for reqwest::multipart::Form {
    fn from(params: UploadParams) -> Self {
        let mut form = reqwest::multipart::Form::new().part(
            "imagedata",
            reqwest::multipart::Part::bytes(params.imagedata).file_name("image.png"),
        );
        form = form.text(
            "access_policy",
            params.access_policy.unwrap_or_else(|| "anyone".to_string()),
        );
        if let Some(metadata_is_public) = params.metadata_is_public {
            form = form.text("metadata_is_public", metadata_is_public);
        }
        if let Some(referer_url) = params.referer_url {
            form = form.text("referer_url", referer_url);
        }
        if let Some(app) = params.app {
            form = form.text("app", app);
        }
        if let Some(title) = params.title {
            form = form.text("title", title);
        }
        if let Some(desc) = params.desc {
            form = form.text("desc", desc);
        }
        if let Some(created_at) = params.created_at {
            form = form.text("created_at", created_at);
        }
        if let Some(collection_id) = params.collection_id {
            form = form.text("collection_id", collection_id);
        }
        form
    }
}
```
これで、`UploadParams` を `reqwest::multipart::Form` に変換できるようになりました。

次に、`Upload` メソッドを実装します。

```rust
pub async fn upload_image(
    &self,
    params: UploadParams,
) -> Result<UploadImageResponse, GyazoError> {
    let form = param.into();
    let response = self
        .client
        .post("https://upload.gyazo.com/api/upload")
        .header("Authorization", format!("Bearer {}", self.access_token))
        .multipart(form)
        .send()
        .await?;

    match response.status() {
        StatusCode::OK => {
            let response = response.json::<UploadImageResponse>().await?;
            Ok(response)
        }
        StatusCode::BAD_REQUEST => Err(GyazoError::BadRequest),
        StatusCode::UNAUTHORIZED => Err(GyazoError::Unauthorized),
        StatusCode::FORBIDDEN => Err(GyazoError::Forbidden),
        StatusCode::NOT_FOUND => Err(GyazoError::NotFound),
        StatusCode::UNPROCESSABLE_ENTITY => Err(GyazoError::UnprocessableEntity),
        StatusCode::TOO_MANY_REQUESTS => Err(GyazoError::RateLimitExceeded),
        StatusCode::INTERNAL_SERVER_ERROR => Err(GyazoError::InternalServerError),
        status => Err(GyazoError::Other(format!("Unexpected status code: {}", status))),
    }
}
```
`UploadParams` を `reqwest::multipart::Form` に変換して、`multipart` メソッドに渡しています。
これで、`Upload` メソッドが完成しました。

### テストを書く
最後に、テストを書きます。
API のテストは、`mockito` クレートを使って、モックサーバーを立ててテストします。
モックする理由は、テスト時に毎回リクエストを送ると、API に負荷がかかったり、上限に引っかかる可能性があるためです。
ただし、ここで問題があります。
上記のようなコードだと、エンドポイントを固定してしまっているため、モックサーバーを立てても、そこにリクエストを送ることができません。
まずは、`GyazoClient` にエンドポイントを引数に取るように変更します。

```rust
#[derive(Clone, Debug)]
pub struct GyazoClient {
    client: Client,
    access_token: String,
    base_url: Url,
    upload_url: Url,
}

#[derive(Default, Clone, Debug)]
pub struct GyazoClientOptions {
    pub access_token: String,
    pub base_url: Option<String>,
    pub upload_url: Option<String>,
}

impl GyazoClient {
    pub fn new(options: GyazoClientOptions) -> Self {
        let base_url = options
            .base_url
            .map(|url| Url::parse(&url).expect("base_url must be a valid URL"))
            .unwrap_or_else(|| Url::parse(DEFAULT_BASE_URL).expect("base_url must be a valid URL"));
        let upload_url = options
            .upload_url
            .map(|url| Url::parse(&url).expect("upload_url must be a valid URL"))
            .unwrap_or_else(|| {
                Url::parse(DEFAULT_UPLOAD_URL).expect("upload_url must be a valid URL")
            });
        GyazoClient {
            client: Client::new(),
            access_token: options.access_token,
            base_url,
            upload_url,
        }
    }
}
```
`GyazoClientOptions` という構造体を作り、`base_url` と `upload_url` を追加しました。
テスト時には、`base_url` と `upload_url` をモックサーバーのエンドポイントに変更します。
実際に `GyazoClient` を使う時、いちいち、`None` を定義するのはあまりいけていないので、`Default` トレイトを実装して、デフォルト値を設定できるようにしました。

```rust
let gyazo_client = GyazoClient::new("YOUR_ACCESS_TOKEN".to_string(), ..Default::default());
```

これで、テストを書く準備ができました。
`mockito` を使って、モックサーバーを立てて、テストします。

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use mockito::Matcher;

    #[tokio::test]
    async fn test_get_image() -> anyhow::Result<()> {
        let mut server = mockito::Server::new_async().await;
        let mock_response = r#"
        {
            "image_id": "abc123",
            "permalink_url": "https://gyazo.com/abc123",
            "thumb_url": "https://thumb.gyazo.com/thumb/abc123",
            "type": "png",
            "created_at": "2024-08-10 12:00:00",
            "metadata": {
                "app": null,
                "title": null,
                "url": null,
                "desc": null
            },
            "ocr": null
        }
        "#;

        server
            .mock("GET", "/api/images/abc123")
            .match_header("Authorization", Matcher::Regex("Bearer .+".to_string()))
            .with_status(200)
            .with_header("content-type", "application/json")
            .with_body(mock_response)
            .create();

        let client = GyazoClient::new(GyazoClientOptions {
            access_token: "fake_token".to_string(),
            base_url: Some(server.url().to_string()),
            upload_url: None,
        });
        let result = client.get_image("abc123").await;

        assert!(result.is_ok());
        let image = result?;
        assert_eq!(image.image_id, "abc123");
        assert_eq!(
            image.permalink_url,
            Some("https://gyazo.com/abc123".to_string())
        );
        Ok(())
    }
}
```
あとは、比較的簡単です。
1. `mockito` でモックサーバーを立てます。
2. `GyazoClient` に、モックサーバーのエンドポイントを渡して、リクエストを送ります。
3. `assert_eq!` で、期待する値と実際の値を比較しています。

`Rust` には、ドキュメントテストというものがあるので、それを使って、テストを書いてもいいかもしれません。
https://doc.rust-jp.rs/rust-by-example-ja/testing/doc_testing.html

あと、共有処理をまとめたり、テストを追加したものが、ありますので、参考にしてみてください。
https://github.com/katayama8000/gyazo-client-rust



## まとめ
`Rust` で API クライアントを作る手順をまとめました。この記事で、誰かの `OSS` 活動の第一歩になれば幸いです。
また、弊社では `Firebase` をバックエンドに使用しており、`Rust` で `Firebase` のクライアントも開発しています。仕事を通じて `OSS` 活動ができるのは、なかなか良い環境だと思います。

現在、エンジニアを絶賛募集中です！カジュアルな面談からでも大歓迎ですので、ぜひお気軽にご連絡ください！
