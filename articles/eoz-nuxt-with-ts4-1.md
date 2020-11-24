---
title: 暫定的に Nuxt で TypeScript 4.1 を使用する
emoji: 🍭
type: tech
topics: [nuxt, typescript, EveOneZenn]
published: true
---

# はじめに

この記事は #EveOneZenn (Everyday One Zenn) の２日目です。

まだ公式でサポートされていない Nuxt で TypeScript 4.1 を使用するまでのセットアップをまとめます。
`@nuxt/typescript-build` が公式に TypeScript 4.1 をサポートするまでの間は、ここで詳細する手順でプロジェクトをセットアップする必要があります。

# TL;DR

`typescript` と `ts-loader` を手動インストールする。

```shell
$ yarn add -D typescript ts-loader
```

# 単純に `@nuxt/typescript-build` をセットアップした場合

まずは、単純に `@nuxt/typescript-build` をインストールしたプロジェクトで TypeScript 4.1 の新機能、 Template Literal Types を使用してみます。

依存関係と `nuxt.config.ts` は次のとおりです。
ここでは、Vueコンポーネントをタイプセーフにするため `@nuxt/composition-api` も使用しています。

```json:package.json
{
  "scripts": {
    "dev": "nuxt-ts"
  },
  "dependencies": {
    "@nuxt/typescript-runtime": "^2.0.0",
    "nuxt": "^2.14.7"
  },
  "devDependencies": {
    "@nuxt/types": "^2.14.7",
    "@nuxt/typescript-build": "^2.0.3",
    "@nuxtjs/composition-api": "^0.15.1"
  }
}
```

```ts:nuxt.config.ts
import { NuxtConfig } from '@nuxt/types';

const config: NuxtConfig = {
  buildModules: [
    '@nuxt/typescript-build',
    '@nuxtjs/composition-api'
  ]
};

export default config;
```

このプロジェクトで次のように Template Literal Types を使用するページを作成してみます。

```vue:pages/index.vue
<script lang="ts">
import { defineComponent, computed } from '@nuxtjs/composition-api';

const NAME = 'simochee';

export default defineComponent({
  setup() {
    const greet = <T extends string>(name: T): `Hello, ${T}` => `Hello, ${name}` as const;
    const greeting = computed((): 'Hello, Zenn' => greet(NAME));

    return {
      greeting,
    };
  },
});
</script>
```

この時点で、エディタ（VSCode）上でシンタックスエラーとなります。

![](https://storage.googleapis.com/zenn-user-upload/gf34zbo1nm9lwlsd2bcu8ohtbvf3)

Nuxt ビルダー上でも同様に複数のエラーが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/3c4e9j8t0i1w17rds74exybiul8k)

# TypeScript のバージョンアップ

続いて、 TypeScript を `@nuxt/typescript-build` に依存しているパッケージとは別に手動で `v4.1` をインストールします。

```shell
$ yarn add -D typescript
```

![](https://storage.googleapis.com/zenn-user-upload/abmjukabl99aiuedh0bhxkuvrenv)

これで、エディタ上に用事されていたエラーは Template Literal Types の型不正のみとなります。

![](https://storage.googleapis.com/zenn-user-upload/gvmrv4a2grezurm0clivj7yg00t3)

:::message
**シンタックスエラーが消えないときは**
シンタックスエラーが消えないときは、 VSCode が TypeScript のバージョン更新を検知できていない可能性があります。
バージョン更新を反映するにはコマンドパレットから `開発者：ウィンドウの再読み込み` を実行します。
これで、エディタのウィンドウが再読み込みされ、エディタ上の依存関係が最新化されます。
:::

しかし、Nuxt ビルダー上でも型不正エラーが表示されますが、コンパイルエラーも表示されます。

![](https://storage.googleapis.com/zenn-user-upload/pbi3d45yshcmeuvbm6tz1w4j6mbs)

# `ts-loader` のインストール

ビルダーに表示されたコンパイルエラーは `ts-loader` を手動でインストールしなおせば解消されます。

```shell
$ yarn add -D ts-loader
```

![](https://storage.googleapis.com/zenn-user-upload/s3l6qg8prp3fycp6315yrx3cp6hz)

これで、 Nuxt ビルダー上でも型不正エラーのみが表示されるようになりました。

![](https://storage.googleapis.com/zenn-user-upload/80nko9ae38lg2qb2z1vzkuidz3ad)
