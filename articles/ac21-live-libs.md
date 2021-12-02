---
title: ライブラリはドキュメントサイトで挙動を確認できる
emoji: 🍭
type: tech
topics: [javascript]
published: true
---

# はじめに

この記事は [個人的なアドベントカレンダー2021](https://zenn.dev/lollipop_onl/scraps/691bacd85c4c80) の2日目です。

**前回：**
https://zenn.dev/lollipop_onl/articles/ac21-future-css-with-postcss

**次回：** 未定

# ライブラリを試したい

ライブラリを使う際、「こういうコードのときってどういう挙動になるんだろう？」というちょっとした疑問が浮かぶことがあるかと思います。

そんなときはそのライブラリのドキュメントサイトのConsoleでライブラリの挙動を試してみましょう！

今回はドキュメントサイトで挙動が確認できるライブラリをいくつかご紹介します。
なお、ここで紹介したライブラリ以外でも、意外とグローバルにライブラリ本体を登録してくれているドキュメントがあるので、試してみても良いかもしれません。

## Lodash

https://lodash.com/

![](https://storage.googleapis.com/zenn-user-upload/d5b74ead3fd2-20211202.png)

グローバルに `_` として Lodash 本体が登録されています。

## Day.js

https://day.js.org/

![](https://storage.googleapis.com/zenn-user-upload/456791b02614-20211202.png)

グローバルに `dayjs` として Day.js 本体とすべてのオフィシャルプラグインが登録されています。
Consoleに `Feel free to try sample codes here 😃` と記載してくれているのがありがたいですね！

## BigNumber.js

https://mikemcl.github.io/bignumber.js/

![](https://storage.googleapis.com/zenn-user-upload/de25853f5331-20211202.png)

グローバルに `BigNumber` クラスが登録されています。

# おわりに

ライブラリのドキュメントサイトでそのライブラリを試せることがあるよ、という記事でした。

もちろん、すべてのライブラリではないと思いますが、検証のために `yarn install` したり CDN からロードしたり... という手間がないので知っておくと便利かなと思います！
