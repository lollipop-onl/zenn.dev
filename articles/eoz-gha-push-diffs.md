---
title: GitHub Actions でワークフロー中に発生した差分を Push する
emoji: 🍭
type: tech
topics: [githubactions, EveOneZenn]
published: true
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.11 です。

GitHub Actions でワークフローのジョブ実行中に発生した差分をリポジトリへ Push する方法を紹介します。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-eslint-failure-warning

# GitHub Actions で差分を Push するステップ

早速ですが、 GitHub Actions で差分を Push するには次のような step を追加します。

```yml
      - name: 差分を push
        run: |
          git remote set-url origin https://github-actions:${GITHUB_TOKEN}@github.com/${GITHUB_REPOSITORY}
          git config --global user.name "${GITHUB_ACTOR}"
          git config --global user.email "${GITHUB_ACTOR}@users.noreply.github.com"
          if (git diff --shortstat | grep '[0-9]'); then \
            git add .; \
            git commit -m "GitHub Actions から差分を Push"; \
            git push origin HEAD:${GITHUB_REF}; \
          fi
```

これで、ワークフローの実行のきっかけとなったユーザー名義で差分を Push できます。

# 解説

`run` で実行しているコマンドをそれぞれ解説します。

まず、冒頭 3行 で Push する Git ユーザーをセットアップしています。

```sh
git remote set-url origin https://github-actions:${GITHUB_TOKEN}@github.com/${GITHUB_REPOSITORY}
git config --global user.name "${GITHUB_ACTOR}"
git config --global user.email "${GITHUB_ACTOR}@users.noreply.github.com"
```

続く `if` では差分が存在しているかチェックしています。
`git diff --shortstat` は、差分が存在する場合は `1 file changed, 8 insertions(+), 5 deletions(-)` のような情報を返し、差分がない場合は何も返しません。
この差を利用して、続く `grep` で出力内容に数字が含まれるかで判定しています。

この条件を抜くと `git push` 時に、変更が無い旨のエラーになってしまいます。

```sh
if (git diff --shortstat | grep '[0-9]'); then \
  # ...
fi
```

最後に、 `if` の内部では通常の `git add` → `git commit` → `git push` の手順でリポジトリへ差分を反映しています。

```sh
if (git diff --shortstat | grep '[0-9]'); then \
  git add .; \
  git commit -m "GitHub Actions から差分を Push"; \
  git push origin HEAD:${GITHUB_REF}; \
fi
```

# 使い所

使い所としては、各種 Linter で Auto fix された内容を自動的に取り込む、みたいなケースです。

たとえば、次のワークフローでは textlint を実行し自動修正できた差分をリポジトリに Push しています。

```yml:textlint.yml
name: textlint
on:
  push:
    paths:
      - '**.md'
      - '.github/workflows/textlint.yml'
      - '.textlintrc'

jobs:
  build:
    name: textlint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 1
      - uses: actions/setup-node@v1
        with:
          node-version: 12.x
      - run: yarn install --ignore-scripts
      - name: Check textlint (with auto-fix)
        run: yarn textlint "./articles/*.md" --fix
        # --fix 付きではエラーがレポートされないので修正された状態で再度チェック
      - name: Check textlint
        run: yarn textlint "./articles/*.md"
      - name: Push auto-fixed files
        run: |
          git remote set-url origin https://github-actions:${GITHUB_TOKEN}@github.com/${GITHUB_REPOSITORY}
          git config --global user.name "${GITHUB_ACTOR}"
          git config --global user.email "${GITHUB_ACTOR}@users.noreply.github.com"
          if (git diff --shortstat | grep '[0-9]'); then \
            git add .; \
            git commit -m "👕 Fixed auto-fixable textlint errors by github-actions"; \
            git push origin HEAD:${GITHUB_REF}; \
          fi
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

例として示した textlint の他にも ESLint 、 stylelint など各種 Linter で同様に修正分の反映が自動化できます。

# 参考

* [GitHub Actions のコンテキストおよび式の構文 - GitHub Docs](https://docs.github.com/ja/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions)
