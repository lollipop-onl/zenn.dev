---
title: Sapper の goto() で遷移したときにフォーカスを維持する
emoji: 🍬
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [svelte, sapper]
published: true
---

Svelte のフレームワーク、Sapper には Svelte にないルーティングに関する一式の機能が含まれています。

そのなかに `goto` というプログラマブルに URL を変更するメソッドがあります。

```ts
import { goto } from '@sapper/app';

// Sapperで定義されたルートに遷移
await goto('/blog/1234');

// Sapper以外のURLに遷移
await goto('/static/images/photo.jpg');

// 外部サイトのURLに遷移
await goto('https://example.com/externals');
```

この `goto` メソッド、実行時に現在のフォーカスを外す処理が実行されます。

https://github.com/sveltejs/sapper/blob/5757a85d195f9ceb2e329caa5e3e38abd5a1123d/runtime/src/app/router/index.ts#L216

これは、アクセシビリティの観点から追加された処理のようです。

https://github.com/sveltejs/sapper/issues/287

ただ、たとえば検索フィールドの入力内容をクエリに連携するみたいなケースではやや厄介な実装です。

そこで、上記の Issue ではフォーカスを維持したいフィールドの onBlur ハンドラを無効化するアイデアが提案されています。

```svelte
<input bind:this={input} />

<script>
let input

$: noBlur(input)

function noBlur(input) {
  input.originalBlur = input.blur
  input.blur = () => {}
}

// input からのフォーカスを外す
input.originalBlur();
</script>
```

かなりハック的なアイデアですが、共通処理として実装されている以上、この回避策は致し方ないかなと思います。

良い Sapper Live を！
