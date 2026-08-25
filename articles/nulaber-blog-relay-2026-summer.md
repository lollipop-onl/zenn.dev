---
title: GitHub Actions で Backlog 課題と GitHub プルリクエストを連動させる
emoji: 🍭
type: tech
topics: [backlog, github, githubactions, cli]
published: true
---

この記事は *ヌーラボブログリレー2026夏* の7日目<!--- Part 2 -->として投稿しています。

![ブログリレーのサムネイル](/images/nulaber-blog-relay-2026-summer/thumbnail.png)

<!--
> Part 1 → [ターミナルから Backlog を操作できる CLI「bee」を公開しました](https://nulab.com/ja/blog/nulab/released-backlog-cli-bee/)
-->

GitHub と Backlog を併用していると、プルリクエストを作ったら課題を「処理中」に、マージしたら「完了」に、と Backlog 側を手で更新することになります。

この記事では、GitHub Actions と [bee CLI](https://nulab.github.io/bee/) ( Backlog CLI ) を組み合わせて、この更新を自動化する方法を紹介します。
他の運用方法にも応用できるので、ぜひ参考にしてみてください。

## 今回実装するワークフロー

`EXAMPLE-123/implement-foo` のような課題キーがブランチ名の先頭に付いたプルリクエストを対象に、次のような連動を行います。

| GitHub プルリクエストの状態 | Backlog 課題の状態 | 課題へのコメント |
| --- | --- | --- |
| プルリクエスト作成 | 処理中 | ○ |
| プルリクエストマージ | 完了 | ○ |
| プルリクエストクローズ（マージされず） | 変更なし | ○ |
| プルリクエスト再オープン | 処理中 | ○ |

## 準備するもの

準備は以下の 2 つだけです。

1. [Backlog API キーを取得する](https://support-ja.backlog.com/hc/ja/articles/360035641754)
2. GitHub リポジトリのシークレットとして API キーを `BACKLOG_API_KEY` という名前で登録する

## ワークフローファイル

以下のワークフローを `.github/workflows/nulaber-blog-relay-2026-summer.yml` として作成します。

ワークフローでは以下の環境変数を使用します。

- `BACKLOG_API_KEY` - Backlog API キー（GitHub シークレットに登録したもの）
- `BACKLOG_SPACE` - Backlog スペースのドメイン（例: `example.backlog.com`）
- `STATUS_IN_PROGRESS` - 「処理中」状態の ID（初期状態で良い場合は `2` 、カスタムステータスを使う場合は、 [bee status list](https://nulab.github.io/bee/commands/status/list/) などで ID を確認）

```yaml
name: Sync PR Lifecycle to Backlog Issue
on:
  pull_request:
    types: [opened, reopened, closed]
    branches: [main]

# Backlog 側だけを更新するので GITHUB_TOKEN の権限は一切不要
permissions: {}

env:
  BACKLOG_API_KEY: ${{ secrets.BACKLOG_API_KEY }}
  BACKLOG_SPACE: example.backlog.com
  STATUS_IN_PROGRESS: "2"

jobs:
  sync:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      # ブランチ名の先頭から課題キーを取り出す（EXAMPLE-123/implement-foo → EXAMPLE-123）
      - name: Extract issue key from branch name
        id: issue
        run: echo "key=$(echo "$BRANCH" | grep -oE '^[A-Z][A-Z0-9_]*-[0-9]+')" >> "$GITHUB_OUTPUT"
        env:
          BRANCH: ${{ github.head_ref }}

      # 後続のステップで bee コマンドを使えるようにする
      - name: Install bee CLI
        if: steps.issue.outputs.key != ''
        run: npm install -g @nulab/bee@1

      # コメント本文に埋め込むプルリクエストへのリンク
      - name: Build PR link text
        id: pr-link
        if: steps.issue.outputs.key != ''
        run: echo "md=[${REPO}#${PR_NUMBER}](${PR_URL})" >> "$GITHUB_OUTPUT"
        env:
          REPO: ${{ github.repository }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          PR_URL: ${{ github.event.pull_request.html_url }}

      # プルリクエスト作成 → 課題を「処理中」にして、どのブランチのプルリクエストとリンクしたかを残す
      - name: Link issue to the opened PR
        if: steps.issue.outputs.key != '' && github.event.action == 'opened'
        run: |
          bee issue edit "$KEY" --status "$STATUS_IN_PROGRESS" \
            --comment "GitHub プルリクエスト $PR_LINK とリンクしました（$HEAD → $BASE）"
        env:
          KEY: ${{ steps.issue.outputs.key }}
          PR_LINK: ${{ steps.pr-link.outputs.md }}
          HEAD: ${{ github.head_ref }}
          BASE: ${{ github.base_ref }}

      # プルリクエスト再オープン → レビュー再開なので課題も処理中に戻す
      - name: Mark issue as in progress again on PR reopen
        if: steps.issue.outputs.key != '' && github.event.action == 'reopened'
        run: |
          bee issue edit "$KEY" --status "$STATUS_IN_PROGRESS" \
            --comment "GitHub プルリクエスト $PR_LINK が再オープンされました"
        env:
          KEY: ${{ steps.issue.outputs.key }}
          PR_LINK: ${{ steps.pr-link.outputs.md }}

      # プルリクエストマージ → 課題をクローズ
      - name: Close issue on merge
        if: steps.issue.outputs.key != '' && github.event.action == 'closed' && github.event.pull_request.merged == true
        run: |
          bee issue close "$KEY" \
            --comment "GitHub プルリクエスト $PR_LINK が $BASE にマージされました"
        env:
          KEY: ${{ steps.issue.outputs.key }}
          PR_LINK: ${{ steps.pr-link.outputs.md }}
          BASE: ${{ github.base_ref }}

      # マージされずにクローズ → 対応は済んでいないので記録だけ残し、ステータスは変えない
      - name: Note that the PR was closed without merging
        if: steps.issue.outputs.key != '' && github.event.action == 'closed' && github.event.pull_request.merged == false
        run: |
          bee issue comment "$KEY" \
            --body "GitHub プルリクエスト $PR_LINK はマージされずにクローズされました"
        env:
          KEY: ${{ steps.issue.outputs.key }}
          PR_LINK: ${{ steps.pr-link.outputs.md }}
```

## 実際の動き

この記事のワークフローを実際に動かしたデモリポジトリを公開しています。
スクリーンショットは、 [lollipop-onl/backlog-pr-sync-demo#5](https://github.com/lollipop-onl/backlog-pr-sync-demo/pull/5) の作成からマージまでを動かしたときのものです。

https://github.com/lollipop-onl/backlog-pr-sync-demo

課題キーで始まるブランチからプルリクエストを作成すると、課題が「処理中」になり、プルリクエストへのリンクがコメントとして残ります。

![プルリクエスト作成時に課題へ投稿されたコメント。状態が未対応から処理中に変わり、プルリクエストへのリンクとブランチ名が記録されている](/images/nulaber-blog-relay-2026-summer/comment-on-open.png)

プルリクエストをマージすると、課題が「完了」になります。

![プルリクエストのマージ時に課題へ投稿されたコメント。状態が処理中から完了に変わり、main にマージされたことが記録されている](/images/nulaber-blog-relay-2026-summer/comment-on-merge.png)

ステータスの変更とコメントが 1 つの履歴にまとまっているので、いつ・どのプルリクエストで状態が変わったのかを後から追いやすくなっています。

## 解説

### ステータス変更とコメントは 1 コマンドで行う

最初はステータス変更 ( `issue edit` ) とコメント追加 ( `issue comment` ) を分けて書いていたのですが、プルリクエストを再オープンしたときだけワークフローが失敗する現象に当たりました（[実際に失敗した run](https://github.com/lollipop-onl/backlog-pr-sync-demo/actions/runs/32757976307)）。

```json
{"errors":[{"message":"No comment content.","code":7,"moreInfo":""}]}
```

Backlog API は、ステータスの変わらない更新をコメントなしでは受け付けません。
クローズした直後に再オープンした場合、課題は「処理中」のままなので同じステータスへの更新となり、このエラーになります。

`issue edit` は `--comment` オプションでステータス変更とコメント投稿を一度に行えます。
コメントの内容があれば同一ステータスでも更新できるため、このワークフローでは 1 コマンドにまとめました。
先ほどのスクリーンショットのように状態変更とコメントが 1 つの履歴にまとまるようになったのは、この修正のおかげです。

### ブランチ名は環境変数を経由して渡す

ブランチ名は、プルリクエストの作成者が自由に決められる値です。
`run` の中に `${{ github.head_ref }}` を直接展開すると、ブランチ名がシェルコマンドとして解釈される余地が生まれてしまいます。
`env` に渡して `"$BRANCH"` として参照すれば、シェルからは単なる文字列として扱われます。

https://docs.github.com/ja/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#understanding-the-risk-of-script-injections

また、このワークフローは GitHub 側には何も書き込まないため、 `permissions: {}` を指定して `GITHUB_TOKEN` の権限をすべて落としています。

### 課題キーがないブランチでは何もしない

`renovate/...` のような課題キーで始まらないブランチでは、最初のステップで `key` が空になり、後続のステップは `if` の条件によってすべてスキップされます。

なお、 `grep` はマッチしないと終了コード 1 を返しますが、ステップの終了コードは `echo` のものになるため、ここでワークフローが失敗することはありません。
`KEY=$(echo "$BRANCH" | grep -oE ...)` のような変数代入の形に書き換えると、 `grep` の終了コードがそのままステップの結果になり、ワークフローが失敗するようになってしまいます。

## 運用に合わせて変える

### マージ時に「処理済み」で止める

マージ時に課題を「完了」ではなく「処理済み」で止めたい場合は、 `issue close` を `issue edit` に差し替えます。

```yaml
          bee issue edit "$KEY" --status "3" \
            --comment "GitHub プルリクエスト $PR_LINK が $BASE にマージされました"
```

なお、 `issue close` でクローズした課題の完了理由は「対応済み」になります。
別の完了理由を設定したい場合は `--resolution` オプションが利用できます。

https://nulab.github.io/bee/commands/issue/close/

### プルリクエストへのリンクをカスタム属性に登録する

コメントとして流れていくだけでなく、課題の属性としてプルリクエストへのリンクを持たせておくこともできます。
カスタム属性の更新は bee の専用コマンドでは対応していませんが、 `bee api` から Backlog API を直接呼び出せます。

```yaml
      # プルリクエスト作成 → カスタム属性に PR リンクを登録
      - name: Save PR link to a custom field
        if: steps.issue.outputs.key != '' && github.event.action == 'opened'
        run: |
          bee api -X PATCH "issues/$KEY" \
            -F "customField_12345=$PR_LINK"
        env:
          KEY: ${{ steps.issue.outputs.key }}
          PR_LINK: ${{ steps.pr-link.outputs.md }}
```

`customField_` に続く数値はカスタム属性の ID で、 `bee api projects/<PROJECT>/customFields` で確認できます。
Markdown のプロジェクトであれば、 `[表示名](URL)` 形式の値はカスタム属性でもリンクとして表示されます。

![課題の属性欄に表示されたカスタム属性「関連 PR」。プルリクエストへのリンクがリンクとして表示されている](/images/nulaber-blog-relay-2026-summer/custom-field.png)

https://nulab.github.io/bee/commands/api/

## おわりに

このワークフローの Backlog 操作は `issue edit` ・ `issue close` ・ `issue comment` の 3 コマンドだけなので、コメントの文面もステータスの遷移も、運用に合わせて自由に組み替えられます。
プルリクエストの本文から課題キーを拾う、レビュー依頼で担当者を切り替える、リリース時にマイルストーンを付ける、といった拡張も同じ要領でできそうです。
まずはこのワークフローをそのまま置いて、チームの運用に合わせて少しずつ育ててみてください。
