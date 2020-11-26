---
title: TypeScript 4.1 で追加された noUncheckedIndexedAccess とは何か
emoji: 🍭
type: tech
topics: [typescript, EveOneZenn]
published: true
---

# はじめに

この記事は #EveOneZenn (Everyday One Zenn) の４日目です。

TypeScript 4.1 で新たに追加された `--noUncheckedIndexedAccess` オプションを紹介します。

https://devblogs.microsoft.com/typescript/announcing-typescript-4-1

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-devtools-console-commands


# `--noUncheckedIndexedAccess` オプション

`--noUncheckedIndexedAccess` オプションは TypeScript 4.1 で追加された、プロパティへのアクセスをより厳密にするオプションです。

これまで `--strictNullChecks` オプションを有効にしていても、配列の要素やオブジェクトのプロパティへのアクセスが厳密な型安全ではありませんでした。

たとえば、次のコードでは JavaScript 的に `undefined` の可能性があるプロパティへアクセスしても型情報としては `undefined` が含まれていませんでした。

```ts
const obj: Record<string, string> = { foo: 'bar' };

obj.foo;
// js: 'bar'
// ts: string

foo.baz;
// js: undefined
// ts: string
```

この実際には `undefined` の可能性があるプロパティへのアクセスを厳密にするのが `--noUncheckedIndexedAccess` オプションです。

次のコードでオプションの有無で型推論がどのように変わるかを確認しましょう。

```ts
type Obj1 = Record<string, string>;

const obj1: Obj1 = {};

// すべてのプロパティで undefined が推論される
obj1.foo;
// js : undefined
// on : string | undefined
// off: string

type Obj2 = {
  foo: string;
  [propName: string]: string;
};

const obj2: Obj2 = { foo: 'hello', bar: 'world' };

// 必須のプロパティなので undefined は推論されない
obj2.foo;
// js : 'hello'
// on : string
// off: string

obj2.bar;
// js : 'world'
// on : string | undefined
// off: string

type Arr1 = string[];

const arr1: Arr1 = ['foo'];

// すべての要素で undefined が推論される
arr1[0];
// js : 'foo'
// on : string | undefined
// off: string

arr1[1];
// js : undefined
// on : string | undefined
// off: string

type Arr2 = [item: string, ...items: string[]];

const arr2: Arr2 = ['foo', 'bar'];

// 必須よ要素なので undefined は推論されない
arr2[0];
// js : 'foo'
// on : string
// off: string

arr2[1];
// js : 'bar'
// on : string | undefined
// off: string

arr2[2];
// js : undefined
// on : string | undefined
// off: string
```

なお、型情報上ではプロパティの値に `undefined` は含まれません。

```ts
type Obj1 = Record<string, string>;

type Foo = Obj1['foo'];
// ts: string

const obj1: Obj1 = { foo: 'bar' };
obj.foo;
// ts: string | undefined
```

また、繰り返し（`for in` や `forEach`）の中で要素を参照する場合、 `undefined` は推論されません。

```ts
type Obj1 = Record<string, string>;

const obj1: Obj1 = { foo: 'bar' };

Object.entries(obj1);
// js: [['foo', 'bar']]
// ts: [string, string][]

type Arr1 = string[];

const arr1: Arr1 = ['hello', 'world'];

arr1.forEach((item) => {
  item;
  // js: 'hello' | 'world'
  // ts: string
});
```

# 使ってみよう

`--noUncheckedIndexedAccess` オプションを有効にするには、 `tsconfig.json` の `compilerOptions` に `noUncheckedIndexedAccess: true ` を追加するか、コンパイル時のオプションとして `--noUncheckedIndexedAccess` を指定します。

```json:tsconfig.json
{
  "compilerOptions": {
    "noUncheckedIndexedAccess": true
  }
}
```

```sh
$ tsc ./index.ts --noUncheckedIndexedAccess
```

# 参考

* [Announcing TypeScript 4.1 - Checked Indexed Accesses (--noUncheckedIndexedAccess)](https://devblogs.microsoft.com/typescript/announcing-typescript-4-1/#checked-indexed-accesses-nouncheckedindexedaccess)
