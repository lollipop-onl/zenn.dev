---
title: JavaScript で Invalid Date を判定する
emoji: 🍭
type: tech
topics: [javascript, EveOneZenn]
published: true
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.07 です。

JavaScript の Date クラスが返す Invalid Date を判定する方法についてまとめます。

（過去に別所で公開していた記事の加筆版です）

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-ts-ncoc-transpile

# Invalid Date とはなにか

`Invalid Date` とは、 JavaScript の Date クラスが返す、不正な日付を生成しようとしたときの値です。

```js
const d = new Date('aaa');

d;
// js: Invalid Date
```

このときの変数 `d` は Date クラスのインスタンスであるのは変わらないので、次のコードはすべてt `true` となります。

```js
const d = new Date('aaa');

typeof d === 'object';
// js: true
d instanceof Date;
// js: true
```

# Invalid Date を判定する

JavaScript には `Invalid Date` に相当する値がないため、比較による判定ができません。

また、 `Invalid Date` を変数に入れて比較しても常に `false` となり判定できません。

```js
const INVALID_DATE = new Date('invalid date');

INVALID_DATE;
// js: Invalid Date

new Date('aaa') === INVALID_DATE;
// js: false
```

## Number.isNaN を使用する

Date クラスのインスタンスで `valueOf` を実行すると、 `getTime()` メソッドと同様 Unix 時刻からの経過ミリ秒が返されます。

```js
new Date('2020-11-29').valueOf();
// js: 1606608000000

+new Date('2020-11-29');
// js: 1606608000000

new Date('2020-11-29').getTime();
// js: 1606608000000
```

`getTime()` メソッドは `Invalid Date` なインスタンスで実行すると `NaN` を返します。

```js
new Date('aaa').getTime();
// js: NaN
```

これらを利用して、`date.getTime()` の結果を `Number.isNaN` メソッドに渡し `true` が返されれば、その値は `Invalid Date` であったということが判定できます。

?> **2021/12/16**  
Number.isNaN(date) では正しく動作しない旨の指摘を受け、 isInvalidDate 関数の内容を修正しました。  
[latica](https://zenn.dev/latica)さんの[コメント](https://zenn.dev/link/comments/49d4790dcf9200)

```ts
const isInvalidDate = (date: Date) => Number.isNaN(date.getTime());

isInvalidDate(new Date('2020-11-29'));
// js: false

isInvalidDate(new Date('aaa'));
// js: true
```

# 参考

* [Date.prototype.getTime() - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Date/getTime)
* [UNIX時間 - Wikipedia](https://ja.wikipedia.org/wiki/UNIX%E6%99%82%E9%96%93)
