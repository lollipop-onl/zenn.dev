---
title: ビルド不要！ Google Apps Script のウェブアプリを Vue.js と Tailwind CSS でモダンに実装しよう
emoji: 🐈‍⬛
type: tech
topics: [googleappsscript, esm, vue, tailwindcss]
published: false
---

この記事は「ヌーラボブログリレー 2025 夏」の 1 日目です。

![](https://storage.googleapis.com/zenn-user-upload/e27f3fa5c329-20240729.png)

明日以降の執筆予定については [夏だ！祭りだ！ブログリレーがやってくる！#ヌーラボ真夏のブログリレー 2024](#) をご覧ください！

# はじめに

Google Apps Script (GAS) は Google の各サービスと連携したアプリケーションを開発できるプラットフォームです。

GAS には `doGet` メソッドを定義することで、 UI を持つ Web アプリケーションを公開する機能があります。
この機能を使えば、サーバーサイドの GAS の関数を JavaScript から呼び出したり、動的な値を埋め込んだ HTML を生成したりすることが可能です。

https://developers.google.com/apps-script/guides/web?hl=ja

しかし、 GAS には一つ大きな制約があります。
それは、 **JavaScript や CSS といったクライアントサイドのアセットファイルを単体でホスティングできない点** です。
このため、 GAS では素朴な JavaScript や CSS を書くことが多いかなと思います。

そこでこの記事では、 **近年のブラウザに標準搭載された ES Modules などの機能を活用し、 GAS 上でモダンなフロントエンド技術スタックを利用する方法** を紹介します。

# 利用する技術・ツール

## ES modules

ES modules (ESM) は JavaScript 公式のモジュールシステムです。
2018 年ごろには主要なブラウザでサポートされました。

ESM では `<script type="module">` を使って定義されたスクリプトでは、 URL から外部の JavaScript ファイルを直接モジュールとして読み込めます。
これにより、 GAS の HTML テンプレートのようなインラインスクリプトでも通常の Vue.js のような使い勝手でスクリプトやコンポーネントを記述できます。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Modules

## Vue.js (CDN)

UI ライブラリとしては Vue.js を利用します。今回は、次のような理由で選定しました。

- Vue 3 から ES modules 向けにビルドされたスクリプトが提供されている
- テンプレートを HTML として記述できることで、シンタックスハイライトの恩恵を受けられる

https://ja.vuejs.org/guide/quick-start#using-vue-from-cdn

## Tailwind CSS (Play CDN)

CSS は Taiwind CSS が提供している Play CDN 版を利用します。

Play CDN 版では、実行時に JavaScript で必要なクラスを動的に読み込めたり、テーマや設定をカスタマイズできたりするため、通常の Tailwind CSS と同等の使い勝手で利用ができます。

https://tailwindcss.com/docs/installation/play-cdn

# 今回作成するもの

この記事では次のようなウェブアプリを開発します。

- ボタンを押すと、サーバーサイドから現在時刻の ISO 文字列を取得する
- クライアントサイドで日時をユーザーの言語に変換して表示する

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
