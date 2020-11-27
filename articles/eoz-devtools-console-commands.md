---
title: Chrome DevTools Console で使える便利なコマンド
emoji: 🍭
type: tech
topics: [devtools, EveOneZenn]
published: true
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) の３日目です。

今回は Chrome DevTools の Console で使用できるコマンド（ Utilities API ）についてまとめます。
なお、紹介するコマンドはすべて、 Console パネルから実行した場合のみ使用できます。

https://developers.google.com/web/tools/chrome-devtools/console/utilities?hl=ja

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-nuxt-with-ts4-1

# 参照系

## $_

![](https://storage.googleapis.com/zenn-user-upload/xbwjehj6ua20x85u6e1wztqrrn5n)

`$_` は Console パネル上で直近に実行された処理の結果を参照します。

これを活用すると、`Promise` を返した関数を `await` させることができます。

![](https://storage.googleapis.com/zenn-user-upload/xrwignty3r9hvd1onaenicen5eya)

## $0 - $4

`$0` は Elements パネル上でアクティブな要素を参照します。

![](https://storage.googleapis.com/zenn-user-upload/ny7vgbnhutnxja4s2l0mevqu82qe)

Elements パネル上で `== $0` と表示されている要素が `$0` で参照できる要素です。

![](https://storage.googleapis.com/zenn-user-upload/ete6d2wre98k43x59h4bfmqofas4)

また、`$1`・`$2`・`$3`・`$4` で直近で選択していた要素を参照できます。

![](https://storage.googleapis.com/zenn-user-upload/3lm0nslpdf096c9fcffsuhw8q2jm)

## $(selector, [startNode])

![](https://storage.googleapis.com/zenn-user-upload/i8mg53psji3wzkfej9yip7kztm0i)

`document.querySelector` ライクなメソッドです。

第２引数に検索対象の親要素を指定すると、その要素の子孫要素からセレクタにマッチする要素が返されます。

## $$(selector, [startNode])

![](https://storage.googleapis.com/zenn-user-upload/9zzg1ri2udlf3any78nrku752jc2)

`document.querySelectorAll` ライクなメソッドです。

第２引数に検索対象の親要素を指定すると、その要素の子孫要素からセレクタにマッチする要素が返されます。

## $x(path, [startNode])

![](https://storage.googleapis.com/zenn-user-upload/99le2onfwaq1ju9wjucrkzkqa7rh)

XPath にマッチする要素を参照します。

:::message
**XPathについて**： [JavaScript で XPath を使用する - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Introduction_to_using_XPath_in_JavaScript)
:::

## getEventListeners(object)

![](https://storage.googleapis.com/zenn-user-upload/2elxlm51aimnbqra8e9y1ylyvwlm)

指定したオブジェクトに登録されているイベントリスナの一覧を参照します。

`$0` と組み合わせれば、選択している要素に登録されているイベントリスナの一覧を参照できます。

![](https://storage.googleapis.com/zenn-user-upload/ox3tt7sobhf132ax1af33m0jg44k)

## inspect(object/function)

引数に指定した Object/Function を Elements パネルまたは Source パネルで開きます。

たとえば、 `inspect($( selector ))` を実行すると、セレクタにマッチする要素を Elements パネルで開きます。

## queryObjects(Constructor)

![](https://storage.googleapis.com/zenn-user-upload/0au2ierk5wfllj6pvocjpcye83nx)

引数に指定したコンストラクタのインスタンスが配列で返されます。

たとえば、 `queryObjects(Promise)` ですべての Promise を、 `queryObjects(HTMLElement)` ですべての HTML要素 を参照できます。

# 監視・デバッグ系

## copy(object)

![](https://storage.googleapis.com/zenn-user-upload/i3l3nro1j2vkbhenrl91hi7b1hl2)

指定したオブジェクトを文字列化してクリップボードへコピーします。

このとき、HTML要素 はそのまま文字列化されますが、オブジェクトは可読性の高い形式でコピーされます。

## debug(function) / undebug(function)

指定した関数が実行される直前で処理を中断し Source パネルでコードをステップ実行できます。

デバッグを停止するには `undebug(function)` を使用するか、UIからすべてのブレークポイントを無効にします。

## monitor(function) / unmonitor(function)

![](https://storage.googleapis.com/zenn-user-upload/lnr9rkkvqmsubsvua9yaofr0650j)

指定した関数が実行されるたびに、関数名と指定された引数を Console パネルに表示します。

監視を停止するには `unmonitor(function)` を使用します。

## monitorEvents(object[, events]) / unmonitorEvents(object[, events])

![](https://storage.googleapis.com/zenn-user-upload/gqnr8k1lcgbybdu2f4k516az4ktd)

指定したオブジェクトに対してイベントがトリガーされるたびに、イベント名とイベント内容を Console パネルに表示します。

イベントには `mouse`・`key`・`touch`・`control` がプリセットとして用意されており、それぞれ次のイベントを監視します。

|プリセットイベント|監視されるイベント|
|:--|:--|
|mouse|`mousedown`, `mouseup`, `click`, `dblclick`, `mousemove`, `mouseover`, `mouseout`, `mousewheel`|
|key|`keydown`, `keyup`, `keypress`, `textInput`|
|touch|`touchstart`, `touchmove`, `touchend`, `touchcancel`|
|control|`resize`, `scroll`, `zoom`, `focus`, `blur`, `select`, `change`, `submit`, `reset`|

監視を終了するには `unmonitorEvents(object[, events])` を使用します。

## profile([name]) / profileEnd([name])

![](https://storage.googleapis.com/zenn-user-upload/1aflttfv17gpuq1sbkumr9162vrd)

JavaScript の CPUプロファイリング を記録します。
`profile([name])` でプロファイリングセッションを開始し、 `profileEnd([name])` で終了します。

プロファイリングセッションはネストして実行することもできます。

記録した CPUプロファイリング は JavaScript Profiler パネルで確認できます。

![](https://storage.googleapis.com/zenn-user-upload/1xcqc4wc9rkrzk5c3cdh5p35c397)

メニューから More tools → JavaScript Profiler と選択してパネルを表示します。

![](https://storage.googleapis.com/zenn-user-upload/v7my5n3b2f0d8o9e6qcqniduc84f)

# エイリアス系

## clear()

![](https://storage.googleapis.com/zenn-user-upload/avolcyn0q3iow49tuajtejqhkngy)

Console パネルの履歴をクリアします。
`console.clear()` メソッドと同等の機能です。

## dir(object)

![](https://storage.googleapis.com/zenn-user-upload/saydv9xpfsccexgvkj1onkxnodde)

指定したオブジェクトを JSON表現 で表示します。
`console.dir()` メソッドと同等の機能です。

## dirxml(object)

![](https://storage.googleapis.com/zenn-user-upload/xr1wwdlserbzw1m1vpu5gxylwla7)

指定したオブジェクトを XML表現 で表示します。
`console.dirxml()` メソッドと同等の機能です。

## keys(object) / values(object)

![](https://storage.googleapis.com/zenn-user-upload/zyvhi7evt2hn9qkryml3wrd8e9io)

指定したオブジェクトのキーおよび値の配列を取得します。
`Object.keys()` メソッド・ `Object.values()` メソッドと同等の機能です。

## table(data[, columns])

![](https://storage.googleapis.com/zenn-user-upload/syx1lq1sk0178nh1ea1drign06cb)

データを表の形式で表示します。
`console.table()` メソッドとほぼ同等の機能ですが、表示する列を指定できます。

# おまけ

## 出力された値を使用する

![](https://storage.googleapis.com/zenn-user-upload/eyp8sb7ath8ywpitfz7z8c17ik7t)

Console パネルに出力されたデータを 右クリック → `Store as global variable` とすると、 `temp[N]` というグローバル変数にデータが代入されます。

実際のコードから出力されたデータを加工したり転用したりするときに便利です。

# 参考

* [Console Utilities API Reference  |  Chrome DevTools  |  Google Developers](https://developers.google.com/web/tools/chrome-devtools/console/utilities?hl=ja)
