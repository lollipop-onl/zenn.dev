---
title: TS で Optional Chaining と Nullish Coalescing はどう出力されるか
emoji: 🍭
type: tech
topics: [typescript, EveOneZenn]
published: false
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) の２日目です。

TypeScript 3.7 で導入された Optional Chaining と Nullish Coalescing （ともに ES2020 で採用）がどのようにトランスパイルされるかをまとめます。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-ts-no-unchecked-indexed-access

# Optional Chaining と Nullish Coalescing について

Optional Chaining と Nullish Coalescing は TypeScript が ECMAScript に先行して 3.7 で導入したシンタックスです。
これらのシンタックスは ES2020 で正規に ECMAScript に採用されました。

## Optional Chaining

https://github.com/tc39/proposal-optional-chaining

Optional Chaining は `undefined` もしくは `null` な値のプロパティをランタイムで安全に参照できるようにするシンタックスです。

```ts
const foo: any = {};

foo.bar.baz;
// js: TypeError: Cannot read property 'baz' of undefined

foo.bar?.baz;
// js: undefined
```

## Nullish Coalescing

https://github.com/tc39/proposal-nullish-coalescing

Nullish Coalescing は値が `undefined` もしくは `null` のとき、フォールバックとして別の値を返せるようにするオペレータです。

```ts
undefined ?? 'fallback';
// js: 'fallback'

null ?? 'fallback';
// js: 'fallback'

'hello' ?? 'fallback';
// js: 'hello'
```

これまで `||` を使用していた初期値代入のうち、`0` や `false` といった Falsy な値を正常値として扱いたいときに Nullish Coalescing で置き換えられます。

```ts
0 || 'fallback';
// js: 'fallback'
0 ?? 'fallback';
// js: 0

false || 'fallback';
// js: 'fallback'
false ?? 'fallback';
// js: false
```

# TypeScript でのトランスパイル結果

Optional Chaining と Nullish Coalescing のトランスパイルは TypeScript のバージョンやコンパイルターゲットによって結果が異なります。

## TypeScript 3.8+

`TypeScript 3.8` 以上のバージョンで `target` を `ES2019` 以下に設定した場合、次のようにトランスパイルされます。

```ts
// ts: --------
const foo: Record<string, any> | undefined = {};

foo?.bar?.baz;
foo?.bar?.baz?.fiz;

foo.bar ?? 0;

// js: --------
var _a, _b, _c, _d;

const foo = {};

(_a = foo === null || foo === void 0 ? void 0 : foo.bar) === null || _a === void 0 ? void 0 : _a.baz;
(_c = (_b = foo === null || foo === void 0 ? void 0 : foo.bar) === null || _b === void 0 ? void 0 : _b.baz) === null || _c === void 0 ? void 0 : _c.fiz;

(_d = foo.bar) !== null && _d !== void 0 ? _d : 0;
```

Optional Chaining は対象のプロパティに対して厳密に `undefined` または `null` かを判定しています。
Nullish Coalescing もどうように厳密な判定をおこなっています。

また、プロパティごとに短い変数へ値を代入しコードを短縮しています。

なお、 `target` を `ES2020` または `ESNext` に設定するとトランスパイル後も同じシンタックスで出力されます。
Optional Chaining と Nullish Coalescing は ES2020 に含まれるので変換されません。

```ts
// ts: --------
const foo: Record<string, any> | undefined = {};

foo?.bar?.baz;
foo?.bar?.baz?.fiz;

foo.bar ?? 0;

// js: --------
const foo = {};

foo?.bar?.baz;
foo?.bar?.baz?.fiz;

foo.bar ?? 0;
```

## TypeScript 3.7

Optional Chaining と Nullish Coalescing が TypeScript に導入された 3.7 では、 Optional Chaining のトランスパイル結果が異なります。

```ts
// ts: --------
const foo: Record<string, any> | undefined = {};

foo?.bar?.baz;
foo?.bar?.baz?.fiz;

// js: --------
var _a, _b, _c, _d, _e;

const foo = {};

(_b = (_a = foo) === null || _a === void 0 ? void 0 : _a.bar) === null || _b === void 0 ? void 0 : _b.baz;
(_e = (_d = (_c = foo) === null || _c === void 0 ? void 0 : _c.bar) === null || _d === void 0 ? void 0 : _d.baz) === null || _e === void 0 ? void 0 : _e.fiz;
```

僅かな違いなので分かりづらいですが、 3.7 では `_a = foo` が含まれるのに対し、 3.8+ では `foo` はそのまま比較されています。

```ts
// 3.8+
(_a = foo === null || foo === void 0 ? void 0 : foo.bar) === null || _a === void 0 ? void 0 : _a.baz;
// 3.7
(_b = (_a = foo) === null || _a === void 0 ? void 0 : _a.bar) === null || _b === void 0 ? void 0 : _b.baz;
```

TypeScript 3.8 から Optional Chaining のトランスパイル結果がスマートになっているのがわかります。

# おわりに

Optional Chaining は便利ですが、重ねすぎるとトランスパイル後のコードが長大になってしまいます。

適切に早期リターンを使用するなどしてコードサイズの節約を心がけたいです。

# 参考

* [TypeScript: Documentation - TypeScript 3.7](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html)
