---
title: ViteでCSSから画像を参照する
date: '2022-10-24'
---

Vite は、`postcss-import` を利用して `@import` を実現しています。

[Features | Vite](https://vitejs.dev/guide/features.html#css)

例えば以下のようなコードがあるとします。

```
src/
  ├── img/
  │    └── about-mainvisual.png
  └── css/
       ├── application.css
       └── pages/
             └── about.css
```

```css:src/css/application.css
@import "./pages/about.css"

main {
  background-color: white;
}
```

```css:src/css/pages/about.css
.about__mainvisual {
  background-image: url(../../img/about-mainvisual.png);
}
```

**これだと上手くいきません。**

1. `@import` の解決
2. `url()` の解決

の順に Vite は処理します。

つまり、1 の時点で

```css:src/css/application.css
.about_mainvisual {
  background-image: url(../../img/about-mainvisual.png);
}

main {
  background-color: white;
}
```

という CSS が一時的に生成されているのと同義であると言えます。
だから、`url(../../img/about-mainvisual.png)` のパスが間違っていると言えるのです。

どうしたらいいのでしょう？
正解は、

```diff css:src/css/pages/about.css
.about__mainvisual {
-  background-image: url(../../img/about-mainvisual.png);
+  background-image: url(/img/about-mainvisual.png);
}
```

**絶対パスで指定してください。そうすれば、画像の参照元に依存せずに解決することができます。
ここで、Vite の `root` 設定が `src` になっていることが前提です。**
