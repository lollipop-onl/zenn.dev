---
title: 短いクラス名で運用できる CSS設計 rscss を CSS Modules 向けにアレンジしてみた
emoji: 🍭
type: tech
topics: [css, rscss, EveOneZenn]
published: false
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) の９日目です。

CSS設計のひとつである rscss を CSS Modules (React向け)にアレンジしてみたので紹介します。

紹介する規約を使った場合、下記のようなクラス名で CSS Modules を運用できます。

```JSX
import React from 'react';
import cn from 'classnames';
import styles from 'styles.module.scss';

const Component = (props) => (
  <button
    className={cn(
      styles.searchButton,
      { [styles.Disabled]: props.disabled }
    )}
  >
    <span className={styles.icon}>🔎</span>
    <span className={styles.text}>Search</span>
  </button>
);
```

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-copy-url-bookmarklet

# rscss とは

https://rscss.io/

> Styling CSS without losing your sanity

rscss は BEM と似た `Components`・`Elements`・`Variants` をベースの構成とする非常にシンプルな CSS クラス設計の一つです。

以降、 rscss の紹介となります。
本題は [CSS Modules に rscss を導入する](#css-modules-%E3%81%A7-rscss-%E3%82%92%E5%B0%8E%E5%85%A5%E3%81%99%E3%82%8B) からはじまりますので、そこだけ読みたいという方はジャンプしてください。

## 特徴

* ネストは１段まで
* Components と Elements は必ず親子セレクタ (`>`) で指定する
* Components のクラス名は `-` を含む２単語以上で
  * 例： `.search-form`・`.account-profile-card`
  * [About components](https://rscss.io/components.html)
* Elements のクラス名は `-` を含まないなるべく１単語で
  * 例： `.title`・`.firstname`
  * [Elements](https://rscss.io/elements.html)
* Variants のクラス名は `-` からはじめる
  * 例： `.-primary`・`.-disabled`・`.-small`
  * [Variants](https://rscss.io/variants.html)
* Helpers のクラス名は `_` からはじめる
  * 例： `._center`・`._unmargin`・`._hidden-sm`
  * [Helpers](https://rscss.io/helpers.html)

[Layouts](https://rscss.io/layouts.html) という考えもありますが、ここでは説明を省きます。
## BEM との比較

rscss を構成する要素は BEM に非常に似ています。
ざっくりと言えば、「 Block と Element が親子であることを強制される BEM 」と表現できるかなと思います。

次のコードは、それぞれ rscss と BEM で同じ要素をマークアップしたものです。
２つの HTML・CSS から違いを見ていきます。

ここでは、次のような検索ボタン＋非活性スタイルをマークアップするとします。

![](https://storage.googleapis.com/zenn-user-upload/r9uu7tm3ypdimu9iu1yvqnuljx3h)

### BEM でマークアップ

[JSFiddle](https://jsfiddle.net/simochee/vm45g9eu/4/)

```HTML:html
<button class="search-button">
  <span class="search-button__icon">🔎</span>
  <span class="search-button__text">Search</span>
</button>

<button class="search-button search-button--disabled">
  <span class="search-button__icon">🔎</span>
  <span class="search-button__text">Search</span>
</button>
```

```SCSS:scss
.search-button {
  $self: &;
  
  display: flex;
  align-items: center;
  padding: 8px 16px;
  margin-bottom: 8px;
  font-size: 12px;
  border: none;
  background: #339194;
  border-radius: 8px;
  
  &__icon {
    margin-right: 8px;
  }
  
  &__text {
    font-size: 1.2em;
    color: #fff;
  }
  
  &--disabled {
    background: #aaa;
  }
  
  &--disabled > #{$self}__icon {
    opacity: 0.8;
  }
}
```

### rscss でマークアップ

[JSFiddle](https://jsfiddle.net/simochee/up40mojh/1/)

```HTML:html
<button class="search-button">
  <span class="icon">🔎</span>
  <span class="text">Search</span>
</button>

<button class="search-button -disabled">
  <span class="icon">🔎</span>
  <span class="text">Search</span>
</button>
```

```SCSS:scss
.search-button {
  display: flex;
  align-items: center;
  padding: 8px 16px;
  margin-bottom: 8px;
  font-size: 12px;
  border: none;
  background: #339194;
  border-radius: 8px;
  
  & > .icon {
    margin-right: 8px;
  }
  
  & > .text {
    font-size: 1.2em;
    color: #fff;
  }
  
  &.-disabled {
    background: #aaa;
  }
  
  &.-disabled > .icon {
    opacity: 0.8;
  }
}
```

### 比べてみる

まず、 HTML の記述量から BEM と比較しかなりクラス名を短くできているのがわかるかと思います。

```HTML:html
<!-- BEM -->
<button class="search-button">
  <span class="search-button__icon">🔎</span>
  <span class="search-button__text">Search</span>
</button>

<!-- rscss -->
<button class="search-button">
  <span class="icon">🔎</span>
  <span class="text">Search</span>
</button>
```

これは、 BEM がクラスの単一性を Block のクラス名をプレフィックスとすることで担保しているのに対し、 rscss は Components からの親子関係で担保していることに起因します。
Elements/Element の記述が BEM の `&__icon` に対し rscss では `& > .icon` となっています。

```SCSS:scss
/* BEM */
.search-block {
  &__icon {}
  &__text {}
}

/* rscss */
.search-block {
  & > .icon {}
  & > .text {}
}
```

また、 Variants/Modifier も BEM の `&--disabled` に対し rscss は `&.-disabled` となっています。

```SCSS:scss
/* BEM */
.search-block {
  &--disabled {}
}

/* rscss */
.search-block {
  &.-disabled {}
}
```

## Pros / Cons

* **Pros**
  * 短いクラス名で運用できる
  * Components からみて遠すぎる子孫が Elements として使用されるのを防止できる
* **Cons**
  * グローバルな CSS では運用が難しい
    * → 1コンポーネント1ファイルなどで運用すれば Components のクラス名が重複することはなくなるのでできないことはない
  * ここ数年、リポジトリが更新されていない

# CSS Modules に rscss を導入する

前項のとおり、 rscss は短いクラス名で BEM に匹敵する設計を実現できます。
特に、 CSS のスコープが小さい CSS Modules や Scoped CSS では rscss の特徴が活かされます。

しかし、 Vue の Scoped CSS はさておき、 CSS Modules で使用する際にはハイフンを使用する規約はかなりわずらわしいです。

次のコードは、ベーシックな rscss を採用した React コンポーネントの例です。
Components と Variants のクラス名を付与する際に `styles['search-button']` のような指定になってしまっています。

```JSX:jsx
import React from 'react';
import cn from 'classnames';
import styles from 'styles.module.scss';

const Component = (props) => (
  <button
    className={cn(
      styles['search-button'],
      { [styles['-disabled']]: props.disabled }
    )}
  >
    <span className={styles.icon}>🔎</span>
    <span className={styles.text}>Search</span>
  </button>
);
```

# CSS Modules 向け rscss

JavaScript でクラス名を参照する以上、できればハイフンを使用したくありません。
そこで、次のように rscss の規約をアップデートしてみます。

* Components のクラス名は **キャメルケースの2単語以上** で
  * 例： `.searchForm`・`.accountProfileCard`
* Elements のクラス名は大文字を含まない1単語で
  * 例： `.title`・`.firstname`
* Variants のクラス名は **パスカルケース** で
  * 例： `.Primary`・`.Disabled`・`.Small`
* Helpers のクラス名は `_` からはじめる
  * 例： `._center`・`._unmargin`・`._hidden-sm`

これで、検索ボタンをマークアップし直すと、次のようになります。

```SCSS:styles.module.scss
.searchButton {
  display: flex;
  align-items: center;
  padding: 8px 16px;
  margin-bottom: 8px;
  font-size: 12px;
  border: none;
  background: #339194;
  border-radius: 8px;
  
  & > .icon {
    margin-right: 8px;
  }
  
  & > .text {
    font-size: 1.2em;
    color: #fff;
  }
  
  &.Disabled {
    background: #aaa;
  }
  
  &.Disabled > .icon {
    opacity: 0.8;
  }
}
```

```JSX:jsx
import React from 'react';
import cn from 'classnames';
import styles from 'styles.module.scss';

const Component = (props) => (
  <button
    className={cn(
      styles.searchButton,
      { [styles.Disabled]: props.disabled }
    )}
  >
    <span className={styles.icon}>🔎</span>
    <span className={styles.text}>Search</span>
  </button>
);
```

すべてのクラスをプロパティとして指定できるようになり簡素に記述できるようになりました。

# おまけ： stylelint でクラス名をチェックする

rscss の規約は stylelint の `stylelint-rscss` でチェックできます。

https://github.com/rstacruz/stylelint-rscss

デフォルトの設定では、ベーシックな rscss の規約に違反していないかチェックしますが、それぞれのクラス名のルールを変更することもできます。
次のように Components と Variants のクラス名のパターンを変更してみます。

```yml:.stylelintrc
extends:
  - stylelint-rscss/config
rules:
  rscss/class-format:
    - true
    - component: ^[a-z0-9]+[A-Z][a-zA-Z0-9]+$
      # element: ^[a-z0-9]+$
      # helper: ^_[a-zA-Z0-9]+$
      variant: ^[A-Z][a-zA-Z0-9]+$
```

これで、それぞれのパターンに違反しているクラス名が存在すれば stylelint が警告してくれます。

![](https://storage.googleapis.com/zenn-user-upload/ft2yovw6pqnpkiz4temdegamshls)

# 参考

* [rscss](https://rscss.io/)
* [rstacruz/stylelint-rscss: Validate CSS with RSCSS conventions](https://github.com/rstacruz/stylelint-rscss)
* [[BEM to RSCSS] Quick Migration Guide - Qiita](https://qiita.com/simochee/items/3e537f530ca94ce6fb3a)
