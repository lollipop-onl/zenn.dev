---
title: Vercel の料金形態と内容について調べてみた - 2020冬
emoji: 🍭
type: tech
topics: [vercel, EveOneZenn]
published: false
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.14 です。

Next.js などを手掛ける Vercel Inc.（旧 Zeit ）が運営しているサービス、Vercel（旧 Zeit Now）の料金形態とその内容についてまとめます。

https://vercel.com/

なお、掲載内容は 2020/12/08 時点のものです。
（過去に別所で公開していた記事の加筆版です）

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-ts-non-nullable

# Vercel について

Vercel は静的サイトとサーバレス機能のホスティングを提供するクラウドプラットフォームです。
JAMStack な Web サイトやサービスをホストしてアプリケーションを簡単にデプロイできます。

スケーリングやサーバーの監視は Vercel が行うため、開発者は Vercel へデプロイするだけでアプリケーションを公開・運用できます。

Vercel へデプロイできるアプリケーションのサンプルは GitHub のリポジトリに各種フレームワーク・ライブラリを使用した例が公開されています。

https://github.com/vercel/vercel/tree/master/examples

# 料金プラン

Vercel の料金プランには Hobby・Pro・Enterprise の 3 種類があります。
これらのプランは 2020/04/09 のアナウンスで発表されました。

https://vercel.com/blog/simpler-pricing

各プランに含まれる機能は次のページで説明されています。

https://vercel.com/pricing

# Hobby プラン

## 料金

無料（従量課金なし）

## 機能

次の機能は Hobby プラン・ Pro プラン・ Enterprise プラン すべてで利用できます。

ただし、一部機能ではプランごとに制限（後述）が設けられています。

* HTTPS 対応のカスタムドメイン
* GitHub・GitLab・BitBucket と連携した継続的デプロイメント
* 高機能なエッジネットワーク
* 無制限の Webサイト と API
* Node.js や Go を使用したサーバレス関数

# Pro プラン

## 料金

メンバーあたり $20/月（14日間の無料トライアルあり）

## 機能

次の機能は Pro プラン・ Enterprise プランで利用できます。

* 最大 10 人のチームメンバー
* 詳細な請求書設定
* [ビルドの並列化](https://vercel.com/docs/platform/users-and-teams?query=concurrent#concurrent-builds)（1 スロットにつき $50/月）
* [ブランチごとのプレビューデプロイメント](https://vercel.com/docs/platform/deployments#preview)（$100/月）
* [独自のパスワード保護](https://vercel.com/docs/platform/projects#password-protection)（$150/月）

# Enterprise プラン

## 料金

問い合わせ

## 機能

* 稼働率 99.99% の SLA
* サーバレス関数のマルチリージョン化
* エンタープライズサポート
* Next.js アプリケーションの監査

# 各種制限

Vercel の各機能に課された制限です。
ここでは主な制限のみをまとめます。

https://vercel.com/docs/platform/limits

## 上限

Vercel ではプランごとにサーバレス関数の数や 1 日あたりのデプロイ回数などに上限が設けられています。
なお、 Next.js の SSG 用のサーバレス関数およびプレビュー用のサーバレス関数は「デプロイごとのサーバレス関数」のカウントから除外されます。

|**|Hobby|Pro|Enterprise|
|:--|:--|:--|:--|
|1日あたりのデプロイ回数|100|300|カスタム|
|デプロイごとのサーバレス関数の数|12|24|カスタム|
|1ヶ月あたりのサーバレス関数のデプロイ回数|160|640|カスタム|
|サーバレス関数の実行期間|10秒|60秒|900秒|
|1週間あたりの CLI からのデプロイ回数|2,000|2,000|カスタム|
|チームごとのメンバー数|-|10|カスタム|

## サーバレス関数のスペック

サーバレス関数のスペックはプランごとに異なります。
なお、マルチリージョンの利用は Enterprise プランのみとなります。

|**|Hobby|Pro|
|:--|:--|:--|
|サイズ|最大50MB|最大50MB|
|メモリ|最大1,024MB|最大3,008MB|
|同時実行性|最大1,000|最大1,000|
|ペイロードサイズ|最大5MB|最大5MB|
|リージョン|--|--|

## 環境変数

環境変数はプロジェクトごとに最大 100 件登録できます。
また、すべての環境変数の名前と値のサイズは最大 4KB までに制限されます。

## レート制限

Vercel ではビルドやデプロイ、ドメイン操作などの回数に制限が設けられています。
ここでは、主なレートの制限のみをまとめます。

なお、サーバレス関数の実行については同時実行数が最大 1,000 であること以外に制限は設けられていません。

一部のレート制限は無料分と有料分で制限の内容が異なります。

|**|制限（無料）|制限（有料）|
|:--|:--|:--|
|1時間あたりのフックトリガーからのデプロイ回数|60|3,600|
|1日あたりのビルド回数|100|3,000|
|1時間あたりのビルド回数|32|512|
|1日あたりのデプロイ回数|100|3,000|
|1時間あたりのデプロイ回数|100|150|
|1日あたりのチーム作成数|5|25|
|1日あたりのチームメンバーの作成数|50|150|
|1日あたりのアップロード数|5,000|40,000|

# Hobby プランまとめ

Vercel は無料枠である Hobby プランでも、サーバレス関数の数や実行数をあまり気にせずしようできることがわかりました。

Hobby プランを使用するか Pro プランを使用するかの判断は、メンバーが複数人いるかかデプロイの頻度かなと思います。

最後に、 Hobby プランでできることと制限ををまとめます。

## 機能

* HTTPS 対応のカスタムドメイン
* GitHub・GitLab・BitBucket と連携した継続的デプロイメント
* 高機能なエッジネットワーク
* 無制限の Webサイト と API
* Node.js や Go を使用したサーバレス関数

## 制限

* サーバレス関数のサイズ：最大 50 MB
* サーバレス関数のメモリ：最大 1,024 MB
* サーバレス関数の同時実行性：最大 1,000
* サーバレス関数が受け取るペイロードサイズ：最大 5 MB
* 1日あたりのビルド数：100
* 1時間あたりのビルド数：32
* 1時間あたりのフックトリガーからのデプロイ回数：60
* 1日あたりのデプロイ数：100
* 1時間あたりのデプロイ数：100

# 参考

* [Develop. Preview. Ship. For the best frontend teams – Vercel](https://vercel.com/)
* [Simpler Pricing – Vercel](https://vercel.com/blog/simpler-pricing)
* [Pricing – Vercel](https://vercel.com/pricing)
* [Limits - Vercel Documentation](https://vercel.com/docs/platform/limits)
