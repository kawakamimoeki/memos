---
title: Sassでも、利用していないCSSセレクタを特定したかった
date: '2022-11-02'
---

CSS なら、https://purgecss.com がある。

では Sass なら？
ビルドされた結果からだけでなく Sass のソースコードからも使っていないセレクタを抹消したい気持ちはありますよね。

試みました。[cc-kawakami/uchini: Find unused Sass codes](https://github.com/cc-kawakami/uchini)

```bash
npm i -g uchini
```

```bash
$ uchini ls --help
List CSS selectors usage

USAGE
  $ uchini ls --scss <value> --content <value> [--output
    <value>] [--unusedOnly]

FLAGS
  --content=<value>  (required) Path to content file
  --output=<value>   Path to output file
  --scss=<value>     (required) Path to CSS file
  --unusedOnly       Unused selectors only

DESCRIPTION
  List CSS selectors usage

EXAMPLES
  $ uchini ls --css path/to/css --content path/to/content --output path/to/output --unusedOnly
```

```bash
uchini ls --scss path/to/scss --content 'path/**/*.html' --output output.json
```

```json:output.json
[
  {
    "selector": ".header",
    "positions": [
      "frontend/stylesheets/_layout.scss:20:2"
    ]
  },
  {
    "selector": ".card",
    "positions": [
      "frontend/stylesheets/_card.scss:1:0"
    ]
  }
]
```

内部では以下のものを利用しています。

- [sass - npm](https://www.npmjs.com/package/sass)
- [purgecss - npm](https://www.npmjs.com/package/purgecss)
- [source-map - npm](https://www.npmjs.com/package/source-map)

まずは、Sass をコンパイルします。

```javascript
const compiled = sass.compile(scss, {
  style: 'compressed',
  sourceMap: true,
  logger: sass.Logger.silent
})
```

そして、source-map で consumer を生成します。

```javascript
const consumer = await new sourceMap.SourceMapConsumer(compiled.sourceMap)
```

次に、purgecss で利用されていない CSS セレクタを洗い出します。

```javascript
const rejectedCSS = await new PurgeCSS().purge({
  content: [content],
  css: [{ raw: css }],
  rejected: true
})[0].rejected
```

そして、consumer によってソースマップを利用して、CSS のある行に対応する Sass の行を辿ることができます。

```javascript
const position = this.consumer.originalPositionFor({
  line: 1, // compressed でソースマップが一行になっているので
  column: startOfSelector
})
```

これで終わり。
