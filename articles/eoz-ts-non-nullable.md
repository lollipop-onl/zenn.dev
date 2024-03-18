---
title: TypeScript で配列から nullable な要素を除去する
emoji: 🍭
type: tech
topics: [typescript, EveOneZenn]
published: true
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.13 です。

配列から nullable な要素を除去するフィルタを TypeScript で実装する方法をまとめます。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-nuxt-dev-memory-leak

# 2024/03/18 更新： TypeScript 5.5 でビルトインサポートになります

:::message
TypeScript 5.5 は 2024/06/18 にリリース予定です。

https://github.com/microsoft/TypeScript/issues/57475
:::

TypeScript 5.5 にて `Array.filter` による型の絞り込みがビルトインでサポートされるようになります。

https://github.com/microsoft/TypeScript/pull/57465

この記事で紹介している例も、 `nonNullable` に相当するユーザー定義の型ガードがなくても期待する型の絞り込みが行えます。

```ts
const arr = [0, 1, 'hello', false, null, undefined];

const result = arr.filter((item) => item != null);
// js: [0, 1, 'hello', false]
// ts: (string | number | boolean)[] 🎉
```

> [TypeScript Playground](https://www.typescriptlang.org/play?ts=5.5.0-dev.20240318&ssl=5&ssc=1&pln=5&pc=37#code/MYewdgzgLgBAhgJwTAvDA2gBgDQwIy4DkAFgKYA25IhuAZnORKbmAK6W6tgAmptAlmFLcAugG4AUBNCRYCUhHaw0iBADoB5KKQQAKXf20BbAJSoAfDEOkjMAIRo2lE5ID0rmACsIALgw58IjJKajoGJhEJdxgoXxhdaARBAHMYAB8YNiMAIx10mGyQEHJSODATdBEgA)

# 配列から nullable な値を除去する

JavaScript で配列から nullable （`undefined` と `null`） を除去する方法としては、 `Array.prototype.filter` メソッドで要素が nullable でないかチェックするのが一般的かと思います。

```js
const arr = [0, 1, 'hello', false, null, undefined];

arr.filter((item) => item != null);
// js: [0, 1, 'hello', false]
```

JavaScript ではこの実装で問題ありませんが、 TypeScript で同様のコードを実行すると型情報上に nullable が残ってしまいます。

```ts
const arr = [0, 1, 'hello', false, null, undefined];
// ts: (number | string | boolean | null | undefined)[]

arr.filter((item) => item != null);
// js: [0, 1, 'hello', false]
// ts: (number | string | boolean | null | undefined)[]
```

# 型情報からも nullable な要素を除去する

`Array.prototype.filter` メソッドのビルトイン型からの推論では、型情報から nullable を除去することができませんでした。

そこで、ユーザー定義の Type Guard を使用して型ガードを明示的に定義します。
ユーザー定義の Type Guard は `a is B` （ `a` が変数、 `B` が型）で変数 `a` の型を `B` と定義し直すことができます。

```ts
const arr = [0, 1, 'hello', false, null, undefined];
// ts: (number | string | boolean | null | undefined)[]

arr.filter((item): item is NonNullable<typeof item> => item != null);
// js: [0, 1, 'hello', false]
// ts: (number | string | boolean)[]
```

`NonNullable` 型は指定した型から `null` と `undefined` を除去するものです。

これで、ランタイムと型情報いずれからも nullable を除去するフィルタが作成できました。

# ユーティリティ化する

上記で紹介したフィルタ関数はユーティリティ化することができます。

```ts
const nonNullable = <T>(value: T): value is NonNullable<T> => value != null;
```

使い所としては例えば、 `Array.prototype.map` が `undefined` を返しうるとき、最終的に `undefined` を除いた配列を返したいケースです。

```ts
const users = [
  { id: 0, name: 'Hanako', active: true },
  { id: 1, name: 'Taro', active: false },
  { id: 2, name: 'Natsumi', active: true },
];

// active: true であるユーザーの名前の配列
const activeUserNames = users
  .map((user): string | undefined => {
    if (!user.active) {
      return;
    }

    return user.name;
  })
  .filter(nonNullable);
// js: ['Hanako', 'Natsumi']
// ts: string[]
```

# ユーザー定義の Type Guard を使う際の注意

ユーザー定義の Type Guard は実際の処理で返される値と型定義がずれることがあります。

たとえば、次のコードはフィルタ内で Falsy な要素を除去していますが、型定義上は nullable を除去していることになっています。
このコードはエラーが発生しません。

```ts
const arr = [0, 1, 'hello', false, null, undefined] as const;
// ts: (number | string | boolean | null | undefined)[]

arr.filter((item): item is NonNullable<typeof item> => !!item);
// js: [1, 'hello']
// ts: (number | string | boolean)[]
```

JavaScript ランタイムでは `false` や `0` も除去されていますが、型定義上では `boolean` も返ることになってしまっています。

このような実装と型の乖離を起こさないよう、ユーザー定義の Type Guard を使用する際は注意して実装しましょう。

# 参考

* [TypeScript: Documentation - Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
* [TypeScript: Documentation - Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html)
