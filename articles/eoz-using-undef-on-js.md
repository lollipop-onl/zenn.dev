---
title: JavaScript で undefined の使用が推奨されない理由
emoji: 🍭
type: tech
topics: [javascript, EveOneZenn]
published: true
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.05 です。

JavaScript で `undefined` を値として使用することが推奨されない理由についてまとめます。

（過去に別所で公開していた記事の加筆版です）

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-ts-no-unchecked-indexed-access

# TypeScript Deep Drive の見解

TypeScript Deep Drive という TypeScript のバイブル的リファレンスに次のようなページがあります。

https://typescript-jp.gitbook.io/deep-dive/recap/null-undefined

このページでは undefined を値として使用するのを推奨していません。

たとえば、値が `undefined` かどうかを調べるには `null` との比較を推奨していたり、

```js
// 👎 Not good...
obj.prop === undefined;
 
// 👍 Better
obj.prop == null;
```

ルートレベルの変数が定義されているかを判断するには `typeof` の使用を推奨していたりします。

```js
// 👎 Not good...
globalObj === undefined;
 
// 👍 Better
typeof globalObj === 'undefined';
```

ただ、TypeSCript Deep Drive には `undefined` を使用するべきではない具体的な理由については言及されていません。

# ESLintルールの見解

同様の主張をしている ESLint の `no-undefined` ルールのドキュメントで具体的な理由が言及されていました。

https://eslint.org/docs/rules/no-undefined

説明の一部を抜粋します。

> `undefined` は代入可能なグローバルオブジェクトのプロパティです。 ECMAScript 3 では `undefined` を上書きすることができます。 ECMAScript 5 からグローバルの `undefined` を上書きできなくなりましたが（代入はできる）、スコープ変数として `undefined` をシャドーウィングすることができます。

つまり、 `undefined` 自体は代入可能な変数のため、実装に寄っては `undefined` に別の値が代入されることがあり得るということです。

次のコードは ES3 時代にグローバル変数として存在する `undefined` に異なる値を代入する例です。
ES5 以降ではグローバル変数の `undefined` への代入でエラーは発生しませんが、代入が反映されません。

```js
// ES3
window.undefined = 1;
// undefined === 1

const obj = {};

obj.prop === undefined; // false

// ES5
window.undefined = 1;
// undefined === undefined
```

また、次のコードは現在でも再現できる、スコープ変数として `undefined` を定義する例です。

```js
const foo = () => {
  const undefined = 1;
  // undefined === 1

  const obj = {};

  obj.prop = undefined; // false
};
```

`no-undefined` ルールでは `undefined` の使用に関するガイドラインを次のように提示しています。

* `undefined` は初期化されていない変数にのみ使用する
* 値が `undefined` かどうかの判定は `typeof` 演算子を使用する

なお、 `no-undefined` ルールは `recommended`・`eslint-config-airbnb`・`eslint-config-standard`・`eslint-config-google` のいずれのルールセットでも有効化されていません。

# `undefined` を保護する

代入やシャドーウィングから `undefined` を保護するには、ESLint の `no-global-assign` ルールと `no-shadow-restricted-names` を使用します。

https://eslint.org/docs/rules/no-global-assign

https://eslint.org/docs/rules/no-shadow-restricted-names

# おわりに

余談として、 `null` はプロパティ以外では代入も宣言もできない値なので、 `undefined` のような問題は発生しません。

```js
const null = 1;
// SyntaxError: Unexpected token 'null'

const foo = () => {
  const null = 1;
  // SyntaxError: Unexpected token 'null'
};

window.null = 1;
// null === null
// window.null === 1
```

# 参考

* [nullとundefined - TypeScript Deep Dive 日本語版](https://typescript-jp.gitbook.io/deep-dive/recap/null-undefined)
* [no-undefined - Rules - ESLint - Pluggable JavaScript linter](https://eslint.org/docs/rules/no-undefined)
* [no-global-assign - Rules - ESLint - Pluggable JavaScript linter](https://eslint.org/docs/rules/no-global-assign)
* [no-shadow-restricted-names - Rules - ESLint - Pluggable JavaScript linter](https://eslint.org/docs/rules/no-shadow-restricted-names)
