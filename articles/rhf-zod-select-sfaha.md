---
title: 'React Hook Form + zodでselectフォームを作る際のはまりどころ'
emoji: '👋'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# select を使うと型が合わない

## 一見問題なさそうなコード

```tsx
const schema = z.object({
  cost: z.number(),
});

type Schema = z.infer<typeof schema>;

const COSTS = [1000, 2000, 3000, 5000, 10000] as const;
const Form = () => {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<Schema>({
    resolver: zodResolver(schema),
    defaultValues: {
      cost: 0,
    },
  });
  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <select {...register('cost')}>
        {COSTS.map((cost) => (
          <option key={cost} value={cost}>
            {cost}
          </option>
        ))}
      </select>
      <button type="submit">送信</button>
    </form>
  );
};

export default Form;
```

このコードだとデフォルト値の場合はエラーが起きませんが、select で選択した値を送信するとエラーが起きます。
watch を使ってみると、選択した値は数値型ではなく文字列型になっていることがわかります。

```tsx
console.log(watch('cost'));
```

## なぜか

select の value は文字列型になるため。

```tsx
<option value={1}>React</option>
```

value に number を入れても文字列型になってしまいます。厄介ですね。

# 解決策１

## valueAsNumber を使う

https://react-hook-form.com/api/useform/register#options
RHF の`register`のオプションに存在します。
上記のプログラムを修正してみます。

```tsx
 <select {...register('cost', { valueAsNumber: true })}>
```

number 型になったことにより、バリテーションを通過するようになりました。

#　解決策２

## zod の transform を使う

https://zod.dev/?id=transform

string として受け入れてから number に変換することで解決できます。
zod の schema を修正してみます。

```tsx
const schema = z.object({
  cost: z.string().transform((val) => Number(val)),
});
```

# まとめ

RHF と zod を使って select フォームを作る際には、number へのキャストが必要なことがわかりました。
RHF にはハマりどころが多いので、理解しながらうまく付き合っていきたいですね！
