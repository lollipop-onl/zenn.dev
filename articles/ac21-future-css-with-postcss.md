---
title: PostCSS で先取りする、未来の CSS 7選
emoji: 🍭
type: tech
topics: [css, postcss]
published: true
---

# はじめに

この記事は [個人的なアドベントカレンダー2021](https://zenn.dev/lollipop_onl/scraps/691bacd85c4c80) の1日目です。

**次回：** 未定

# postcss-preset-env

PostCSS のプラグイン `postcss-preset-env` は、未来の機能候補として実際に議論されている記法や機能を Polyfill するプラグインをまとめたものです。

https://preset-env.cssdb.org/

`postcss-preset-env` では、 custom property (`var(--some-value)`) や nesting rules (ネストしたセレクタ) といった Sass などのプリプロセッサでお馴染みの機能から、ニッチだが便利な機能まで、さまざまな策定中の機能を取り入れることができます。

また、いずれも策定中の機能であるため、将来、CSSの機能として組み込まれる可能性がある機能も含まれており、より標準に近い形で新機能を使えます。  
（策定中のため、書き方が変わったり、機能そのものがちょっとずつ変わることはありますが）

この記事では、 `postcss-preset-env` の提供する Polyfill の中から書き方を改善する系のよく使うものを5つピックアップして紹介します。

なお、 custom property については [ブラウザ対応が十分に行われている](https://caniuse.com/mdn-css_properties_custom-property_var) うえに、 Polyfill されるとメディアクエリなどで変更できなくなるため、最近では有効にしないことが多いです。

## [nesting rules](https://preset-env.cssdb.org/features#nesting-rules)

```css
.article-card {
  & > .title {
    font-size: 1.8rem;
  }

  &.-inactive {
    opacity: 0.5;
  }

  & a:hover {
    text-decoration: none;
  }
}
```

お馴染みのセレクタをネストできるようにする機能です。
Sass などと異なる点として、ネストしたセレクタは必ず `&` から始めなければなりません。

```css
.article-card {
  /* OK !! */
  & p {}
  
  /* NG... */
  p {}
}
```

## [custom media queries](https://preset-env.cssdb.org/features#custom-media-queries)

```css
@custom-media --viewport-mobile (max-width: 600px);
@custom-media --viewport-desktop screen and (min-width: 601px);

@media (--viewport-mobile) {
  /* モバイル幅でのみ適用したいスタイル */
}

@media (--viewport-desktop) {
  /* ディスプレイのデスクトップ幅でのみ適用したいスタイル */
}
```

custom property のように、メディアクエリを変数化できる機能です。
`@media (--viewport-mobile) and (--viewport-desktop) {}` のように `and` でつなげることもできます。

## [media query range](https://preset-env.cssdb.org/features#media-query-ranges)

```css
@custom-media --sample1 (width >= 200px);
@custom-media --sample2 (100px >= height >= 500px);
```

範囲を指定するメディアクエリを不等号で書けるようにする機能です。
`min-` / `max-` より直感的に書けるので、個人的に早く来てほしい機能です。

指定できるメディア特性は [プラグインの Media feature names](https://github.com/postcss/postcss-media-minmax#media-feature-names) に記載があります。
このうち、`device-` 系のメディア特性はすでに廃止されているため、使うとしても `width`・`height`・`aspect-ratio` くらいかなと思われます。

## [alpha hex colors](https://preset-env.cssdb.org/features#media-query-ranges)

```css
section {
  background-color: #f3f3f3f3;
  color: #0003;
}
```

Sass でサポートされている、カラーコード＋透明度の記法です。
透明度を指定するためにわざわざ `rgba()` に置き換える必要がなくなるため、意外と使い所が多いです。

## [:matches pseudo-class](https://preset-env.cssdb.org/features#matches-pseudo-class)

```css
.app-button:matches(:disabled, .-disabled) {
  opacity: 0.6;
}
```

ひとつのセレクタに対して複数のセレクタを充てることができる機能です。
サンプルコードのように、ボタンの非活性を `disabled` 属性と `.-disabled` クラスの両方で表現することがあるようなときに便利です。

## [:not pseudo-class](https://preset-env.cssdb.org/features#not-pseudo-class)

```css
.list-item:not(:first-child, .-modify) {
  border-bottom: 1px solid grey;
}
```

`:not()` 擬似クラスを一度に指定できる機能です。

## [overflow shorthand property](https://preset-env.cssdb.org/features#overflow-property)

```css
section {
  overflow: auto hidden;
}
```

`overflow` プロパティで `overflow-x` と `overflow-y` のそれぞれのプロパティを指定できるようにする機能です。
