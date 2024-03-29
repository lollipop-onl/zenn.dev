---
title: TailwindCSS v4.0 を試そう！
emoji: 🌬️
type: tech
topics: [tailwindcss, vite]
published: true
---

2024年3月7日に TailwindCSS v4.0 のアルファ版が公開されました。

変更点の詳細はオフィシャルのブログ記事をご覧ください。

https://tailwindcss.com/blog/tailwindcss-v4-alpha

そんなアルファ版ですが、執筆時点では [tailwindcss@4.0.0-alpha.11](https://www.npmjs.com/package/tailwindcss/v/4.0.0-alpha.11) が利用可能です。

ただし、 v4 を Vite などのビルドツールで利用する方法などについて言及がされていません。
ここでは、各ツールで TailwindCSS 4.0 のアルファ版を使う方法をまとめます。

## for Vite

Vite 向けには公式で `@tailwindcss/vite` プラグインが用意されているようです。

https://github.com/tailwindlabs/tailwindcss/tree/a79fa45bf2ac57854d41a3d0c6f226090944c0c5/packages/%40tailwindcss-vite

プラグインの依存関係に同バージョンの `tailwindcss` が含まれているので、個別でのインストールは不要です。

```shell
npm install -D @tailwindcss/vite@next
```

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [tailwindcss()],
});
```

以上でセットアップは完了です。

v3 での `contents` にあたるコンテンツの検出が自動化されているため、 `tailwind.config.js` も不要になりました。

## for PostCSS

PostCSS 向けには公式で `@tailwindcss/postcss` プラグインが用意されています。

https://github.com/tailwindlabs/tailwindcss/tree/next/packages/%40tailwindcss-postcss

プラグインの依存関係に同バージョンの `tailwindcss` が含まれているので、個別でのインストールは不要です。

```shell
npm install -D @tailwindcss/postcss@next
```

```js
// postcss.config.js
module.exports = {
  plugins: {
    '@tailwindcss/postcss': {},
    autoprefixer: {},
  },
};
```

Vite と同じく、 `tailwind.config.js` の作成は不要です。

## for CLI

CLI ツールは `@tailwindcss/cli` として用意されています。

https://github.com/tailwindlabs/tailwindcss/tree/next/packages/%40tailwindcss-cli

```shell
npm install -D @tailwindcss/cli@next
```

```shell
$ npx tailwindcss -h
≈ tailwindcss v4.0.0-alpha.11

Usage:
  tailwindcss [--input input.css] [--output output.css] [--watch] [options…]

Options:
  -i, --input ··········· Input file
  -o, --output ·········· Output file
  -w, --watch ··········· Watch for changes and rebuild as needed
  -m, --minify ·········· Optimize and minify the output
      --optimize ········ Optimize the output without minifying
      --cwd ············· The current working directory [default: `.`]
  -h, --help ············ Display usage information
```