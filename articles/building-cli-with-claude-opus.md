---
title: "Claude Code と AGENTS.md で Backlog CLI を実装しきった話"
emoji: "🐝"
type: "tech"
topics: ["claude", "cli", "typescript", "ai"]
published: false
---

Backlog の CLI ツール bee を Claude Opus と一緒に作りました。

```sh
npm i -g @nulab/bee@rc
```

https://github.com/nulab/bee

自分が書いたコードはごくわずかで、役割は「コードを書くこと」よりも「AI が迷わない環境を作ること」に寄っていました。

この記事では、AI にコードを任せる上で実際に効いたテクニックと、やってみて気づいたことを書きます。

# 前提

bee は GitHub CLI のように Backlog をターミナルから操作する CLI です。

```sh
bee issue list --assignee @me --status 処理中
bee issue view PROJECT-123 --web
```

以前 Vibe Coding で個人的に作った [@simochee/backlog-cli](https://github.com/simochee/backlog-cli) がベースになっていますが、実装漏れや不具合が多く実用には心許ない品質でした。
そこで「Backlog API → CLI コマンド」というパターンが明確なこの題材を、AI 向けの体裁を整えた上で作り直すことにしました。

コードのほとんどを Claude Opus 4.6 が書き、自分は「選定」「統一感」「レビュー」「レール敷き」を担当するという体制です。

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
約 400 行になりましたが、この規模なら「ドキュメントとして妥当」な文量に収まっています。

## Linter で選択肢を潰す

AI は「`interface` と `type` のどちらを使うか」「default export にするか」といった、どちらでも動くけど統一感に影響する選択を頻繁にします。
表現がブレるとレビューで見るべき箇所が増え、地味にストレスです。

[oxlint](https://oxc.rs/) でこれらの選択肢を潰しておけば、レビューで指摘する必要がなくなります。
自分が実際に設定したルールはこのあたりです。
https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-type-definitions
https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-type-imports
https://oxc.rs/docs/guide/usage/linter/rules/import/no-default-export
https://oxc.rs/docs/guide/usage/linter/rules/typescript/array-type
https://oxc.rs/docs/guide/usage/linter/rules/typescript/consistent-indexed-object-style

ポイントは、これらが「正解がない」スタイルの問題だということです。
人間同士でもブレる箇所なので、AI に任せるなら Linter で機械的に強制するのが確実です。

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
AI はロジックやドメインを混在させがちなので、パッケージ境界で強制的に分けることでレビューの指摘が減りました。

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

ドキュメントサイト側では、ビルド時に [Jiti](https://github.com/nicolo-ribaudo/jiti) で CLI のコマンドファイルを直接 import し、Commander.js インスタンスからメタデータを抽出しています。

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
規約と構造が整っていれば、AI は並列に走れます。
コマンドグループを一気に実装できたのは、この並列開発の成果です。

## difit によるローカルレビューループ

difit はローカルで差分をレビューできるツールです。
https://github.com/yoshiko-pg/difit

Claude Code で `/difit` スキルを使うと、現在の差分の適切な範囲でレビュー画面を開いてくれます。

レビュー画面を閉じると、指摘内容が自動的にエージェントに渡り、修正まで実行してくれます。
この「レビュー → 自動修正」のループが快適で、レビューの心理的ハードルが下がりました。
push 前にチェックでき、任意の差分範囲のコードも確認できるので、worktree 並列開発との相性も良いです。

## AI の「引き出し」を開ける

開発中に「もっとスマートなやり方がないかな」と感じたとき、その疑問をそのまま聞いてみます。

たとえば、各コマンドで `--project` や `--assignee` の説明文がバラバラに書かれていました。
「共通化できないか」と聞いたところ、共通オプションを関数として切り出す構成を提案してくれました。

```ts
// apps/cli/src/lib/common-options.ts
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

Claude Opus は知識を豊富に持っているのに、聞かれるまで出してこないことが多いです。
人間の「勘」や「冗長さへの違和感」が、AI の知識を引き出すトリガーになります。
引き出した知見を `AGENTS.md` にフィードバックすれば、以降の実装にも反映されていきます。

# 振り返り

## 人間がレビューで指摘していたこと

リファクタリングのコミットログを振り返ると、人間の指摘は大きく 3 つに分類できました。

### バラバラなものを統一して

最も多かったです。
メッセージフォーマットの標準化、共通引数定義の集約、テストヘルパーの統一など。
AI は個々のコマンドを正しく実装できるのですが、コマンド間の統一感は人間が見ないとブレます。

### 置き場所が違う

CLI に直書きされたユーティリティを packages に移動する、apps と packages の区別を正す、といったものです。
AI は「動くコード」は書けるのですが、アーキテクチャ的にどこに置くべきかの判断はたまに弱いことがあります。
過信して放置すると、あとから大規模な整理が必要になるので要注意です。

### もっとスマートなやり方がある

前述の共通オプションの切り出しや、環境変数フォールバックの宣言的な解決など。
人間が「冗長さ」を感じて AI に聞くことで、より良い解決策が出てきます。

## やってみて思ったこと

設計や実装そのものは Opus に任せられました。
人間が注力すべきだったのは「選定」と「統一感」です。

テストが充実していれば大規模なリプレースも AI に任せられます。
実際、コマンドパーサーを [citty](https://github.com/unjs/citty) から [Commander.js](https://github.com/tj/commander.js) へほぼ完成後に移行する必要が出たのですが、Opus に指示したところ段階的なリファクタリングでやりきってくれました。
https://github.com/nulab/bee/pull/53

レールを敷く → AI が走る → 発見をレールに戻す、というサイクルが bee を実装する上での肝でした。

Vibe Coding で作った元の backlog-cli がそうだったように、人間がガードレールを引かなければ AI の出力品質は安定しません。
AI をナビゲートする存在が、AI 協業の前提だと感じました。

# おわりに

AI との協業で求められるのは「コードを書く力」よりも「AI が迷わない環境を作る力」でした。
`AGENTS.md` の整備、Linter による選択肢の制限、パッケージ分割による責務の明確化。
地味な作業ですが、積み重ねるほど AI の出力品質が上がっていきました。

bee は RC 版を公開しています。
試してみてもらえると嬉しいです。

```sh
npm i -g @nulab/bee@rc
```

https://github.com/nulab/bee

https://nulab.github.io/bee
