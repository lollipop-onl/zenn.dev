---
title: JavaScript で「新しいタブでリンクを開く」を実装する
emoji: 🍭
type: tech
topics: [javascript, EveOneZenn]
published: true
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.16 です。

フロントエンドの開発でたまに必要になる JavaScript からリンクを新しいタブで開く実装を紹介します。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-devtools-rendering-panel

# ソースコード

早速ですが、リンク先を新しいタブで開くコードを紹介します。

```ts
export const openLinkInNewTab = (href: string) => {
  const $a = document.createElement('a');

  $a.setAttribute('href', href);
  $a.setAttribute('target', '_blank');
  $a.setAttribute('rel', 'noopener noreferrer');

  document.body.appendChild($a);

  $a.click();

  document.body.removeChild($a);
};
```

処理の流れとしては次のとおりです。

* `a` タグを生成
* `href`・`target`・`rel` の各属性を設定
* `a` タグが不可視となるようスタイルを設定
* `a` タグを `body` タグの末尾に追加
* `a` タグをクリックさせる
* `a` タグを `body` 要素から削除

# 使い方

前項で定義した関数の使い方は、 `openLinkInNewTab` へリンクを渡すだけです。

```ts
openLinkInNewTab('https://zenn.dev/lollipop_onl');
```
