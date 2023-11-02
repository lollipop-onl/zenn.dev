## Top-level await

Top-level await は 2023/11/02 現在、 iOS 15 以上のすべてのモダンブラウザでサポートされています。

https://caniuse.com/mdn-javascript_operators_await_top_level

また、ECMAScript は ES2022 からサポートおり、 Node.js でも 14.8.0 （すでに EoL したバージョン）以上で利用できます。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/await#browser_compatibility

## Nuxt 3 で Top-level await を使う

Nuxt 3 で Top-level await を使ったコードをビルドしようとすると、次のようにエラーが発生してビルドが失敗します。

> ERROR: Top-level await is not available in the configured target environment ("chrome87", "edge88", "es2020", "firefox78", "safari14" + 2 overrides)

@[stackblitz](https://stackblitz.com/edit/nuxt-starter-qcc9gn?embed=1&file=app.vue)

これは、 Vite と Nitro がデフォルトで使用するビルドターゲットが以下の機能をすべてサポートしたバージョンで設定されているために発生します。

- ネイティブ ES モジュール
- ネイティブ ESM の動的インポート
- `import.meta`

たとえば、 Vite の `build.target` オプションの初期値は `'modules'` ですが、このときのビルドターゲットは `['es2020', 'edge88', 'firefox78', 'chrome87', 'safari14']` として扱われます。

https://ja.vitejs.dev/config/build-options.html#build-target

しかし、 Top-level await をサポートする各 UA のバージョンは `'modules'` のバージョンより上なため、 esbuild によるトランスパイル時にエラーとして失敗しています。

| UA | ES モジュール | 動的インポート | `import.meta` | Top-level await |
| :---: | :---: | :---: | :---: | :---: |
| ESModule | ES2020 | ES2020 | ES2020 | ES2022 |
| MS Edge | 16 | 79 | 79 | 89 |
| Firefox | 60 | 67 | 64 | 89 |
| Chrome | 61 | 63 | 64 | 89 |
| Safari | 10.1 | 11.1 | 11.1 | 15 |
| iOS Safari | 10.3 | 11.3 | 12.0 | 15.0 |

これは同様に Nitro のビルドでも発生します。

## ビルドターゲットを変更する

Top-level await に対応したビルドターゲットに変更しましょう。

Nuxt 3 は Vite （クライアント）と Nitro （サーバー）のビルドで esbuild を使用しているため、それぞれに設定が必要です。
変更する設定は [`vite.build.target`](https://nuxt.com/docs/api/nuxt-config#vite) と [`nitro.esbuild.options.target`](https://nuxt.com/docs/api/nuxt-config#nitro) です。

各 UA のサポートバージョンは以下のとおりなので、これらのバージョンで設定します。

| - | ESModule | MS Edge | Firefox | Chrome | Safari |
| Top-level await | ES2022 | 89 | 89 | 89 | 15 |

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  vite: {
    build: {
      target: ['es2022', 'edge89', 'firefox89', 'chrome89', 'safari15']
    },
  },
  nitro: {
    esbuild: {
      options: {
        // Node.js のバージョンのみ指定すればOK
        target: 'es2022',
      },
    },
  },
});
```

これで、 Top-level await を利用してもビルドエラーが発生しなくなりました。

@[stackblitz](https://stackblitz.com/edit/nuxt-starter-2dgsx4?embed=1&file=nuxt.config.ts)
