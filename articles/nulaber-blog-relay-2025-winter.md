---
title: 無料 (Gemini 2.5 Flash) で LLM を使った屋台検索を実装してみる
emoji: 🐈‍⬛
type: tech
topics: [ヌーラボブログリレー2025冬, gemini]
---

この記事は [ヌーラボブログリレー2025 冬](https://adventar.org/calendars/11965) の3日目として投稿しています。

[![無料 (Gemini 2.5 Flash) で LLM を使った屋台検索を実装してみる](https://storage.googleapis.com/zenn-user-upload/db9f678b11cd-20251203.png)](https://adventar.org/calendars/11965)

他の日に投稿される記事については [【ヌーラボブログリレー2025 冬】for Tech Advent Calendar 2025](https://adventar.org/calendars/11965) をご覧ください！

# はじめに

さまざまなモデルが群雄割拠な昨今ですが、進化の過程で上限はありますが無料で使えるモデルも増えてきました。

この記事では Gemini 2.5 Flash を Google AI Studio で発行した API キーを利用して呼び出し、特定のデータに基づいた回答の生成をやってみます。

なお、内容はプロンプトやデータの細かいチューニングというよりは Node.js から Gemini を呼び出すためのハウツーのようなものになっています。

## 利用するモデル

今回利用するモデルは Gemini API で利用できるモデルのうち、無料枠が大きい Gemini 2.5 Flash です。

Gemini 2.5 は 10 リクエスト/分（ 250 リクエスト/日）、 25万トークン/分 まで無料で利用できるため、検証や学習程度であれば十分です。

なお、レート上限に達した場合は Gemini API から 429 レスポンスが返却されます。
その際は数分待ってから実行し直しましょう。

https://ai.google.dev/gemini-api/docs/rate-limits?hl=ja

# 事前準備

## Gemini API キーの取得

![](https://storage.googleapis.com/zenn-user-upload/0e3f3ae1cfe4-20251203.png)

ドキュメントに記載の手順で Google AI Studio で API キーを発行してください。

https://ai.google.dev/gemini-api/docs/api-key?hl=ja

## オープンデータの取得

この記事では福岡市が提供するオープンデータ「屋台基本情報」を利用します。

下記から JSON 形式でファイルをダウンロードしてください。

https://data.bodik.jp/dataset/401307_yataiopendata/resource/328edbc1-6967-4d0a-8f6d-6678420f4fe2

## 開発環境

Node.js 22 以上がインストールされている環境を前提とします。

`npm init` で `package.json` を作成後、空の `index.mjs` ファイルを作成した状態です。
前項でダウンロードした屋台のデータは `dataset.json` としてプロジェクトに移動させています。

```
.
|- index.mjs
|- dataset.json
`- package.json
```

## Gemini を呼び出すクライアントのインストール

今回は Google によって提供されている `@google/genai` を利用して Gemini API を呼び出します。

以下のコマンドでパッケージをインストールしてください。

```sh
npm add @google/genai
```

# Gemini を Node.js から呼び出してみる

まずは `@google/genai` を使って Gemini API を呼び出してみます。

`@google/genai` の README を少し変更したプロンプトを指定します。

```js
import { GoogleGenAI } from '@google/genai';
const GEMINI_API_KEY = process.env.GEMINI_API_KEY;

const ai = new GoogleGenAI({ apiKey: GEMINI_API_KEY });

async function main() {
  const response = await ai.models.generateContent({
    model: 'gemini-2.5-flash',
    contents: 'ヌーラボの代表的なキャラクターは？',
  });
  console.log(response.text);
}

await main();
```

これを次のコマンドで実行します。

```sh
GEMINI_API_KEY=**** node index.mjs
```

数秒待つと次のような出力が得られました。

```
ヌーラボの代表的なキャラクターといえば、プロジェクト管理ツール **Backlog（バックログ）の「カエルさん」** です。

鮮やかな緑色のカエルで、Backlogのウェブサイトやロゴ、Tシャツなどのグッズにも登場し、ヌーラボ全体のマスコットキャラクターのような存在感があります。

*   **サービス名:** Backlog（バックログ）
*   **キャラクター名:** カエルさん
*   **特徴:** 緑色で親しみやすいデザイン。製品のアイコンやブランディングに広く使われています。

他のヌーラボ製品には、Backlogのカエルさんほど明確で前面に出ているキャラクターはいませんが、カエルさんはヌーラボの顔として広く認識されています。
```

ヌーラボの代表的なキャラクターは [ヌーマン](https://x.com/nulabjp/status/1097360567243812864) やサルの[ダイミョー](https://backlog.com/ja/blog/interview-backlog-saru-waka-daimyo/)、ゴリラのゴリットくんなどなので誤った出力となりました。

# データセットを指定する

ここからが本題です。
福岡市の屋台をユーザーにおすすめする実装を行っていきます。

## データセットなしで出力させてみる

まずは、データセットなしで Gemini を呼び出してみます。

ここでは地鶏の炭火焼きが食べられるお店を聞いてみます。
オープンデータによると「KUROチャン」という屋台が該当するようです。

```js
async function main() {
  const response = await ai.models.generateContent({
    model: 'gemini-2.5-flash',
    contents: '福岡市の屋台で地鶏の炭火焼きが食べられるお店は？',
  });
  console.log(response.text);
}
```

実行した結果が次の通りです：

```
福岡市の屋台で「地鶏の炭火焼き」を専門に提供しているお店は、非常に珍しい、またはほとんどないと考えるのが一般的です。

(長いので省略)

結論として、屋台の魅力は雰囲気と手軽さにありますが、地鶏の炭火焼きという特定のメニューにこだわるのであれば、専門店を訪れることをお勧めします。
```

屋台ではなく地鶏の専門店をおすすめされてしまいました。

## データセットをそのまま含めて出力させてみる

データセットがない状態では特にニッチな内容でリクエストすると、期待したものではないレスポンスが得られました。

続いて、屋台基本情報の JSON データをそのまま埋め込んで試してみましょう。

```js
import dataset from './dataset.json' with { type: 'json' };

async function main() {
  const response = await ai.models.generateContent({
    model: 'gemini-2.5-flash',
    contents: `
福岡市の屋台で地鶏の炭火焼きが食べられるお店は？

以下のデータセットから回答して
${JSON.stringify(dataset)}
    `.trim(),
  });
  console.log(response.text);
}
```

実行した結果が次の通りです：

```
福岡市の屋台で地鶏の炭火焼きが食べられるお店は、「**KUROチャン**」です。

**KUROチャンの情報：**
*   **名称:** KUROチャン
*   **場所:** 福岡市博多区中洲中島町1（地下鉄空港線 中洲川端駅 2番出口 徒歩3分）
*   **営業時間:** 18:00～1:00
*   **定休日:** 日曜日
*   **店主のオススメ！:** 地鶏の炭火焼き 1,300円

故郷、宮崎のソウルフード「地鶏の炭火焼き」は、糸島産赤鶏のモモ肉を豪快に炭火で焼き上げ、ピリ辛の柚子胡椒を付けて熱々の鉄板で提供されます。
```

未編集の JSON データでしたが、期待したとおり「KUROチャン」がレスポンスされました。

# データを整理してトークン数を節約する

利用モデルの章で説明したとおり、 Gemini 2.5 Flash は分間のトークン数でレート上限が設定されています。
そうでなくても、一般的にトークンが増えるほど、応答精度が下がりやすくなり、発生する料金も上がってしまいます。

ここまでで作成したコードではデータセットをそのまま埋め込んでいましたが、 Gemini が必要とするデータのみに絞ってリクエストするよう調整していきます。

なお、トークン数は文字数ともワード数とも異なる粒度でカウントされます。
https://ai.google.dev/gemini-api/docs/tokens?hl=ja&lang=javascript

また、 Gemini とカウント方法が若干異なる可能性はありますが、 OpenAI の Tokenizer を利用することでトークン数を概算できます。
https://platform.openai.com/tokenizer

## やりとりで発生したトークン数を確認する

`ai.models.generateContent()` のレスポンスには `usageMetadata` というプロパティが含まれています。
この内容を確認することで、 Gemini API とのやりとりで発生したトークン数などが確認できます。

試しに、現在の実装で `usageMetadata` を出力させてみましょう。

```diff
const response = await ai.models.generateContent({ ... });

console.log(response.text);
+ console.log(response.usageMetadata);
```

実行した結果が次の通りです。

```jsonc
{
  // 入力として Gemini に送信されたトークンの総量
  promptTokenCount: 133333,
  // Gemini によって生成されたトークンの総量
  candidatesTokenCount: 141,
  // Gemini がレスポンスまでに生成された思考トークンの総量
  thoughtsTokenCount: 5715,
  // 各トークン数の合計
  totalTokenCount: 139189
}
```

調整していない状態での入力トークン数はおよそ13万でした。

この値がなるべく小さくなるように入力するデータを調整していきます。

## データのフォーマットを変更する

:::message
今回はデータにカンマが含まれないため、簡単にカンマ区切りにしています。
厳密に行うには [Papa Parse](https://www.papaparse.com) などの CSV パーサーを利用してください。
:::

現状ではデータセットを JSON でプロンプトに含めています。
JSON は中括弧やクォーテーションといった記号を多く含み、トークン数がかさんでしまいます。

```
# Tokens: 20
{ "name":  "simochee", "company": "Nulab Inc." }
```

```
# Tokens: 13
name,company
simochee,Nulab Inc.
```

ここでは CSV のようなカンマ区切りのフォーマットに変更してみます。

```diff
  const response = await ai.models.generateContent({
    model: 'gemini-2.5-flash',
    contents: `
福岡市の屋台で地鶏の炭火焼きが食べられるお店は？

以下のデータセットから回答して
+ ${dataset.fields.map(({ id }) => id).join(',')}
+ ${dataset.records.map((values) => values.join(',')).join('\n')}
- ${JSON.stringify(dataset)}
    `.trim(),
  });
}
```

実行した結果が次の通りです。

```json
{
  promptTokenCount: 131411,
  candidatesTokenCount: 554,
  totalTokenCount: 135942,
  promptTokensDetails: [ { modality: 'TEXT', tokenCount: 131411 } ],
  thoughtsTokenCount: 3977
}
```

わずかですが、入力トークンが減少しました。

## おすすめに不要なデータを除去する

現状、屋台IDなどの行政用データや住所などの屋台のおすすめに直接関係のない項目もプロンプトに含めてしまっています。
必要な項目を精査し、含める項目を減らします。

```js
import dataset from './dataset.json' with { type: 'json' };
import { GoogleGenAI } from '@google/genai';
const GEMINI_API_KEY = process.env.GEMINI_API_KEY;

const ai = new GoogleGenAI({ apiKey: GEMINI_API_KEY });

const records = dataset.records.map((values) => [
  values[0], // _id
  values[3], // 区名
  values[4], // エリア
  values[5], // カテゴリー
  values[9], // 名称
  values[44], // リード文
  values[46], // 本文
]);

async function main() {
  const response = await ai.models.generateContent({
    model: 'gemini-2.5-flash',
    contents: `
福岡市の屋台で地鶏の炭火焼きが食べられるお店は？

以下のデータセットから回答して
id,区名,エリア,カテゴリー,名称,リード文,本文
${records.map((values) => values.join(',')).join('\n')}
    `.trim(),
  });
}

await main();
```

実行した結果が次の通りです。

```json
{
  promptTokenCount: 39147,
  candidatesTokenCount: 162,
  totalTokenCount: 39877,
  promptTokensDetails: [ { modality: 'TEXT', tokenCount: 39147 } ],
  thoughtsTokenCount: 568
}
```

大幅にプロンプトのトークン数を削減することができました。

もし、キャッシュレス決済や貸切の可否なども含めたい場合は、 `records` に情報を追加すれば対応できます。

# おわりに

ここまでの実装で固定データをプロンプトに含めることで、 LLM が把握していない情報源から出力させることができました。

これに対して、数万件など大量のデータから LLM に出力させようとすると、データベースなどからある程度のデータを参照してプロンプトにデータを渡す RAG (検索拡張生成) が必要になります。

また、この記事ではトークン数やデータの形式を主として書きましたが、省略することで逆に精度が落ちることもあります。
前後のプロンプトやデータそのものの内容によっても結果が異なることがあるため、高い精度の生成を行うには試行錯誤が必要となります。

ただ、これだけの実装であいまい検索やちょっとしたレコメンデーションが実現できるのは、自作できるアイデアの幅が広がりますね！
