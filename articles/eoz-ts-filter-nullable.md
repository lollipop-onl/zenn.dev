---
title: 表示しているページのタイトルをコピーするブックマークレット
emoji: 🍭
type: tech
topics: [bookmarklet, EveOneZenn]
published: true
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) の８日目です。

表示しているページのタイトルをコピーするブックマークレットを紹介します。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-judge-js-invalid-date

# 表示しているページのタイトルをコピーするブックマークレット

下記のブックマークレットをブックマークの URL に入力して保存してください。

ソースコード：

```js
(() => {
  // タイトルを参照
  const title = document.title;

  // コピーの元にするtextareaを作成
  const textarea = document.createElement('textarea');
  textarea.textContent = title;

  // bodyタグの最後にtextareaを追加
  document.body.appendChild(textarea);

  // textareaの内容をクリップボードにコピー
  textarea.select();
  document.execCommand('copy');

  // 最初に作成したtextareaをbodyから削除
  document.body.removeChild(textarea);

  console.log(`Copyed this website title "${title}"`);
})();
```

ブックマークレット：

```js
javascript:(()=>{const e=document.title,t=document.createElement("textarea");t.textContent=e,document.body.appendChild(t),t.select(),document.execCommand("copy"),document.body.removeChild(t),console.log(`Copyed this website title "${e}"`)})();
```

# 表示しているページのタイトルとリンクを Markdown でコピーするブックマークレット

ブックマークレット：

```js
javascript:(()=>{const e=`[${document.title}](${location.href})`,t=document.createElement("textarea");t.textContent=e,document.body.appendChild(t),t.select(),document.execCommand("copy"),document.body.removeChild(t),console.log(`Copyed this website title and URL: ${e}`)})();
```

