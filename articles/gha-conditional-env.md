---
title: GitHub Actions で動的な環境変数を実現する
emoji: 🍬
type: tech
topics: [githubactions]
published: true
---

# やりたいこと

GitHub Actions のワークフローで、ある値を元に環境変数を切り替えたくなるときがたまにあります。

たとえば、 `workflow_dispatch` で `inputs` の指定をもとに環境変数が切り替えられると、一つの指定に対して複数の値を決定できるので入力をシンプルにできます。

```yaml
# .github/actions/deploy-web.yml
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        description: デプロイする環境
        required: true
        options:
          - 開発
          - 検証
          - 本番

env:
  NEXT_PUBLIC_API_ORIGIN: |
    environment == 開発 -> https://dev.api.example.com
    environment == 検証 -> https://stg.api.example.com
    environment == 本番 -> https://api.example.com
```

これを実現するアイデアをご紹介します。

なお、この記事で紹介する表現は環境変数だけではなく、アクションの引数やコマンドの一部としても同様の実装を利用できます。

# fromJSON を使う

GitHub Actions のワークフローでは `fromJSON` という文字列を JSON に変換するメソッドが利用できます。

https://docs.github.com/ja/enterprise-cloud@latest/actions/learn-github-actions/expressions#fromJSON

`fromJSON` で JSON にしたものは JavaScript での操作と同様、プロパティアクセスができます。

これを使うことで、プロパティに条件となる文字列を設定した JSON から値を取得して期待の挙動を実現できます。

```yaml
# .github/actions/deploy-web.yml
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        description: デプロイする環境
        required: true
        options:
          - 開発
          - 検証
          - 本番

env:
  NEXT_PUBLIC_API_ORIGIN: |
    ${{ fromJSON('{
      "開発": "https://dev.api.example.com",
      "検証": "https://stg.api.example.com",
      "本番": "https://api.example.com"
    }')[github.event.inputs.environment] }}\
```

:::message
環境変数の末尾に改行を含めないために `\` の指定が必須です。
:::

# デフォルト値を指定する

設定されうる値を全て列挙しない場合にはデフォルト値の指定が必要になることがあります。

プロパティアクセスで値が参照できなかった場合のデフォルト値の指定は JavaScript と同様、 `||` で行います。

```yaml
env:
  # 検証・本番以外では開発の値を使用したい
  NEXT_PUBLIC_API_ORIGIN: |
    ${{ fromJSON('{
      "検証": "https://stg.api.example.com",
      "本番": "https://api.example.com"
    }')[github.event.inputs.environment] || 'https://dev.api.example.com' }}\
```

# 値にシークレットや他の環境変数を使う

`fromJSON` の値にシークレットや他の環境変数を使いたい場合は `format` という文字列中のプレースホルダーを置き換えされるメソッドを利用します。

https://docs.github.com/ja/actions/learn-github-actions/expressions#format

```yaml
env:
  STAGING_API_ORIGIN: https://stg-api.example.com
  # 検証・本番以外では開発の値を使用したい
  NEXT_PUBLIC_API_ORIGIN: |
    ${{ fromJSON(
      format(
        '{{
          "検証": "{0}",
          "本番": "{1}"
        }}',
        env.STAGING_API_ORIGIN,
        secrets.PRODUCTION_API_ORIGIN
      )
    )[github.event.inputs.environment] || 'https://dev.api.example.com' }}\
```

:::message
`format` メソッドの引数内では `{` と `}` が特殊文字として扱われるため、JSONのカッコは `{{` と `}}` にする必要があります。
:::

煩雑な記述になってしまいますが、ソースコード中に記載したくない値なども同様の仕組みに組み込むことができます。

# 参考

https://github.com/orgs/community/discussions/25725

https://docs.github.com/ja/actions/learn-github-actions/expressions

https://docs.github.com/ja/actions/security-guides/encrypted-secrets
