---
title: Chrome DevTools の Sensors タブを使ってみよう
emoji: 🍭
type: tech
topics: [devtools, EveOneZenn]
published: false
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) vol.17 です。

Chrome DevTools に組み込まれている、センサー関連の挙動をオーバーライドできる Sensors タブについてまとめます。

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-js-open-new-tab

# Sensors タブを表示する

Chrome DevTools の Sensors タブは メニュー（三点ドット）→ More tools → Sensors から表示できます。

![](https://storage.googleapis.com/zenn-user-upload/qrdmw917ebj046z5atyh33p2benf)

Elements や Console といったメインのパネルが並んでいる上部のナビゲーションではなく、下部のナビゲーションで Sensors パネルが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/liwc1jbbppn6ven3dxn2ooss23ia)

# Sensors タブの機能

Sensors タブで使用できる各種機能を紹介します。

## Location

ブラウザがアプリケーションに提供する位置情報やタイムゾーン、言語をオーバーライドします。

![](https://storage.googleapis.com/zenn-user-upload/t3bkdc4c58cjh2cda8czyk9kmokl)

ドロップダウンから登録されている地点の情報を使用するか、手動で情報を入力するかを選択できます。
また、 Location unavailable を選択すれば位置情報が取得できなかった状況を再現できます。

![](https://storage.googleapis.com/zenn-user-upload/gf0zdrgpjt4hqpyk3h7nkrm63i3b)

地点の登録・編集は Manage ボタンからアクセスできる設定で行います。
デバッグで頻繁に使用する地点があれば登録すると良いでしょう。

![](https://storage.googleapis.com/zenn-user-upload/byay2q3o06ha5hr2wf9op2mg8uuc)

## Orientation

ブラウザがアプリケーションに提供するデバイスの向きをオーバーライドします。

![](https://storage.googleapis.com/zenn-user-upload/40w5xpwh9t3tk0p8hu8ckj9iqj1e)

ドロップダウンから基本的な方向を選択できるほか、 Custom orientation では数値と右のモデルを実際に回して方向を設定できます。

![](https://storage.googleapis.com/zenn-user-upload/un6k1ytoopijoku2pu6tevhhbef4)

## Touch

アプリケーション上で検知するクリックイベントを強制的にタッチイベントに変更するか設定します。

Device-based は現在のデバイスの設定を使用しますが、 Force enabled は常にタッチイベントが発火するようになります。

![](https://storage.googleapis.com/zenn-user-upload/jhvpjov15belgdk4wvomth3jcb0m)

## Emulate Idle Detector state

ユーザーのアクティブ状態を検知する Idle Detection API の受け取る状態をオーバーライドします。

![](https://storage.googleapis.com/zenn-user-upload/jhvpjov15belgdk4wvomth3jcb0m)

ドロップダウンから「ユーザーがアクティブか」と「スクリーンがロックされているか」を設定できます。

![](https://storage.googleapis.com/zenn-user-upload/ejbsm6yhjxzlaiswnpumofnoek1y)

なお、 Idle Detection API は Google Chrome ではフラグを有効にしないと使用できません。

https://web.dev/idle-detection/

# 関連記事

https://zenn.dev/lollipop_onl/articles/eoz-devtools-console-commands
https://zenn.dev/lollipop_onl/articles/eoz-devtools-rendering-panel
