---
title: "Node.js で backlog-js の HTTP リクエストにロガーを差し込む"
emoji: "🍬"
type: "tech"
topics: ["nodejs", "backlog"]
published: true
---

[backlog-js](https://github.com/nulab/backlog-js) が発行する HTTP リクエストを、ライブラリのソースコードを改変せずにデバッグログとして出力したくなりました。
backlog-js は内部で Node.js 組み込みの `fetch` を使っており、`fetch` の実体は [undici](https://github.com/nodejs/undici) です。undici の `Dispatcher.compose` と `setGlobalDispatcher` を組み合わせると、グローバルにインターセプターを挿入してリクエストをロギングできます。

なお、この方法は backlog-js に限らず、Node.js の `fetch` を利用しているライブラリ全般に応用できます。

## 前提

- Node.js >= 20.18
- undici >= 7.0.0

Node.js には undici がバンドルされていますが、`Dispatcher.compose` などの API をアプリケーションコードから利用するには undici パッケージを別途インストールする必要があります。

:::message
[isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch) を使っている場合は undici が利用されないため、この記事の方法は適用できません。
:::

## 仕組み

undici の `Agent` はデフォルトの HTTP ディスパッチャーです。[`Dispatcher.compose`](https://github.com/nodejs/undici/blob/main/docs/docs/api/Dispatcher.md) メソッドにインターセプター関数を渡すことで、dispatch の前後にロジックを挟めます。
作成したディスパッチャーを `setGlobalDispatcher` でグローバルに設定すると、Node.js 組み込みの `fetch` を含むすべての HTTP リクエストがこのディスパッチャーを経由するようになります。

1. `fetch("https://example.com/api")` が呼ばれる
2. `setGlobalDispatcher` で設定した `Agent` が dispatch を受け取る
3. `compose` で追加したインターセプターが実行される（ここでログ出力）
4. 実際の HTTP リクエストが送信される

## インターセプターの実装

インターセプターのシグネチャは以下のようになっています。

```ts
type Interceptor = (dispatch: Dispatcher["dispatch"]) => Dispatcher["dispatch"];

// Dispatcher["dispatch"] のシグネチャ
type Dispatch = (opts: DispatchOptions, handler: DispatchHandler) => boolean;
```

`dispatch` は次段の dispatch 関数で、`opts` にリクエスト情報、`handler` にレスポンスのコールバックが入ります。
インターセプターは `handler` を差し替えた上で `dispatch` を呼ぶことで、レスポンスの前後にロジックを挟めます。

### 基本の実装

まず最小限のロガーを実装します。リクエスト開始時にメソッドとパスを、レスポンス受信時にステータスコードと所要時間をログに出します。

```ts:http-logger.ts
import { Agent, type Dispatcher, setGlobalDispatcher } from "undici";

type Interceptor = (dispatch: Dispatcher["dispatch"]) => Dispatcher["dispatch"];

const createLoggingInterceptor = (): Interceptor => {
  let originLogged = false;

  // インターセプターは dispatch 関数を受け取り、新しい dispatch 関数を返す
  return (dispatch) => (opts, handler) => {
    // API ホストは初回リクエスト時に一度だけ表示
    const origin = opts.origin?.toString() ?? "";
    if (!originLogged && origin) {
      console.log(`API host: ${origin}`);
      originLogged = true;
    }

    // リクエスト情報を取得して開始時刻を記録
    const method = opts.method ?? "UNKNOWN";
    const path = opts.path ?? "";
    const start = performance.now();

    console.log(`→ ${method} ${path}`);

    // handler を差し替えてレスポンスのタイミングでもログを出す
    return dispatch(opts, {
      // 元の handler にそのまま委譲するコールバック
      onRequestStart(controller, context) {
        handler.onRequestStart?.(controller, context);
      },
      onRequestUpgrade(controller, statusCode, headers, socket) {
        handler.onRequestUpgrade?.(controller, statusCode, headers, socket);
      },
      // レスポンスを受信したらステータスコードと所要時間をログに出す
      onResponseStart(controller, statusCode, headers, statusMessage) {
        const elapsed = Math.round(performance.now() - start);
        console.log(`← ${statusCode} ${method} ${path} (${elapsed}ms)`);
        handler.onResponseStart?.(controller, statusCode, headers, statusMessage);
      },
      onResponseData(controller, data) {
        handler.onResponseData?.(controller, data);
      },
      onResponseEnd(controller, trailers) {
        handler.onResponseEnd?.(controller, trailers);
      },
      // エラー時もログを出してから元の handler に委譲
      onResponseError(controller, err) {
        const elapsed = Math.round(performance.now() - start);
        console.log(`✗ ${method} ${path} (${elapsed}ms)`);
        handler.onResponseError?.(controller, err);
      },
    });
  };
};
```


### センシティブなパラメータのマスク

Backlog API では `apiKey` がクエリパラメータに含まれるため、ログにそのまま出力するとシークレットが漏れます。正規表現で `apiKey=xxx` を `apiKey=***` に置換します。

```ts:http-logger.ts
// apiKey=xxxxx → apiKey=*** に置換
const maskSensitiveParams = (url: string): string =>
  url.replaceAll(/([?&])(apiKey)=[^&]*/gi, "$1$2=***");
```

### パスのフォーマット

エンコードされたクエリパラメータ（例: `projectId%5B%5D=1`）はそのままだと読みづらいので、デコードして可読性を上げます。

```ts:http-logger.ts
// projectId%5B%5D=1 → projectId[]=1 のように読みやすくする
const formatPath = (path: string): string => {
  try {
    return decodeURIComponent(maskSensitiveParams(path));
  } catch {
    // 不正なエンコードの場合はマスクのみ適用
    return maskSensitiveParams(path);
  }
};
```

基本の実装で `opts.path` をそのまま使っていた部分を `formatPath(opts.path ?? "")` に差し替えれば完成です。

## グローバルへのインストール

```ts:http-logger.ts
const installHttpLogger = (): void => {
  // デフォルトの Agent にインターセプターを合成して、グローバルに設定
  const agent = new Agent().compose(createLoggingInterceptor());
  setGlobalDispatcher(agent);
};

export { createLoggingInterceptor, installHttpLogger };
```

エントリーポイントで `installHttpLogger()` を呼ぶだけで、以降のすべての `fetch` にロガーが適用されます。

```ts:index.ts
import { installHttpLogger } from "./http-logger";

installHttpLogger();
```

:::message alert
`setGlobalDispatcher` はプロセス全体の `fetch` に影響します。backlog-js 以外のライブラリや自前のリクエストもすべてロガーを経由するため、他の HTTP 通信が混在する場合はログにノイズが入る点に注意してください。
:::

## 出力例

```sh
API host: https://example.backlog.com
→ GET /api/v2/space/activities?apiKey=***
← 200 GET /api/v2/space/activities?apiKey=*** (279ms)
→ GET /api/v2/notifications?apiKey=***
← 200 GET /api/v2/notifications?apiKey=*** (712ms)
```

## 参考

https://github.com/nulab/bee/pull/79
