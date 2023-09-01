---
title: Sassから純CSSの世界へ戻ろうとして踏みとどまった話
date: '2022-11-01'
---

**Sass の問題点ははっきりしていました**。同時に、**CSS の新しい仕様たちは非常に明快**で注目に値するものでした。この辺りを踏まえて純 CSS へ戻ろうとしましたが、結果的に踏みとどまったので、その辺りの経緯をまとめます。

## 🙋‍♀️ そもそも我々は Sass をなぜ使っているのか？

個人的に、チーム的にこんな感じでした。この辺りは人それぞれあると思います。

1. クラス名を付けずにタグに対してスタイルを指定したいときにスタイル規則を**ネスト**したい
2. ソースコードの視認性を高めるためにコンテンツのセクションごとにスタイル規則を**ネスト**したい
3. 検索性を高めるために目的別・コンテンツ別に**ファイルを別けたい**
4. 大規模なプロジェクトでは**色のパターンを定義して再利用したい**
5. メディアクエリの**画面幅の数値を定義して再利用したい**
6. 大型のプロジェクトでは`@extend`を利用して**スタイルを共通化**している
7. 一部のプロジェクトでは`@mixin`を利用して**ベンダープレフィクスを共通化**（最近は autoprefixer を使っているが）したり、**スタイル規則を定義**したり、**ブレイクポイントを共通化**したりしている

## 🤷‍♀️ Sass の何が困っているか？

### 1. 全ての機能がプリセットされている

**利用していない機能もプリセットされている**。チームとして、どの機能を使っていきたいか、の基準を明確化しなければ、**新規参画したメンバーは片っ端から機能を使い倒し**、レビューが辛くなる可能性が高い。

### 2. 言語の実装が複数あったが廃止されているので移行対応が必要

- [Sass: LibSass is Deprecated](https://sass-lang.com/blog/libsass-is-deprecated)
- [Sass: Ruby Sass Has Reached End-Of-Life](https://sass-lang.com/blog/ruby-sass-is-unsupported)

Dart Sass に加えて、[Sass: Embedded Sass is Live](https://sass-lang.com/blog/embedded-sass-is-live)も出てきました。

### 3. 本質的に使用の方針が不安定になる傾向がある

**Sass はリアルワールドで便利とされている CSS 設計理論に合わせて、仕様を変化させるような、現実的な方針をとっています**。以下の機能は現在進行形で廃止に向かっているので、過去のコードを変更する必要があります。

#### `@import`

`@import`だと階層的に`@import`を利用している場合でもスタイル規約にアクセスできるますが、この仕様が問題視されはじめ、代替案として`@use`が採用されました。これは呼び出したファイルでしかスタイル規約にアクセスできないようになります。

[Sass: @import](https://sass-lang.com/documentation/at-rules/import)

### 4. CSS 本体の仕様とコンフリクトする可能性がある

#### `/`を利用した除算

CSS では`/`を区切り文字とするプロパティがあります。
この時に Sass からしてみれば、その`/`が区切り文字なのか、除算なのか区別がつかなくなってしまいます。これの代替案として`Math.div`を推奨しています。

[Sass: Breaking Change: Slash as Division](https://sass-lang.com/documentation/breaking-changes/slash-div)

## 🤔 ところで CSS の動向は？

[cssdb - CSS Database](https://cssdb.org/) では**stage と呼ばれる、新しい機能の標準化への段階**を定義しています。

| Stage  |         Level          | 成熟レベル |
| :----: | :--------------------: | :--------: |
| Stage4 | Standardized（標準化） |     高     |
| Stage3 |    Embraced（包容）    |     :      |
| Stage2 |   Allowable（許容）    |     :      |
| Stage1 | Experimental（実験的） |     :      |
| Stage0 | Aspirational（野心的） |     低     |

Stage が上がるにつれて、実際にブラウザの対応状況も良くなっていっているようですね。
また、[postcss-plugins/plugin-packs/postcss-preset-env](https://github.com/csstools/postcss-plugins/tree/main/plugin-packs/postcss-preset-env)ではこの stage に準拠して、将来的に標準化されるであろう機能を利用することができます。

Sass でよく使っていたような機能も結構出てきているみたいですね。

| 実現したいこと         | CSS                          | PostCSS プラグイン            |
| ---------------------- | ---------------------------- | ----------------------------- |
| ネスト                 | nesting rule (stage 1)       | postcss-nesting               |
| ファイル分割           | -                            | postcss-import                |
| 色の再利用             | custom property (stage 3)    | postcss-custom-property       |
| メディアクエリの再利用 | custom media query (stage 2) | postcss-custom-media          |
| クラス継承             | 議論中                       | postcss-extend                |
| Mixin                  | 議論中 [^1]                  | postcss-mixins, postcss-apply |

[^1]: `@apply`が議論されているが、技術的な課題があって難航しているらしい

それぞれのルールの記法については、色々調べてみてください。

### Mixin の用途別の再現方法

- 共通のスタイル規則を定義 => シングルクラスを採用しない
- ベンダープレフィクスの共通化 => Autoprefixer
- ブレイクポイントの共通化 => CSS の custom media query

## 🫣 stage が低いルールの仕様はとても不安定

W3C で定義されている stage は、正式なリリースバージョンではなく、その使用の安定性は低いです。
**stage が低いものは、仕様が変更されたり、reject される可能性が高いことも考慮しなければならない**。
CSS の草案に対しての知識や経験がほとんどないうちには、postcss-preset-env のデフォルト（stage 2）に従った方がいいと思われます。

例えば、nesting rule はまだ stage 1 なので、仕様が変更される可能性が高いです。**もし postcss-preset-env で実装しても、もしかすると nesting の部分だけあとで実装し直す必要があるかもしれませんね**。

## 😌 postcss-preset-env の採用はまだ早いかも

上の話でやっぱり postcss-preset-env の採用はまだ早いかもしれません。
今後は、

- **CanIUse などを使ってブラウザの対応状況を見ていく**
- **もし使いたい機能の stage が 4 になり、ブラウザ対応が十分になれば採用する**
- **autprexier のように、CSS 本体とは別なアプローチのために PostCSS は採用を進めていく**

かなと思いました。ありがとうございました。
