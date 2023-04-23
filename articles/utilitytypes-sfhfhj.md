---
title: '【TypeScript】Utility Types ハックして自作の型を作ってみた'
emoji: '🦍'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['typescript']
published: true
publication_name: 'gohan_dao'
---

# Utility Types とは

https://www.typescriptlang.org/docs/handbook/utility-types.html

```ts
type User = {
  name: string;
  age: number;
  address: string;
  phoneNumber: string;
};

type PartialUser = Partial<User>;

// PartialUserの型
PartialUser = {
  name?: string;
  age?: number;
  address?: string;
  phoneNumber?: string;
};
```

上記の例のように、`Utility Types`とはコード内で型変換を簡単にしてくれます。
ただ、`phoneNumber`のみオプショナルにしたいとき、それに相当する`Utility Types`は存在しないので、組み合わせて作ってみます。

# 自作 UtilityTypes を作る

`Partial`、`Pick`と`Omit`を組み合わせて作ります。

```ts
type customPartial<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
```

実際の使い方は以下のようになります。

```ts
type User = {
  name: string;
  age: number;
  address: string;
  phoneNumber: string;
};

type PartialUser = customPartial<User, 'phoneNumber'>;

// PartialUserの型
PartialUser = {
  name: string;
  age: number;
  address: string;
  phoneNumber?: string;
};

// phoneNumberはオプショナルなのでなくても良い
const user: PartialUser = {
  name: 'hoge',
  age: 20,
  address: 'fuga',
};
```

## 解説

```ts
type customPartial<T, K extends keyof T>
```

の部分から見ていきます。
T は User が入ります。`keyof T`は`name`、`age`、`address`、`phoneNumber`の 4 つの文字列の Union 型になります。つまり、`K`は`name`、`age`、`address`、`phoneNumber`のいずれかの文字列になります。

次に、`Omit`の部分です。

```ts
Omit<T, K>;
```

Omit は`T`から`K`を除外した型を返します。
例えば

```ts
type User = {
  name: string;
  age: number;
  address: string;
  phoneNumber: string;
};

type OmitUser = Omit<User, 'name' | 'age'>;
```

のようにすると、`OmitUser`の型は

```ts
OmitUser = {
  address: string;
  phoneNumber: string;
}
```

となります。

次に、`Partial`と`Pick`の部分です。

```ts
Partial<Pick<T, K>>;
```

`Partial`は`T`の全てのプロパティをオプショナルにします。`Pick`は`T`から`K`を選択した型を返します。
そして選択された型を`Partial`でオプショナルにします。

例えば

```ts
type User = {
  name: string;
  age: number;
  address: string;
  phoneNumber: string;
};

type PickAndPartialUser = Partial<Pick<User, 'name' | 'age'>>;
```

のようにすると、`PartialUser`の型は

```ts
type PickAndPartialUser = {
  name?: string;
  age?: number;
};
```

となります。

最後にこれらを組み合わせると、

```ts
type PartialUser = {
  name?: string;
  age?: number;
  address: string;
  phoneNumber: string;
};
```

となります。

これにて utilitytypes のハック完了です。

## オプショナルをとるパターン

逆に必須にしるパターンも作ってみます。

```ts
type customRequired<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;
```

同じ考えでできますね。
