---
title: 'トランスパイラとESLintは完璧じゃない'
date: '2022-11-01'
---

## 前提

- トランスパイラ利用の有無に関わらず、ES2017 相当の（2021/10 時点でブラウザの対応率が十分に高い）記述がユーザーのブラウザで動くようにする、という方針
- トランスパイラを用いても、**ECMAScript の全ての記述を変換してくれるわけではない**ことは薄々気づいていた
- ESLint を用いても、**ECMAScript の全ての記述を解析してくれるわけではない**ことは薄々気づいていた

## 目的

- ECMAScript を利用した場合に
  - トランスパイラは何を変換し、何を無視するのかを検証する
  - ESLint は何を解析し、何を無視するのかを検証する

## 検証用の入力

- ES6 〜 ESNext の規格に基づいたコードサンプル
  - 利用する機会がありそうな機能をピックアップ

```javascript
/**
 * ES2015
 * ------------------------------------------------
 */
/** const, let */
const cst = 1
let lt = 2
console.log(cst)
console.log(lt)

/** Allow function */
const fn = () => {
  console.log('fn')
}

/** classes */
class Hoge {
  fuga() {
    console.log('fuga')
  }
}

/**
 * ES2016
 * ------------------------------------------------
 */
/** Array.prototype.includes */
const array = [1, 2, 3, 4, 5]
console.log(array.includes(2))

/** べき乗演算子 */
console.log(2 ** 2)

/**
 * ES2017
 * ------------------------------------------------
 */
/** Async functions */
async function log() {
  console.log('hoge')
}

/** Object.values() */
const obj = { hoge: 0, fuga: 1, piyo: 2 }
console.log(Object.values(obj))

/**
 * ES2018
 * ------------------------------------------------
 */
/** Spread Properties */
const hoge = { fuga: 'piyo' }
console.log({ ...hoge })

/** RegExp named capture groups */
console.log(/(?<year>[0-9]{4})年/.test('2021年'))

/** RegExp Lookbehind Assertions */
console.log(/(?<=[0-9]+)\.[0-9]+/.test('.34'))

/**
 * ES2019
 * ------------------------------------------------
 */
/** flat array methods */
console.log([[1, 2], 3, 4].flat())

/**
 * ES2020
 * ------------------------------------------------
 */
/** Optional chaining operator */
console.log(hoge?.fuga)

/**
 * ES2021
 * ------------------------------------------------
 */
/** Logical assignment operators */
console.log((a ||= b))

/** Numeric separators */
console.log(100_000_000)

/**
 * ESNext
 * ------------------------------------------------
 */
/** static class field */
class Foo {
  static bar = 1
}

console.log(Foo.bar)
```

## 対象のトランスパイラと設定

| No  |          名前          |     ターゲット     |
| :-: | :--------------------: | :----------------: |
|  1  |  Babel / conservative  | - ( `> 1% in JP` ) |
|  2  |  Babel / progressive   | - ( `> 5% in JP` ) |
|  3  |   esbuild / default    |       ESNext       |
|  4  |  esbuild / to ES2017   |       ES2017       |
|  5  |         rollup         |   not transpile    |
|  6  | rollup-plugin-esbuild  |       ES2017       |
|  7  |  Typescript / default  |        ES3         |
|  8  | Typescript / to ES2017 |       ES2017       |

## 検証結果

### すべてのトランスパイルで変換されない記述

- RegExp named capture groups (ES2018)
- RegExp Lookbehind Assertions (ES2018)
- flat array methods (ES2019)

### ESLint で検証できない記述

- flat array methods (ES2019)
  - [freaktechnik/eslint-plugin-array-func](https://github.com/freaktechnik/eslint-plugin-array-func) を試してみたがダメ

## その他気がついたこと

- esbuild はデフォルトでコメントが削減される

## (おまけ) 実行時間

| No  |          名前          |     ターゲット     | 時間 ( s ) |
| :-: | :--------------------: | :----------------: | :--------: |
|  1  |  Babel / conservative  | - ( `> 1% in JP` ) |    0.63    |
|  2  |  Babel / progressive   | - ( `> 5% in JP` ) |    0.53    |
|  3  |   esbuild / default    |       ESNext       |    0.31    |
|  4  |  esbuild / to ES2017   |       ES2017       |    0.18    |
|  5  |         rollup         |   not transpile    |    0.20    |
|  6  | rollup-plugin-esbuild  |       ES2017       |    0.27    |
|  7  |  Typescript / default  |        ES3         |    1.21    |
|  8  | Typescript / to ES2017 |       ES2017       |    1.36    |

## 参考

- [ESLint - Pluggable JavaScript linter](https://eslint.org/)
- [Babel · The compiler for next generation JavaScript](https://babeljs.io/)
- [esbuild - An extremely fast JavaScript bundler](https://esbuild.github.io/)
- [rollup.js](https://rollupjs.org/guide/en/)
- [egoist/rollup-plugin-esbuild: Use ESBuild with Rollup to transform ESNext and TypeScript code](https://github.com/egoist/rollup-plugin-esbuild)
- [TypeScript: JavaScript With Syntax For Types.](https://www.typescriptlang.org/)
- [ES6〜ES10(es2015〜es2019)まとめ - Qiita](https://qiita.com/ozoneboy/items/9c11ac3323ca94919052)
