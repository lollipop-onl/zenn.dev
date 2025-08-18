---
title: CSS で Tailwind CSS の bg-black/75 みたいなことをする
emoji: 🍬
type: tech
topics: [css]
published: true
---

:::message
この記事で紹介する `rgb()` の相対値構文は Baseline 2024 であり、比較的新しいブラウザでのみ動作します。
https://caniuse.com/css-relative-colors
:::


Tailwind CSS では `bg-black/75` のように `カラーパレット/透明度` という表現が可能です。

https://tailwindcss.com/docs/colors#adjusting-opacity

これと似たようなことを CSS でやろうとするとき、私の場合は `0, 0, 0` のような RGB の値部分を CSS カスタムプロパティとして定義して `rgb()` で使うか `rgba()` で使うかを利用時に選択することが多いです。

```css
--color-black: 0, 0, 0;

background-color: rgb(var(--color-black));
background-color: rgba(var(--color-black), 0.75);
```

これでも良いのですが、透明度を指定したい箇所が限定的なときに都度 `rgb()` を書くのは煩雑だと感じます。  
また、定義するときに HEX カラーコードを利用できないのも個人的は改善したいです。

そこで利用できるのが 相対値構文 (Relative value syntax) です。

## 相対値構文

相対値構文は `rgb()` で利用できる、基準となる色から R, G, B を取り出し、別の色にできる構文です。

https://developer.mozilla.org/ja/docs/Web/CSS/color_value/rgb

具体的には次のように `from` キーワードと併せて利用できます。

```css
rgb(from green r g b);
```

この機能と `rgb(r g b / A)` のアルファチャンネル値の指定を併用することで、 Tailwind CSS の `bg-black/75` のような表現が可能になります。

## コード例

カラーパレットを CSS カスタムプロパティとして定義しつつ、透明度を指定したい箇所では `rgb()` の相対値構文を利用します。

```css
--color-black: #000;

background-color: var(--color-black);
background-color: rgb(from var(--color-black) r g b / 0.75);
```

また、 Chrome 139 からでのみ利用できる CSS カスタム関数を併用することで、よりシンプルな記述が可能です。

https://developer.chrome.com/release-notes/139

https://drafts.csswg.org/css-mixins-1/

```css
@function --color-alpha(--color, --alpha) {
  result: rgb(from var(--color) r g b / var(--alpha));
}

background-color: --color-alpha(var(--color-black), 0.75);
```

@[jsfiddle](https://jsfiddle.net/38Ldsakn/2/embedded/css,html,result/)
