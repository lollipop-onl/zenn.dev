---
title: GitLab のフロントエンド開発チェックリストが真理だったのでまとめる
emoji: 🍭
type: tech
topics: [gitlab, frontend, EveOneZenn]
published: false
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol. です。

GitLab のドキュメントに記載されている Frontend Development Process 内の Development Checklist がとても参考になる内容だったのでまとめます。

★ 誤訳などあればコメントまでお問い合わせいただきたいです。

https://docs.gitlab.com/ee/development/fe_guide/development_process.html

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-nuxt-dev-memory-leak

# 開発チェックリスト

このチェックリストは新しい機能を構築したり、なにかを始めたりするときに特定のトピックをリマインドするためのものです。
チェックリストはパイロットのような他の業界でもよく使われるもので、標準化されたチェックリストを使えば問題を早期に減らせます。

なお、このリストは大きな機能やリファクタリングといった開発を対象としており、「常に使用してすべての項目をパスする必要がある」リストではありません。

# 開発計画

## あなたの見積もりは実際の課題の工数感と合っていますか？

> Check the current set weight of the issue, does it fit your estimate?

## あなたから見て

> Are all [departments](https://about.gitlab.com/handbook/engineering/#engineering-teams) that are needed from your perspective already involved in the issue? (For example is UX missing?)
