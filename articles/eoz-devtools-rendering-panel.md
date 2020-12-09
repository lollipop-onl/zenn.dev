---
title: Chrome DevTools の Rendering タブを使ってみよう
emoji: 🍭
type: tech
topics: [devtools, EveOneZenn]
published: false
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.15 です。

Chrome DevTools に組み込まれている Rendering タブについてまとめます。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-vercel-pricing-2020

# Rendering タブとは

# Rendering タブを表示する

Chrome DevTools の Rendering タブは メニュー（三点ドット）→ More tools → Rendering から表示できます。

![](https://storage.googleapis.com/zenn-user-upload/3vezew6rohwhxtnuk1ktd20hzr2m)

Elements や Console といったメインのパネルが並んでいる上部のナビゲーションではなく、下部のナビゲーションで Rendering パネルが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/flc3efi3ci9nrj8u3gzll26vhgfq)

# Rendering タブの機能

Rendering タブで使用できる各種機能を紹介します。

## Paint flashing

ページ上で再描画が発生したエリアを緑色でハイライトします。
DOM の変化や CSS プロパティ、アニメーションの変化を可視化できます。

[![Image from Gyazo](https://i.gyazo.com/322d28029a81680da43525459ed8d470.gif)](https://gyazo.com/322d28029a81680da43525459ed8d470)

## Layout Shift Regions

レイアウトがシフトしたエリアを青色でハイライトします。
CSS の `position` プロパティの変化や画像読み込み時のガタつきを可視化できます。

[![Image from Gyazo](https://i.gyazo.com/4bea38066b84f7dc07801ed0ca97663b.gif)](https://gyazo.com/4bea38066b84f7dc07801ed0ca97663b)

## Layer borders

内部的なレイヤーとガイドをオレンジ、オリーブ、シアンでハイライトします。
ボーダーの色の意味は Chromium のソースコード [`debug_colors.cc`](https://source.chromium.org/chromium/chromium/src/+/master:cc/debug/debug_colors.cc) のコメントに記載されているとおりです。

![](https://storage.googleapis.com/zenn-user-upload/5shgkvwx17jxap18rgeqpxeh0bxn)

## Frame Rendering Stats

フレームごとの処理能力や間引かれたフレームの分布、 GPU メモリの使用率を表示します。

![](https://storage.googleapis.com/zenn-user-upload/cu3z3ddnxz2cb4yzfkvgjn3r1olq)

## Scrolling performance issues

タッチやホイール、その他のメインスレッドでの処理からスクロールを遅延させる可能性がある要素を青緑でハイライトします。

![](https://storage.googleapis.com/zenn-user-upload/tsb7096x631p3nfhppadj77193j5)

画像は [Performance Analysis Reference  |  Chrome DevTools  |  Google Developers](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference) より引用

## Highlight ad frames

広告と判定されるエリアを赤くハイライトします。

![](https://storage.googleapis.com/zenn-user-upload/qc0037rgpuyrkx6nb0hfyxkbj299)

## Hit-test borders

ヒットテスト領域の周辺のボーダーを表示します。

詳細不明...

## Disable local fonts

`@font-face` の `local()` を無効化します。
サーバーからフォントファイルをダウンロードさせたいときなどに有効にします。

チェックしたらページをリロードすることで反映されます。

## Emulate a focusing page

ページ内にフォーカスを残します。
`:focus` 擬似クラスのスタイルを維持したい場合などに有効にします。

次のアニメーションでは、フォーカスされた状態を維持して DevTools を操作できるようにするため、この機能を有効にしています。

[![Image from Gyazo](https://i.gyazo.com/2397e438af49776ea1eaa89fbbf2eb42.gif)](https://gyazo.com/2397e438af49776ea1eaa89fbbf2eb42)

## Emulate CSS media type

CSS のメディアタイプを `print`・`screen` に強制します。
プリント用のスタイルを指定しているページのデバッグなどで便利です。

## Emulate CSS media feature prefers-color-scheme

CSS のメディア特性 `prefers-color-scheme` を `light`・`dark` に強制します。
ライトモード・ダークモードに対応しているページのデバッグなどで便利です。

https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-color-scheme

## Emulate CSS media feature prefers-reduced-motion

CSS のメディア特性 `prefers-reduced-motion` を `reduce` に強制します。

https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-reduced-motion

## Emulate vision deficiencies

ページで視覚障害のエミュレーションを強制します。
次の5つの視覚障害での見え方をエミュレートできます。

### Blurred vision（霧視）

![](https://storage.googleapis.com/zenn-user-upload/ddi59kvxc58hj2fww15jck735u8c)

### Protanopia（赤色盲）

![](https://storage.googleapis.com/zenn-user-upload/m2hep1rand6nvepi8s5uxkaq79uh)

### Deuteranopia（緑色盲）

![](https://storage.googleapis.com/zenn-user-upload/yzan2yb7ryj3tqyvdga7kgal34gc)

### Tritanopia（青色盲）

![](https://storage.googleapis.com/zenn-user-upload/qjsdggn3sqpqwjvz0wj0xlaj9opc)

### Achromatopsia（全色盲）

![](https://storage.googleapis.com/zenn-user-upload/o6ih34m2plxwnie01ukklyia7ygy)

# 参考

* [Chrome DevTools layer borders color meaning - Stack Overflow](https://stackoverflow.com/questions/35301036/chrome-devtools-layer-borders-color-meaning)
* [Performance Analysis Reference  |  Chrome DevTools  |  Google Developers](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference)
* [prefers-color-scheme - CSS: カスケーディングスタイルシート | MDN](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-color-scheme)
* [prefers-reduced-motion - CSS: カスケーディングスタイルシート | MDN](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-reduced-motion)
