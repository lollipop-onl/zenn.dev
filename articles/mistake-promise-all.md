---
title: 誤用しがちな Promise.all
emoji: 🍬
type: tech
topics: [JavaScript, Promise]
published: true
---

非同期処理を並列実行するときに便利な `Promise.all()` ですが、雑に使ってしまうと意図した挙動にならないことがあります。

# まとめ

1. `Promise.all` 内で関数を定義する際は即時実行関数式にしなければ実行されない
2. `Promise.all` に渡す際に `await` してしまうと意図せず直列処理になってしまう

# サンプルコード

以下のコードで例示した①・②・③・④のうち、期待した挙動にならないものがあります。  
（いずれの実行もエラーにならないものとします）

```ts
let response;

const syncFn = () => {};
const asyncFn = async () => {};

const results = await Promise.all([
  // ①
  syncFn();
  
  // ②
  asyncFn(),

  // ③
  async () => {
    const data = await fetchData();
    await nextProcess(data);
  },

  // ④
  await asyncFn(),
]);
```

このうち、③・④は意図した挙動になりません。

# Promise.all には Promise を渡す

`Promise.all` は **待機状態の Promise** を渡すことで、それぞれの Promise が解決されることを並行で待機します。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/all

ここで注意すべきことは「渡された関数を実行するわけではない」ということです。  
そのため、サンプルコード ③ はそもそも何も起こりません。

```ts
const results = await Promise.all([
  // 関数は実行されない
  async () => {
    const data = await fetchData();
    await nextProcess(data);
  },
]);
```

このコードでは `Promise` ではなく、関数自体が渡されているため、結果としても関数が返されます。

```ts
results[0] === async () => {
  const data = await fetchData();
  await nextProcess(data);
}
```

`Promise.all` の引数内で関数を定義したい場合は、即時実行関数式で記述する必要があります。

```ts
const results = await Promise.all([
  // 関数内の処理が実行される
  (async () => {
    const data = await fetchData();
    await nextProcess(data);
  })(),
]);
```

# 引数内で Promise の解決を待機させない

サンプルコード ④ では、`Promise.all` の引数内で Promise を `await` してしまっています。  
このとき、実際に `Promise.all` に渡されるのは `asyncFn()` 関数の返り値なので、待機中の Promise ではありません。

```ts
const results = await Promise.all([
  await asyncFn(),
]);
```

たとえば、以下のコードは３つの非同期関数を並行実行しているように見えて、それぞれの関数が渡される際に Promise を解決してしまうため、結果的には直列実行になってしまいます。

```ts
const sleep30s = async () => { await sleep(30); console.log('sleep 30s'); };
const sleep100s = async () => { await sleep(100); console.log('sleep 100s'); };
const sleep60s = async () => { await sleep(60); console.log('sleep 60s'); };

await Promise.all([
  await sleep30s(),
  await sleep100s(),
  await sleep60s(),
]);

// 実行結果：
//   sleep 30s
//   sleep 100s
//   sleep 60s
```

並列処理にするには、非同期関数が返す待機状態の Promise を `Promise.all` に渡す必要があるため、 `await` させてはいけません。

```ts
const sleep30s = async () => { await sleep(30); console.log('sleep 30s'); };
const sleep100s = async () => { await sleep(100); console.log('sleep 100s'); };
const sleep60s = async () => { await sleep(60); console.log('sleep 60s'); };

await Promise.all([
  sleep30s(),
  sleep100s(),
  sleep60s(),
]);

// 実行結果：
//   sleep 30s
//   sleep 60s
//   sleep 100s
```
