---
title: "Claude Code と AGENTS.md で Backlog CLI を実装しきった話"
emoji: "🐝"
type: "tech"
topics: ["claude", "cli", "typescript", "ai"]
published: false
---

## はじめに

Backlog の CLI ツール [bee](https://github.com/nulab/bee) を Claude Code で作りました。
自分が書いたコードは多くなく、役割は「コードを書くこと」よりも「AI が迷わない環境を作ること」に寄っていました。

この記事では、`AGENTS.md` や Linter の設定など、AI にコードを任せる上で実際に効いたテクニックと、やってみて気づいたことを書きます。

# 前提

bee は GitHub CLI のように Backlog をターミナルから操作する CLI です。

```sh
bee issue list --assignee @me --status 処理中
bee issue view PROJECT-123 --web
```

以前 Vibe Coding で作った [simochee/backlog-cli](https://github.com/simochee/backlog-cli) がベースになっていますが、実装漏れや不具合が多く実用には心許ない品質でした。
そこで「Backlog API → CLI コマンド」というパターンが明確なこの題材を、AI 向けの体裁を整えた上で作り直すことにしました。

コードのほとんどを Claude Code（モデルは Claude Opus 4.6）が書き、自分は「選定」「統一感」「レビュー」「レール敷き」を担当するという体制です。
結果として約 90 のサブコマンドと 1,300 超のテストケースを、約 1 週間で実装しました。

# AI が迷わないレールを敷く

## AGENTS.md 駆動の開発

最序盤に `CLAUDE.md`（現在の `AGENTS.md`）の初版を作りました。
以降、レビューで気づいたことをどんどん書き足しています。

たとえば、成功メッセージのフォーマットがコマンドごとにバラバラだと気づいたら、こういう規約を追記します。

```markdown
### Success message format

| Resource type | Pattern | Example |
|---|---|---|
| ID-based (category, status, ...) | `<Verb> <resource> <name> (ID: <id>)` | `Created category Bug (ID: 5)` |
| Key-based (issue, project, ...) | `<Verb> <resource> <key>: <name>` | `Created issue PROJ-1: Fix bug` |
```

以降の AI はこの規約に従って実装するようになり、レビューでの指摘が減っていきます。

引数の `description` と `valueHint` の書き分けルールも同様です。
（`valueHint` は citty にあった機能で、Commander.js への移行後は独自の仕組みとして引き継いでいます。）

```markdown
### Writing argument descriptions

- **`description` is pure prose** — keep it a short, human-readable explanation.
  Do not embed format hints, examples, or choice lists in the description.
- **Use `valueHint` for supplementary value information** — choices, formats,
  examples, and type hints go in the `valueHint` property.
```

こうした規約がないと、AI はコマンドごとに微妙に異なるスタイルでコードを書きます。
「レビューで指摘 → `AGENTS.md` に書く → 以降の AI が従う」のサイクルが開発全体を通してうまく回りました。

なお、後述する Linter のルールと内容が重複しても、「どう書くか」を `AGENTS.md` にも明記しておくと効果的でした。
Linter を厳しくすると CI で lint が失敗する頻度が上がりますが、同じルールが `AGENTS.md` にも書いてあれば AI が最初から正しいスタイルで書いてくれるので、CI のやり直しによるストレスが減ります。

約 400 行になりましたが、この規模なら「ドキュメントとして妥当」な文量に収まっています。

## Linter で選択肢を潰す

AI は「`interface` と `type` のどちらを使うか」「default export にするか」といった、どちらでも動くけど統一感に影響する選択を頻繁にします。
表現がブレるとレビューで見るべき箇所が増え、地味にストレスです。

実際、巻き上げの有無で関数の定義スタイルが変わったり、利用したライブラリのドキュメントに引っ張られて書き方がブレたりすることがありました。
マジでどちらでもいい話だからこそ気になるので、[oxlint](https://oxc.rs/) で機械的に強制させました。

設定したルールはこのあたりです。

- [`typescript/consistent-type-definitions`](https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-type-definitions)
- [`typescript/consistent-type-imports`](https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-type-imports)
- [`import/no-default-export`](https://oxc.rs/docs/guide/usage/linter/rules/import/no-default-export)
- [`typescript/array-type`](https://oxc.rs/docs/guide/usage/linter/rules/typescript/array-type)
- [`typescript/consistent-indexed-object-style`](https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-indexed-object-style)

これらはどれも「正解がない」スタイルの問題です。
人間同士でもブレる箇所なので、AI に任せるなら Linter で潰しておくのが確実です。

ちなみに ESLint + Prettier ではなく oxlint + oxfmt を選んだのは、 Git Worktree や Claude Code on the Web を使った並列開発で PR ごとに CI が回る頻度が高く、CI の速さが体験に直結したからです。

## モノレポのパッケージ分割

[Turborepo](https://turborepo.dev/) でモノレポを管理しています。

| パッケージ | 責務 |
|---|---|
| `cli` | コマンド定義とその処理 |
| `backlog-utils` | Backlog API クライアント・認証・Git コンテキスト取得 |
| `cli-utils` | 出力整形・テーブル・プロンプト・引数パース |
| `config` | `.beerc` の読み書き・スペース管理・設定スキーマ |
| `test-utils` | モック（API クライアント・consola）・テストヘルパー |

特に `cli-utils` と `backlog-utils` の分離が効きました。
AI はロジックやドメインを混在させがちで、CLI に直書きすべきでないユーティリティが紛れ込むことがあります。
パッケージ境界で強制的に分けておけば、「動くけど置き場所が違う」というレビュー指摘を構造的に防げます。

テストもパッケージ境界でモックしています。

```ts
const mockClient = {
  getIssues: vi.fn(),
  getMyself: vi.fn().mockResolvedValue({ id: 99 }),
  getProjects: vi.fn().mockResolvedValue([{ id: 123, projectKey: "PROJ" }]),
};

vi.mock("@repo/backlog-utils", async (importOriginal) => ({
  ...(await importOriginal()),
  getClient: vi.fn(() => Promise.resolve({ client: mockClient, host: "example.backlog.com" })),
}));

it("displays issue list in tabular format", async () => {
  mockClient.getIssues.mockResolvedValue(sampleIssues);
  const { default: list } = await import("./list");
  await list.parseAsync([], { from: "user" });
  expect(mockClient.getIssues).toHaveBeenCalled();
});
```

CLI コマンドのテストでは `@repo/backlog-utils` をまるごとモックし、コマンドが正しい引数で API を呼んでいるかだけを検証します。
責務の混在を構造的に防ぐことで、AI が間違った場所にコードを書くリスクを減らせました。

## Single Source of Truth

AI との協業では「一度に複数箇所を変更しなければならない設計」がトラブルの元です。

当初、コマンドのドキュメントを個別の Markdown で管理していましたが、定義漏れや修正漏れが頻発しました。
そこで、Commander.js のコマンド定義そのものをドキュメントの唯一のソースにしています。

```ts
const list = new BeeCommand("list")
  .summary("List issues")
  .description("By default, sorted by last updated date in descending order.")
  .addOption(opt.project())
  .addOption(opt.assigneeList())
  .examples([
    { description: "List issues in a project", command: "bee issue list -p PROJECT" },
    { description: "List your assigned issues", command: "bee issue list -p PROJECT -a @me" },
  ])
  .action(async (opts) => { /* ... */ });
```

ドキュメントサイト側では、ビルド時に [Jiti](https://github.com/unjs/jiti) で CLI のコマンドファイルを直接 import し、Commander.js インスタンスからメタデータを抽出しています。

```ts
const jiti = createJiti(import.meta.url, { moduleCache: true, fsCache: true });
const mod = await jiti.import(filePath);
const cmd = mod.default; // BeeCommand インスタンス
```

この仕組みで `--help` の出力、ドキュメントサイト、llms.txt の 3 つが同じコマンド定義から自動生成されます。
- [Commands | Backlog CLI](https://nulab.github.io/bee/commands/)
- [llms.txt](https://nulab.github.io/bee/llms.txt)
- [llms-full.txt](https://nulab.github.io/bee/llms-full.txt)

AI がコマンドを追加・修正するときに触るのはコマンドファイルだけなので、ドキュメントの更新漏れが構造的に起きません。

## Git Worktree を使った並列開発

基盤の実装と `AGENTS.md` の整備が一通り済んだ段階で、Git Worktree を使った 6 並列開発に移行しました。

あらかじめ `PLAN.md` にタスクを書き出しておき、各 worktree のセッションで main にあるプランのパスと TODO の ID を指定して作業を開始させます。
タスクは issue、wiki、project などコマンドグループ単位で分けました。
規約と構造が整っていれば、あとは走らせて待つだけです。
6 つのターミナルで同時に Claude Code がコードを書いている光景は壮観でした。

マージ時はコンフリクトが大量に発生しましたが、そこも Claude Code が対応してくれたので、Squash Merge での苦しみはほとんどありませんでした。

## difit によるローカルレビューループ

difit はローカルで差分をレビューできるツールです。
https://github.com/yoshiko-pg/difit

Claude Code にはカスタムコマンド（Skill）を追加できる仕組みがあり、`/difit` スキルを使うと現在の差分の適切な範囲でレビュー画面を開いてくれます。

レビュー画面を閉じると、指摘内容が自動的にエージェントに渡り、修正まで実行してくれます。
この「レビュー → 自動修正」のループが快適で、レビューの心理的ハードルが下がりました。
push 前にチェックでき、任意の差分範囲のコードも確認できるので、worktree 並列開発との相性も良いです。

## AI の「引き出し」を開ける

開発中に「もっとスマートなやり方がないかな」と感じたとき、その疑問をそのまま聞いてみます。

たとえば、各コマンドで `--project` や `--assignee` の説明文がバラバラに書かれていました。
「共通化できないか」と聞いたところ、共通オプションを関数として切り出す構成を提案してくれました。

```ts
// apps/cli/src/lib/common-options.ts
// RequiredOption は Commander.js の Option を拡張した bee 独自のクラス
const project = () =>
  new RequiredOption("-p, --project <id>", "Project ID or project key").env("BACKLOG_PROJECT");
const assigneeList = () =>
  new Option("-a, --assignee <id>", "Assignee user ID (repeatable). Use @me for yourself.")
    .argParser(collect)
    .default([]);
```

各コマンドでは `opt.project()` を呼ぶだけで、description やフラグ名が統一されます。

もうひとつ。`BACKLOG_PROJECT` 環境変数のフォールバックを `resolveProject` 関数で手書きしていたのですが、冗長さを感じて聞いてみたところ、Commander.js の `.env()` メソッドの存在を教えてくれました。

```ts
// Before: ヘルパー関数でフォールバック
const project = resolveProject(args.project); // process.env.BACKLOG_PROJECT ?? args.project

// After: Commander.js の .env() で宣言的に解決
new RequiredOption("-p, --project <id>", "...").env("BACKLOG_PROJECT");
```

Claude Opus でも、知識は豊富に持っているのに聞かれるまで出してこないことが多いです。
人間の「勘」や「冗長さへの違和感」が、AI の知識を引き出すトリガーになります。
引き出した知見を `AGENTS.md` にフィードバックすれば、以降の実装にも反映されていきます。

# 振り返り

テストが充実していれば、大規模なリプレースも AI に任せられます。
実際、コマンドパーサーを [citty](https://github.com/unjs/citty) から [Commander.js](https://github.com/tj/commander.js) へほぼ完成後に移行する必要が出ました（citty ではサブコマンドのヘルプ表示やオプションの柔軟性に限界があったため）。
Opus に指示したところ、テストを頼りに段階的なリファクタリングでやりきってくれました（[#53](https://github.com/nulab/bee/pull/53)）。

レールを敷く → AI が走る → 発見をレールに戻す。
このサイクルが bee を実装する上での肝でした。

Vibe Coding で作った元の backlog-cli がそうだったように、人間がガードレールを引かなければ AI の出力品質は安定しません。
AI をナビゲートする存在が、AI 協業の前提だと感じています。

# おわりに

`AGENTS.md` を整え、Linter でルールを敷き、レビューで軌道修正する。
振り返ると、自分がやっていたのはコードを書くことではなく、AI が迷わない環境を整えることでした。

やってみて気づいたのは、自分はコードを書くこと自体が好きだったのではなく、環境を作ってモノができあがっていくのが好きだったということです。
AI がその過程を加速してくれるなら、この先の開発はもっと面白くなると思っています。

bee は RC 版を公開しています。
試してみてもらえると嬉しいです。

```sh
npm i -g @nulab/bee@rc
```

https://github.com/nulab/bee

https://nulab.github.io/bee
