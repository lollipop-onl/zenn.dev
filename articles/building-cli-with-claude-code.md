---
title: "Claude Code で Backlog CLI を実装しきった話"
emoji: "🐝"
type: "tech"
topics: ["claude", "cli", "backlog", "ai"]
published: false
---

## はじめに

Backlog の CLI ツール [bee](https://github.com/nulab/bee) を Claude Code で作りました。
93 のサブコマンドと 2,000 超のテストケースを約 1 週間で実装し、[ccusage](https://github.com/hotoku/ccusage) の計測では $800 相当のトークンを消費しています。

私が書いたコードは多くなく、役割はコードを書くことよりも AI が迷わない環境を作ることに寄っていました。
この記事では、`AGENTS.md` や Linter の設定など、AI にコードを任せる上で実際に効いたテクニックと、気づいたことを書きます。

## 前提

以前 Vibe Coding で作った [simochee/backlog-cli](https://github.com/simochee/backlog-cli) がベースになっていますが、実装漏れや不具合が多く実用には心許ない品質でした。
そこで Backlog API → CLI コマンドというパターンが明確なこの題材を、AI 向けの体裁を整えた上で作り直すことにしました。

コードのほとんどを Claude Code（モデルは Claude Opus 4.6）が書き、私は選定、統一感、レビュー、レール敷きを担当するという体制でした。

# AI にコードを任せるためにやったこと

## AGENTS.md でスタイルを統制する

https://github.com/nulab/bee/blob/main/AGENTS.md

最序盤に `CLAUDE.md`（現在の `AGENTS.md`）の初版を作り、以降はレビューで気づいたことをどんどん書き足すように Claude Code へ指示していました。

たとえば、成功メッセージのフォーマットがコマンドごとにバラバラだと気づいたら、こういう規約を追記していました。

```markdown
### Success message format

| Resource type | Pattern | Example |
|---|---|---|
| ID-based (category, status, ...) | `<Verb> <resource> <name> (ID: <id>)` | `Created category Bug (ID: 5)` |
| Key-based (issue, project, ...) | `<Verb> <resource> <key>: <name>` | `Created issue PROJ-1: Fix bug` |
```

以降の AI はこの規約に従って実装するようになり、レビューでの指摘が減っていきました。

こうした規約がないと、AI はコマンドごとに微妙に異なるスタイルでコードを書いていました。
レビューで指摘 → `AGENTS.md` に書く → 以降の AI が従う、のサイクルが開発全体を通してうまく回りました。

なお、後述する Linter のルールと内容が重複しても、どう書くかを `AGENTS.md` にも明記しておくと効果的でした。
Linter を厳しくすると CI で lint が失敗する頻度が上がりますが、同じルールが `AGENTS.md` にも書いてあれば AI が最初から正しいスタイルで書いてくれるので、CI のやり直しによるストレスが減りました。

## Linter で機械的にブレを防ぐ

https://github.com/nulab/bee/blob/main/.oxlintrc.json

Claude Code は `interface` と `type` のどちらを使うか、default export にするかといった、どちらでも動くけど統一感に影響する選択を頻繁にしていました。
表現がブレるとレビューで見るべき箇所が増え、地味にストレスでした。

実際、巻き上げの有無で関数の定義スタイルが変わったり、利用したライブラリのドキュメントに引っ張られて書き方がブレたりすることがありました。
個人的にどちらでもいいスタイルの話はかなり気になるので、[oxlint](https://oxc.rs/) で機械的に強制させました。

以下のようなルールを設定しました。

- [`typescript/consistent-type-definitions`](https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-type-definitions)
- [`typescript/consistent-type-imports`](https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-type-imports)
- [`import/no-default-export`](https://oxc.rs/docs/guide/usage/linter/rules/import/no-default-export)
- [`typescript/array-type`](https://oxc.rs/docs/guide/usage/linter/rules/typescript/array-type)
- [`typescript/consistent-indexed-object-style`](https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-indexed-object-style)

これらはどれも正解がないスタイルの問題です。
人間同士でもブレる箇所なので、AI に任せるなら Linter で潰しておくのが確実です。

なお、ESLint + Prettier ではなく oxlint + oxfmt を選んだのは、高速さが決め手でした。

oxlint は十分に高速なので、lefthook で push 前にフルチェックを走らせるのもよさそうだなと思っています。
autofix できないルールも push 前に検出できれば、CI で初めて気づくストレスを減らせます。

人間だけの開発では厳しすぎる Linter 設定はストレスになりがちですが、AI が書いたコードに対しては厳しくしておくほうが結果的に楽でした。
人間にとっては厳しすぎてやめるような設定も、AI 相手なら採用しやすくなります。

## モノレポのパッケージ分割で AI の迷いを減らす

bee では、 [Turborepo](https://turborepo.dev/) でモノレポを管理していました。

| パッケージ | 責務 |
|---|---|
| `cli` | コマンド定義とその処理 |
| `backlog-utils` | Backlog API クライアント・認証・Git コンテキスト取得 |
| `cli-utils` | 出力整形・テーブル・プロンプト・引数パース |
| `config` | `.beerc` の読み書き・スペース管理・設定スキーマ |
| `test-utils` | モック（API クライアント・consola）・テストヘルパー |

特に `cli-utils` と `backlog-utils` を分離したのは効果があったなと感じています。

Claude Code は指示によってはロジックやドメインを混在させがちで、CLI に直書きすべきでないユーティリティが紛れ込むことがありました。
パッケージ境界で強制的に分けておけば、動くけど置き場所が違うといったレビュー指摘を構造的に防げます。

また、誤ったところにコードを書いてしまっても、周辺の実装から正しい場所を推測しやすくなるため、リファクタリングの指示もシンプルなものにできました。

テストもパッケージ境界でモックしていました。

https://github.com/nulab/bee/blob/1c0fe8d3949a09edac31c048c9ea65a3e0c3c81d/apps/cli/src/commands/issue/list.test.ts#L7-L11

https://github.com/nulab/bee/blob/1c0fe8d3949a09edac31c048c9ea65a3e0c3c81d/apps/cli/src/commands/issue/list.test.ts#L33-L42

CLI コマンドのテストでは `@repo/backlog-utils` をまるごとモックし、コマンドが正しい引数で API を呼んでいるかだけを検証していました。

責務の混在を構造的に防ぐことで、Claude Code が間違った場所にコードを書くリスクを減らせました。

## Single Source of Truth で更新漏れを防ぐ

Claude Code との協業では、一度に複数箇所を変更しなければならない設計がトラブルの元になりやすかったです。

当初、コマンドのドキュメントを個別の Markdown で管理していましたが、定義漏れや修正漏れが頻発しました。
そこで、 bee では Commander.js のコマンド定義そのものをドキュメントの唯一のソースにしました。

https://github.com/nulab/bee/blob/1c0fe8d/apps/cli/src/commands/issue/list.ts#L25-L69

ドキュメントサイト側では、ビルド時に [Jiti](https://github.com/unjs/jiti) で CLI のコマンドファイルを直接 import し、Commander.js インスタンスからメタデータを抽出しました。

https://github.com/nulab/bee/blob/1c0fe8d/apps/docs/src/lib/commands.ts#L127-L185

この仕組みで `--help` の出力、ドキュメントサイト、llms.txt の 3 つが同じコマンド定義から自動生成されるようにしました。
- [Commands | Backlog CLI](https://nulab.github.io/bee/commands/)
- [llms.txt](https://nulab.github.io/bee/llms.txt)
- [llms-full.txt](https://nulab.github.io/bee/llms-full.txt)

AI がコマンドを追加・修正するときに触るのはコマンドファイルだけなので、ドキュメントの更新漏れが構造的に起きなくなりました。

これによって、レビュー時に確認するべき場所がコマンドファイルだけに絞られ、レビューの負荷が大幅に減りました。

## Git Worktree で並列開発する

bee では Git Worktree を使い、最大6本の worktree を同時に走らせていました。
コマンドグループ（issue, project, wiki など）を read 系と write 系にさらに分けて、それぞれの worktree で独立した Claude Code セッションを立てて開発しました。

worktree ごとにセッションを分けることで、コンテキストが混ざらず、`AGENTS.md` のパターンに沿った横展開を並列で進められました。

この並列開発の頻度が高かったことが、CI やローカルチェックの速さを重視する理由にもなりました。

## difit によるローカルレビューループ

difit はローカルで差分をレビューできるツールです。
https://github.com/yoshiko-pg/difit

Claude Code にはカスタムコマンド（Skill）を追加できる仕組みがあり、`/difit` スキルを使うと現在の差分の適切な範囲でレビュー画面を開いてくれます。

![difit のレビュー画面。左にファイルツリー、右にコードの差分とコメント入力欄が表示されている](https://storage.googleapis.com/zenn-user-upload/6795dadbdc0f-20260310.png)

レビュー画面を閉じると、指摘内容が自動的にエージェントに渡り、修正まで実行してくれます。
このレビュー → 自動修正のループが快適で、レビューの心理的ハードルが下がりました。
push 前にチェックでき、任意の差分範囲のコードも確認できるので、worktree 並列開発との相性も良かったです。

## AI の知識を引き出す

開発中にもっとスマートなやり方がないかなと感じたとき、その疑問をそのまま聞いてみていました。

たとえば、オプションの `description` に値の候補やフォーマットまで詰め込んでいましたが、冗長さを感じて聞いてみたところ、citty がサポートする `valueHint` で分離できることを教えてくれました。開発の半ばまでこの機能の存在に気づいていませんでした。

```ts
// Before: description に値の候補を詰め込んでいた
defineCommand({
  args: {
    format: { type: "string", description: "Output format (table, json)" },
  },
});

// After: valueHint に分離
defineCommand({
  args: {
    format: { type: "string", description: "Output format", valueHint: "table | json" },
  },
});
```

もうひとつ、`BACKLOG_PROJECT` 環境変数のフォールバックを `resolveProject` 関数で手書きしていましたが、冗長さを感じて聞いてみたところ、Commander.js の `.env()` メソッドの存在を教えてくれました。

```ts
// Before: ヘルパー関数でフォールバック
const project = resolveProject(args.project); // process.env.BACKLOG_PROJECT ?? args.project

// After: Commander.js の .env() で宣言的に解決
new RequiredOption("-p, --project <id>", "...").env("BACKLOG_PROJECT");
```

Claude Opus でも、知識は豊富に持っているのに聞かれるまで出してこないことが多かったです。
人間の勘や冗長さへの違和感が、AI の知識を引き出すトリガーになりました。

さらに、引き出した知見を `AGENTS.md` にフィードバックすることで、以降の実装にも反映されていきました。

## テストが揃えば大規模な変更も任せられる

今回、いくつかの大規模な変更が発生しましたが、いずれもかなりシンプルな指示から Claude Code が作業を完遂してくれました。

たとえば、コマンドパーサーを [citty](https://github.com/unjs/citty) から [Commander.js](https://github.com/tj/commander.js) へほぼ完成後に移行する必要が出たときには、テストを頼りに移行をやりきってくれました。
https://github.com/nulab/bee/pull/53

また、 `Number()` を valibot スキーマに全面置換する作業、`--space` フラグを 85 以上のコマンドに一括追加する作業も同様に、テストがあることで安心して任せることができました。
https://github.com/nulab/bee/pull/72
https://github.com/nulab/bee/pull/75

いずれも `AGENTS.md` でパターンが定まっているからこそ、AI が迷わず横展開できた例です。

# おわりに

レールを敷く → AI が走る → 発見をレールに戻す。このサイクルを回すことが bee を実装する上での私の仕事の大半を占めていました。
Vibe Coding で作った元の backlog-cli がそうだったように、人間がガードレールを引かなければ AI がモノを作りきるのはまだ難しいと感じています。

一方で、 AI に任せる部分を増やすほど、コードを書くことよりもモノができ上がっていくのが好きだったんだ、と気付くことになりました。
AI と協業して、これまで一人で作りきれなかった規模のプロジェクトを作りきる体験はとても楽しかったです。

## Try it out!

bee は RC 版を公開しています。ぜひ試してみていただき、GitHub Issue や Discussions でフィードバックをいただけると嬉しいです！

```sh
npm i -g @nulab/bee@rc
```

https://github.com/nulab/bee

https://nulab.github.io/bee
