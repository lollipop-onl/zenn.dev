---
title: Nuxt (+TS) の開発サーバーがメモリ不足でクラッシュする事象を暫定的に回避する
emoji: 🍭
type: tech
topics: [nuxt, typescript, EveOneZenn]
published: true
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.12 です。

Nuxt の開発サーバーがメモリ不足で落ちた際の暫定的な対処方法を紹介します。
なお、ここで紹介する方法は事象を完全に回避するものではなく、事象の発生までの猶予を広げるものとなります。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-gha-push-diffs

# 発生環境

macOS のマシンで Nuxt を `@nuxt/typescript-build` と使用しているケースで発生した事象です。

* macOS Catalina (メモリ 16GB)
* Nuxt 2.14.4
* `@nuxt/typescript-build` 2.0.3

# 発生したエラー

Nuxt + `@nuxt/typescript-build` でローカルサーバー起動中、次のようなログとともにサーバーがクラッシュしました。

```sh
<--- Last few GCs --->

[32028:0x10291f000] 47294868 ms: Mark-sweep 2896.6 (2929.3) -> 2896.6 (2928.1) MB, 1305.4 / 0.0 ms  (average mu = 0.105, current mu = 0.114) last resort GC in old space requested
[32028:0x10291f000] 47295919 ms: Mark-sweep 2896.6 (2928.1) -> 2896.6 (2928.1) MB, 1050.3 / 0.0 ms  (average mu = 0.059, current mu = 0.000) last resort GC in old space requested


<--- JS stacktrace --->

==== JS stack trace =========================================

    0: ExitFrame [pc: 0x100950919]
    1: StubFrame [pc: 0x10092f465]
Security context: 0x124cd89008d1 <JSObject>
    2: exec [0x124cd89149b1](this=0x124cf1ed06e9 <JSRegExp <String[33]: ^******$>>,0x124cb6a0fbf1 <String[15]: /ja/payment/3ds>)
    3: 0x124c8d8c4c51 <Symbol: Symbol.match>(aka [Symbol.match]) [0x124cd8914899](this=0x124cf1ed06e9 <JSRegExp <String[33]: ^*****$>>,0x124cb6a0fbf...

FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
 1: 0x100080c68 node::Abort() [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
 2: 0x100080dec node::errors::TryCatchScope::~TryCatchScope() [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
 3: 0x100185167 v8::Utils::ReportOOMFailure(v8::internal::Isolate*, char const*, bool) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
 4: 0x100185103 v8::internal::V8::FatalProcessOutOfMemory(v8::internal::Isolate*, char const*, bool) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
 5: 0x10030b2f5 v8::internal::Heap::FatalProcessOutOfMemory(char const*) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
 6: 0x1003130ed v8::internal::Heap::AllocateRawWithRetryOrFail(int, v8::internal::AllocationType, v8::internal::AllocationOrigin, v8::internal::AllocationAlignment) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
 7: 0x1002dfa09 v8::internal::Factory::CodeBuilder::BuildInternal(bool) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
 8: 0x1002dffde v8::internal::Factory::CodeBuilder::Build() [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
 9: 0x10085a3f8 v8::internal::RegExpMacroAssemblerX64::GetCode(v8::internal::Handle<v8::internal::String>) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
10: 0x1005d5fa6 v8::internal::RegExpCompiler::Assemble(v8::internal::Isolate*, v8::internal::RegExpMacroAssembler*, v8::internal::RegExpNode*, int, v8::internal::Handle<v8::internal::String>) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
11: 0x1005ef8c2 v8::internal::RegExpImpl::Compile(v8::internal::Isolate*, v8::internal::Zone*, v8::internal::RegExpCompileData*, v8::base::Flags<v8::internal::JSRegExp::Flag, int>, v8::internal::Handle<v8::internal::String>, v8::internal::Handle<v8::internal::String>, bool) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
12: 0x1005eee01 v8::internal::RegExpImpl::CompileIrregexp(v8::internal::Isolate*, v8::internal::Handle<v8::internal::JSRegExp>, v8::internal::Handle<v8::internal::String>, bool) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
13: 0x1005eff6d v8::internal::RegExp::IrregexpPrepare(v8::internal::Isolate*, v8::internal::Handle<v8::internal::JSRegExp>, v8::internal::Handle<v8::internal::String>) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
14: 0x1005ee7d2 v8::internal::RegExpImpl::IrregexpExec(v8::internal::Isolate*, v8::internal::Handle<v8::internal::JSRegExp>, v8::internal::Handle<v8::internal::String>, int, v8::internal::Handle<v8::internal::RegExpMatchInfo>) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
15: 0x10063ead4 v8::internal::Runtime_RegExpExec(int, unsigned long*, v8::internal::Isolate*) [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
16: 0x100950919 Builtins_CEntry_Return1_DontSaveFPRegs_ArgvOnStack_NoBuiltinExit [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
17: 0x10092f465 Builtins_RegExpPrototypeExecSlow [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
18: 0x10092ba33 Builtins_RegExpPrototypeExec [/Users/USERNAME/.nodebrew/node/v12.16.1/bin/node]
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

このエラーログ（ `JavaScript heap out of memory` ）は実行中の Node.js プロセスがメモリ不足でクラッシュする際に出力されるものです。

# 対応

## Node.js プロセスのメモリ上限を上げる

まずは、Node.js のプロセスで使用できるメモリ上限を上げましょう。
Node.js のプロセスが使用できるメモリ上限は、デフォルトで 2GB 未満です。

`--max-old-space-size` オプションで任意のメモリ上限をメガバイト単位で指定します。

https://nodejs.org/api/cli.html#cli_max_old_space_size_size_in_megabytes

ここでは、8GB (8192MB) をメモリ上限として設定します。

```sh
$ NODE_OPTIONS="--max-old-space-size=8192" nuxt
```

## ForkTsCheckerWebpackPlugin のメモリ上限を上げる

Node.js 自体のメモリに併せて、 ForkTsCheckerWebpackPlugin のメモリ上限を上げましょう。
`nuxt.config.ts` へ次のようなオプションを追加します。

```ts:nuxt.config.ts
import { NuxtConfig } from '@nuxt/types';

const config: NuxtConfig = {
  buildModules: ['@nuxt/typescript-build'],
  typescript: {
    typeCheck: {
      typescript: {
        memoryLimit: 8192,
      },
    },
  },
};

export default config;
```

https://github.com/TypeStrong/fork-ts-checker-webpack-plugin#typescript-options

以上でメモリ上限までの猶予が大きくなり、メモリ不足でのクラッシュが発生しづらくなります。

# 参考

* [Command-line options | Node.js v15.3.0 Documentation](https://nodejs.org/api/cli.html)
* [TypeStrong/fork-ts-checker-webpack-plugin: Webpack plugin that runs typescript type checker on a separate process.](https://github.com/TypeStrong/fork-ts-checker-webpack-plugin)
