---
title: ビルド不要！ Google Apps Script のウェブアプリを Vue.js と Tailwind CSS で実装しよう
emoji: 🐈‍⬛
type: tech
topics: [googleappsscript, esm, vue, tailwindcss, ヌーラボブログリレー2025夏]
published: true
---

この記事は [ヌーラボブログリレー2025夏](https://nulab.com/ja/blog/nulab/nulaber-blog-relay-2025-summer/) の1日目として投稿しています。

![ビルド不要！ Google Apps Script のウェブアプリを Vue.js と Tailwind CSS でモダンに実装しよう #ヌーラボブログリレー2025夏](https://storage.googleapis.com/zenn-user-upload/dd36d3b993e0-20250806.png)

明日以降の執筆予定については [夏の恒例！「ヌーラボブログリレー2025 夏」を開催します！ #ヌーラボブログリレー2025夏](https://nulab.com/ja/blog/nulab/nulaber-blog-relay-2025-summer/) をご覧ください！

# はじめに

Google Apps Script (GAS) は Google の各サービスと連携したアプリケーションを開発できるプラットフォームです。

GAS には `doGet` メソッドを定義することで、 UI を持つ Web アプリケーションを公開する機能があります。
この機能を使えば、サーバーサイドの GAS の関数を JavaScript から呼び出したり、動的な値を埋め込んだ HTML を生成したりすることが可能です。

https://developers.google.com/apps-script/guides/web?hl=ja

しかし、 GAS には一つ大きな制約があります。
それは、 **JavaScript や CSS といったクライアントサイドのアセットファイルを単体でホスティングできない点** です。
このため、 GAS では素朴な JavaScript や CSS を書くことが多いかなと思います。

そこでこの記事では、 **近年のブラウザに標準搭載された ES Modules などの機能を活用し、 GAS 上でモダンなフロントエンド技術スタックを利用する方法** を紹介します。

# 今回作成するファイルの全文

:::details script.gs

```js
function doGet() {
  const template = HtmlService.createTemplateFromFile("index");

  return template.evaluate();
}

function getCurrentTime() {
  return {
    currentTime: new Date().toISOString(),
  };
}
```

:::

:::details index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <base target="_top" />
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script type="importmap">
      {
        "imports": {
          "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
        }
      }
    </script>
  </head>
  <body>
    <script type="module">
      import { createApp, ref, computed, onMounted } from "vue";

      const app = createApp();

      const useGoogleScript = (name) => {
        const isPending = ref(false);
        const error = ref(null);

        const callGoogleScript = (...args) =>
          new Promise((resolve, reject) => {
            google.script.run
              .withSuccessHandler(resolve)
              .withFailureHandler(reject)
              [name](...args);
          });

        const run = async (...args) => {
          try {
            isPending.value = true;
            error.value = null;

            return await callGoogleScript(...args);
          } catch (err) {
            error.value = err;
          } finally {
            isPending.value = false;
          }
        };

        return { run, isPending, error };
      };

      app.component("ServerCurrentTime", {
        template: "#server-current-time",
        setup() {
          const isoString = ref(null);
          const isInitializing = ref(true);

          const formattedDate = computed(() =>
            Intl.DateTimeFormat(navigator.language, {
              dateStyle: "long",
              timeStyle: "long",
            }).format(new Date(isoString.value))
          );

          const { run, isPending } = useGoogleScript("getCurrentTime");

          const getCurrentTime = async () => {
            const { currentTime } = await run();

            console.log(currentTime);

            isoString.value = currentTime;
          };

          onMounted(() => {
            getCurrentTime().finally(() => {
              isInitializing.value = false;
            });
          });

          return {
            isoString,
            formattedDate,
            isInitializing,
            isPending,
            getCurrentTime,
          };
        },
      });

      app.mount("#root");
    </script>

    <template id="server-current-time">
      <div class="grid gap-5 border rounded px-6 py-4">
        <template v-if="isInitializing">
          <p class="text-gray-400">取得中...</p>
        </template>
        <template v-else>
          <dl class="grid gap-3">
            <dt class="font-bold">現在の日時（サーバー）</dt>
            <dd>
              <time :datetime="isoString">{{ formattedDate }}</time>
            </dd>
          </dl>
          <div class="flex justify-center">
            <button
              :class="['px-4 py-1 rounded border border-blue-300 bg-blue-200 text-blue-600 hover:opacity-60 disabled:opacity-60', { 'animate-pulse': isPending }]"
              :disabled="isPending"
              @click="getCurrentTime"
            >
              日時を更新
            </button>
          </div>
        </template>
      </div>
    </template>

    <div id="root">
      <div class="py-10 mx-auto max-w-md">
        <server-current-time></server-current-time>
      </div>
    </div>
  </body>
</html>
```

:::

# 利用する技術・ツール

## [ES modules](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules)

ES modules (ESM) は JavaScript 公式のモジュールシステムです。
2018 年ごろには主要なブラウザでサポートされました。

ESM では `<script type="module">` を使って定義されたスクリプトでは、 URL から外部の JavaScript ファイルを直接モジュールとして読み込めます。
これにより、 GAS の HTML テンプレートのようなインラインスクリプトでも通常の Vue.js のような使い勝手でスクリプトやコンポーネントを記述できます。

## [importmap](https://developer.mozilla.org/ja/docs/Web/HTML/Reference/Elements/script/type/importmap)

`importmap` を使うと、 ESM のモジュール名を実際の URL にマッピングできます。

```html
<script type="importmap">
  {
    "imports": {
      "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
    }
  }
</script>
```

上記のような設定を行うと、 ESM スクリプト内で `import { createApp } from 'vue'` のようにパッケージ名でモジュールを読み込めるようになります。

## [Vue.js](https://ja.vuejs.org) (CDN)

UI ライブラリとしては Vue.js を利用します。今回は、次のような理由で選定しました。

- Vue 3 から ES modules 向けにビルドされたスクリプトが提供されている
- テンプレートを HTML として記述できることで、シンタックスハイライトの恩恵を受けられる

## [Tailwind CSS](https://tailwindcss.com) (Play CDN)

CSS は Tailwind CSS が提供している Play CDN 版を利用します。

Play CDN 版では、実行時に JavaScript で必要なクラスを動的に読み込めたり、テーマや設定をカスタマイズできたりするため、通常の Tailwind CSS と同等の使い勝手で利用ができます。

# 今回作成するもの

この記事では次のようなウェブアプリを開発します。

- ボタンを押すと、サーバーサイドから現在時刻の ISO 文字列を取得する
- クライアントサイドで日時をローカライズして表示する

# スクリプトプロジェクトのセットアップ

:::message
すでにウェブアプリとしてセットアップができている場合は、この章をスキップできます。
:::

スクリプトプロジェクトは以下のガイドを参考に作成されているものとします。

https://developers.google.com/apps-script/guides/projects?hl=ja

## スクリプトファイル

まずは、スクリプトファイルにはウェブアプリを表示するための `doGet` メソッドと、クライアントサイドから利用する `getCurrentTime` メソッドを作成します。

```js
/* filename: script.gs */

function doGet() {
  const template = HtmlService.createTemplateFromFile("index");

  return template.evaluate();
}

function getCurrentTime() {
  return {
    currentTime: new Date().toISOString(),
  };
}
```

他の Google サービスと連携させる処理や、クライアントサイドでは実行できない処理などはスクリプトファイルにメソッドで定義してください。

## テンプレートファイル

続いて、テンプレートファイルを作成します。

ウェブアプリとしてページが表示できる最低限の内容となります。
おそらく、 HTML ファイルを追加した際の初期の内容のままで構いません。

```html
<!-- index.html -->

<!DOCTYPE html>
<html>
  <head>
    <base target="_top" />
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  </head>
  <body></body>
</html>
```

## スクリプトをデプロイする

最後に、スクリプトをウェブアプリとしてデプロイします。
以下のガイドを参考にウェブアプリのデプロイを行ってください。

https://developers.google.com/apps-script/guides/web?hl=ja#deploy_a_script_as_a_web_app

一度でもデプロイを行うと「デプロイをテスト」から最新の内容でウェブアプリの動作確認ができます。

# フロントエンドを実装する

クライアントサイドの実装を行います。
`index.html` を編集していきます。

## Tailwind CSS のセットアップ

https://tailwindcss.com/docs/installation/play-cdn

Tailwind CSS は `<head>` タグ内に Play CDN のスクリプトを追加することで利用可能になります。

```html
<head>
  <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
</head>
```

:::details ここまでの index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <base target="_top" />
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
  </head>
  <body></body>
</html>
```

:::

## Vue.js のセットアップ

https://ja.vuejs.org/guide/quick-start#using-vue-from-cdn

Vue.js は `importmap` で `vue` のモジュールを定義してインポートするようにします。

まず、 `importmap` で `vue` のモジュールを定義します。

```html
<head>
  <script type="importmap">
    {
      "imports": {
        "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
      }
    }
  </script>
</head>
```

続いて、 Vue.js をマウントする要素を定義します。

```html
<div id="root"></div>
```

最後に、 Vue.js のアプリケーションをマウントさせます。

```html
<script type="module">
  import { createApp } from "vue";

  const app = createApp();

  app.mount("#root");
</script>
```

これで、 Vue.js のアプリケーションを ESM でマウントできました。

:::details ここまでの index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <base target="_top" />
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script type="importmap">
      {
        "imports": {
          "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
        }
      }
    </script>
  </head>
  <body>
    <script type="module">
      import { createApp } from "vue";

      const app = createApp();

      app.mount("#root");
    </script>

    <div id="root"></div>
  </body>
</html>
```

:::

## Vue.js コンポーネントの実装

https://ja.vuejs.org/guide/components/registration

現在時刻を状態に持つ Vue.js のコンポーネントを実装します。

### テンプレート

まず、テンプレート部分を HTML として定義します。
コンテンツテンプレート要素でコンポーネントのテンプレートを記述します。

```html
<template id="server-current-time">
  <div class="grid gap-5 border rounded px-6 py-4">
    <dl class="grid gap-3">
      <dt class="font-bold">現在の日時（サーバー）</dt>
      <dd>
        <time :datetime="isoString">{{ formattedDate }}</time>
      </dd>
    </dl>
    <div class="flex justify-center">
      <button
        class="px-4 py-1 rounded border border-blue-300 bg-blue-200 text-blue-600 hover:opacity-60"
      >
        日時を更新
      </button>
    </div>
  </div>
</template>
```

### コンポーネントの登録

次に、定義したテンプレートを利用したコンポーネントをグローバルに登録します。
ESM では `<script>` タグを跨いで定義された変数を利用できないため、同じ `<script>` タグでコンポーネントを登録する必要があります。

```html
<script type="module">
  import { createApp, ref, computed } from "vue";

  const app = createApp();

  app.component("ServerCurrentTime", {
    template: "#server-current-time",
    setup() {
      const isoString = ref(new Date().toISOString());

      const formattedDate = computed(() =>
        new Date(isoString.value).toLocaleString()
      );

      return { isoString, formattedDate };
    },
  });

  app.mount("#root");
</script>
```

### コンポーネントの使用

:::message

HTML の仕様により、コンポーネント名や Props 、イベントリスナーは小文字のケバブケースで記述する必要があります。
また、 `<server-current-time />` のような自己クロージングタグも利用できません。

> [DOM 内テンプレート解析の注意点](https://ja.vuejs.org/guide/essentials/component-basics.html#in-dom-template-parsing-caveats)

:::

前項で登録した `ServerCurrentTime` コンポーネントを HTML に描画します。

```html
<div id="root">
  <server-current-time></server-current-time>
</div>
```

これで、クライアントサイドで取得した日時に基づいて、ローカライズされたテキストとして表示できるようになりました。

:::details ここまでの index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <base target="_top" />
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script type="importmap">
      {
        "imports": {
          "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
        }
      }
    </script>
  </head>
  <body>
    <script type="module">
      import { createApp, ref, computed } from "vue";

      const app = createApp();

      app.component("ServerCurrentTime", {
        template: "#server-current-time",
        setup() {
          const isoString = ref(new Date().toISOString());

          const formattedDate = computed(() =>
            new Date(isoString.value).toLocaleString()
          );

          return { isoString, formattedDate };
        },
      });

      app.mount("#root");
    </script>

    <template id="server-current-time">
      <div class="grid gap-5 border rounded px-6 py-4">
        <dl class="grid gap-3">
          <dt class="font-bold">現在の日時（サーバー）</dt>
          <dd>
            <time :datetime="isoString">{{ formattedDate }}</time>
          </dd>
        </dl>
        <div class="flex justify-center">
          <button
            class="px-4 py-1 rounded border border-blue-300 bg-blue-200 text-blue-600 hover:opacity-60"
          >
            日時を更新
          </button>
        </div>
      </div>
    </template>

    <div id="root">
      <div class="py-10 mx-auto max-w-md">
        <server-current-time></server-current-time>
      </div>
    </div>
  </body>
</html>
```

:::

## GAS スクリプトの呼び出し

ここまでで実装した Vue.js アプリケーションから GAS で定義したメソッドを呼び出せるようにします。

### コンポーザブル

GAS のスクリプトは、テンプレートから `google.script.run` クラスで呼び出せます。

https://developers.google.com/apps-script/guides/html/reference/run?hl=ja

このクラスを利用した呼び出しと関連する状態をまとめたコンポーザブルを実装します。

```js
const useGoogleScript = (name) => {
  const isPending = ref(false);
  const error = ref(null);

  const callGoogleScript = (...args) =>
    new Promise((resolve, reject) => {
      google.script.run
        .withSuccessHandler(resolve)
        .withFailureHandler(reject)
        [name](...args);
    });

  const run = async (...args) => {
    try {
      isPending.value = true;
      error.value = null;

      return await callGoogleScript(...args);
    } catch (err) {
      error.value = err;
    } finally {
      isPending.value = false;
    }
  };

  return { run, isPending, error };
};
```

### 呼び出し

前項で実装したコンポーザブルを `ServerCurrentTime` コンポーネントで呼び出します。
併せて、呼び出し中にボタンを押せないようにしたり、初期取得中にメッセージを表示したりする修正を加えます。

```html
<script type="module">
  import { ..., ref, onMounted } from 'vue';

  ...

  app.component('ServerCurrentTime', {
    template: '#server-current-time',
    setup() {
      const isoString = ref(null);
      const isInitializing = ref(true);

      const formattedDate = computed(() =>
        Intl.DateTimeFormat(
          navigator.language,
          { dateStyle: 'long', timeStyle: 'long' },
        ).format(new Date(isoString.value))
      );

      const { run, isPending } = useGoogleScript('getCurrentTime');

      const getCurrentTime = async () => {
        const { currentTime } = await run();

        isoString.value = currentTime;
      }

      onMounted(() => {
        getCurrentTime()
          .finally(() => {
            isInitializing.value = false;
          })
      });

      return {
        isoString,
        formattedDate,
        isInitializing,
        isPending,
        getCurrentTime
      };
    },
  });
</script>

<template id="server-current-time">
  <div class="grid gap-5 border rounded px-6 py-4">
    <template v-if="isInitializing">
      <p class="text-gray-400">取得中...</p>
    </template>
    <template v-else>
      <dl class="grid gap-3">
        <dt class="font-bold">現在の日時（サーバー）</dt>
        <dd>
          <time :datetime="isoString">{{ formattedDate }}</time>
        </dd>
      </dl>
      <div class="flex justify-center">
        <button
          :class="['px-4 py-1 rounded border border-blue-300 bg-blue-200 text-blue-600 hover:opacity-60 disabled:opacity-60', { 'animate-pulse': !isPending }]"
          :disabled="isPending"
          @click="getCurrentTime"
        >
          日時を更新
        </button>
      </div>
    </template>
  </div>
</template>
```

これで、 GAS のスクリプトファイルで定義した関数から取得した日時に基づいて、ローカライズされた日時を表示できるようになりました。

:::details ここまでの index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <base target="_top" />
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script type="importmap">
      {
        "imports": {
          "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
        }
      }
    </script>
  </head>
  <body>
    <script type="module">
      import { createApp, ref, computed, onMounted } from "vue";

      const app = createApp();

      const useGoogleScript = (name) => {
        const isPending = ref(false);
        const error = ref(null);

        const callGoogleScript = (...args) =>
          new Promise((resolve, reject) => {
            google.script.run
              .withSuccessHandler(resolve)
              .withFailureHandler(reject)
              [name](...args);
          });

        const run = async (...args) => {
          try {
            isPending.value = true;
            error.value = null;

            return await callGoogleScript(...args);
          } catch (err) {
            error.value = err;
          } finally {
            isPending.value = false;
          }
        };

        return { run, isPending, error };
      };

      app.component("ServerCurrentTime", {
        template: "#server-current-time",
        setup() {
          const isoString = ref(null);
          const isInitializing = ref(true);

          const formattedDate = computed(() =>
            Intl.DateTimeFormat(navigator.language, {
              dateStyle: "long",
              timeStyle: "long",
            }).format(new Date(isoString.value))
          );

          const { run, isPending } = useGoogleScript("getCurrentTime");

          const getCurrentTime = async () => {
            const { currentTime } = await run();

            console.log(currentTime);

            isoString.value = currentTime;
          };

          onMounted(() => {
            getCurrentTime().finally(() => {
              isInitializing.value = false;
            });
          });

          return {
            isoString,
            formattedDate,
            isInitializing,
            isPending,
            getCurrentTime,
          };
        },
      });

      app.mount("#root");
    </script>

    <template id="server-current-time">
      <div class="grid gap-5 border rounded px-6 py-4">
        <template v-if="isInitializing">
          <p class="text-gray-400">取得中...</p>
        </template>
        <template v-else>
          <dl class="grid gap-3">
            <dt class="font-bold">現在の日時（サーバー）</dt>
            <dd>
              <time :datetime="isoString">{{ formattedDate }}</time>
            </dd>
          </dl>
          <div class="flex justify-center">
            <button
              :class="['px-4 py-1 rounded border border-blue-300 bg-blue-200 text-blue-600 hover:opacity-60 disabled:opacity-60', { 'animate-pulse': isPending }]"
              :disabled="isPending"
              @click="getCurrentTime"
            >
              日時を更新
            </button>
          </div>
        </template>
      </div>
    </template>

    <div id="root">
      <div class="py-10 mx-auto max-w-md">
        <server-current-time></server-current-time>
      </div>
    </div>
  </body>
</html>
```

:::

# おまけ： Tips 集

## 属性にダブルクォーテーションを使わない

GAS のテンプレートでは Vue の SFC と異なり、あくまで HTML として描画されるため属性の値にダブルクォーテーションを直接使うことはできません。
（形式が正しくない HTML コンテンツ としてエラーになります）

`&quot;` のように特殊文字を使用する方法もありますが、属性の値ではシングルクォーテーションを利用するようにするのが手軽です。

```html
<my-component
  :class="['border', { 'border-red-400': flag }]"
  @complete="showToast('処理が完了しました')"
>
</my-component>
```

## VueUse を利用する

https://vueuse.org/

Vue.js 向けのコンポジションユーティリティライブラリである VueUse も CDN 経由で ESM として利用が可能です。

```html
<script type="importmap">
  {
    "imports": {
      "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js",
      "@vueuse/shared": "https://unpkg.com/@vueuse/shared@13/index.mjs",
      "@vueuse/core": "https://unpkg.com/@vueuse/core@13/index.mjs"
    }
  }
</script>

<script type="module">
  import { usePointer } from '@vueuse/core';

  ...
</script>
```

VueUse の場合は参照しているモジュールが少ないため上記のコードのみで動きます。
他のライブラリを利用する場合は、そのモジュールが参照している他のモジュールについての `importmap` を定義する必要があります。

ライブラリによっては依存関係が複雑になることもあるため、 `importmap` の作成には JSPM Generator が便利です。

https://generator.jspm.io/
