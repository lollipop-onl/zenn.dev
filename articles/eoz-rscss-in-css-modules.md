---
title: 
emoji: 🍭
type: tech
topics: [css, rscss, EveOneZenn]
published: false
---

# はじめに

この記事は [#EveOneZenn](https://zenn.dev/topics/eveonezenn) (Everyday One Zenn) の８日目です。

概要書く

**前回：**
https://zenn.dev/lollipop_onl/articles/eoz-judge-js-invalid-date

# rscss とは

https://rscss.io/

> Styling CSS without losing your sanity

rscss は BEM と似た `Components`・`Elements`・`Variants` をベースの構成とする非常にシンプルな CSS クラス設計の一つです。

## 特徴

* ネストは１段まで
* 必ず兄弟(`>`)としてセレクタを指定する
* Componentのクラス名は `-` を含む２単語以上で
* Elementのクラス名は `-` を含まないなるべく１単語で
* Variantのクラス名は `-` からはじめる

## BEM との比較

次のコードは rscss と BEM で同じ要素をマークアップしたものです。
BEM には `.contact`・`.contact__submit-button`・`.contact__submit-button--disabled` の３つのクラスが、 rscss には `.contact-form`・`.submit`・`.-disabled` の３つのクラスが定義してあります。

```scss
// BEM
.contact { // Block
  &__submit-button { // Element
    &--disabled { } // Modifier
  }
}

// rscss
.contact-form { // Component
  & > .submit { } // Element
  & > .submit.-disabled { } // Variant
}
```

```html
<!-- RSCSS -->
<form className="contact">
  <button className={classNames(
    'contact__submit-button',
    { 'contact__submit-button--disabled': isDisabled }
  )}>Submit</button>
</form>

<!-- rscss -->
<form className="contact-form">
  <button className={classNames(
    'submit',
    { '-disabled': isDisabled }
  )}>Submit</button>
</form>
```

rscss の一番の特徴は **Components と Elements を親子の関係で定義する** というものです。
これにより、 rscss ではクラス名を短くしつつ、競合しないように運用できます。

rscss と BEM の書き方の比較は、次の記事も併せてご覧ください。

https://qiita.com/simochee/items/3e537f530ca94ce6fb3a

# CSS Modules で rscss の設計を採用する

上記で紹介したとおり、 rscss はクラス名を短く保ちつつ BEM とほぼ同等の設計を実現できます。
ただ、グローバルスコープでは Components のクラス名が重複しないよう運用しなければならないためあまり現実的ではありません。

そこで、小さなスコープでの重複を注意するだけで良い CSS Modules や Scoped CSS であれば rscss を運用しやすい
