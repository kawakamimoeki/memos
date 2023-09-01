個人的に収集していたエンジニアリングに関係するブックマークをオープンソースにしてみたら面白いかもしれないと考えました。日本語コミュニティの叡智の結晶を育めるのではという厨ニ心も騒ぎはじめ、実際に手を動かしはじめました。

リンク集を定義した JSON と、それをもとに構築されたウェブサイトを実装しました。

https://www.stoc.dev/

![](https://storage.googleapis.com/zenn-user-upload/ac6e0c2f7160-20230830.png)

GitHub のリポジトリで公開しています。

https://github.com/moekidev/stoc.dev

今回はこちらの実装方針と、それに至るまでの思考をまとめたいと思います。

## 簡単な構成紹介

stoc.dev は"ドキュメント"と"タグ"から構成されています。ドキュメントはリンクに対応していて、ブログ記事やツールのドキュメント、プラクティスが紹介されたウェブサイトなどです。そしてそれらのドキュメントをまとめるためにタグを用意しています。JSON ファイルの中身は以下のとおりです。

```json
{
  "tags": [
    {
      "slug": "svelte",
      "name": "Svelte",
      "icon": true,
      "createdAt": "2023-08-28T14:25:20.061Z"
    },
    {
      "slug": "framework",
      "name": "フレームワーク",
      "icon": false,
      "createdAt": "2023-08-28T14:25:20.061Z"
    },
    {
      "slug": "official",
      "name": "公式",
      "icon": false,
      "createdAt": "2023-08-28T14:25:20.061Z"
    },
    {
      "slug": "javascript",
      "name": "Javascript",
      "icon": true,
      "createdAt": "2023-08-28T14:25:20.061Z"
    }
  ],
  "docs": [
    {
      "id": "203f102f-ec52-479b-9a9e-8ea7f3cd01e2",
      "url": "https://tailwindcss.com/",
      "tags": ["tailwindcss", "css", "official"],
      "title": "Tailwind CSS - Rapidly build modern websites without ever leaving your HTML.",
      "description": "Tailwind CSS is a utility-first CSS framework for rapidly building modern websites without ever leaving your HTML.",
      "icon": "https://tailwindcss.com/favicons/favicon-32x32.png?v=3",
      "host": "tailwindcss.com",
      "image": "https://tailwindcss.com/_next/static/media/social-card-large.a6e71726.jpg",
      "createdAt": "2023-08-28T14:42:09.489Z"
    },
    {
      "id": "fed16a22-7a29-4f10-9715-50db07a70ab5",
      "url": "https://vercel.com/",
      "tags": ["vercel", "hosting", "official"],
      "title": "Vercel: Develop. Preview. Ship. For the best frontend teams",
      "description": "Vercel's frontend cloud gives developers the frameworks, workflows, and infrastructure to build a faster, more personalized Web.",
      "icon": "",
      "host": "vercel.com",
      "image": "https://assets.vercel.com/image/upload/front/vercel/dps.png",
      "createdAt": "2023-08-28T14:42:24.505Z"
    }
  ]
}
```

ウェブサイトは、SvelteKit を利用して Static なページを生成しています。サイトの構成は以下のとおりです。ドキュメントには UUID を定義し、タグについては、扱いやすかったので slug を利用しました。

```
GET /
GET /feed.xml
GET /search
GET /docs/:id
GET /tags/:slug
GET /terms
```

## 1. なぜ JSON か

今回の肝は、リンク集が私一人に依存しないことです。そこで、最優先事項を、リンク集の管理方法としました。つまり、リンク集にリンクを追加したり、タグを編集したりするためのフローをできるだけ効率的に、メンテコストを下げつつ、安全に行う方法を採用したいと思いました。検討した候補としては、以下のとおりでした。

#### A. RDB での管理

- 👍 検索などのデータ操作がしやすい
- 👍 データをスケールしやすい

#### B. JSON での管理

- 👍 GitHub で共同編集のためのフローを組むことができる
- 👍 サイトを静的に生成することができる

この比較で考慮した点は、

- RDB を利用しつつ共同編集の仕組みを用意するためのコストが高い
- 本当に有益な情報のみを集めたかったのでスケール感は比較的小さい（おそらく最大でも数千）

でした。以上を踏まえ、JSON で管理することにしました。実際に数千件の JSON からすべてのデータを Javascript に出力しつつページを生成してみたところ、Lighthouse でスコア 100 を維持することが出来ました。

## 2. JSON にカッコよくデータを挿入する

ただし、JSON でデータを管理する場合には、以下のような課題がありましたので一つ一つ解消していきました。

- URL に利用しやすい ID の生成
- 日付順で並び替えられるように
- CLI を利用したい
- リンク先の情報をいい感じに自動的に取得したい
- 画像も管理したい

これらを解消するために利用したライブラリは以下のとおりです。

↓Node.js で UUID を生成する

https://github.com/uuidjs/uuid

↓Node.js で JSON にデータを挿入する。

https://github.com/typicode/lowdb

↓Node.js で CLI を実装する。

https://github.com/cacjs/cac

↓ リンク先をスクレイピングする

https://github.com/jsdom/jsdom

↓ 画像をウェブサイトに最適化された Webp に変換する。

https://github.com/lovell/sharp

CLI を npm で管理するために、リポジトリに１つ階層を掘って、CLI 用のスペースを利用しました。

```
./
├── CODE_OF_CONDUCT.md
├── LICENSE.txt
├── README.md
├── cli/
│   ├── index.js*
│   ├── lib/
│   ├── package-lock.json
│   └── package.json
├── package-lock.json
├── package.json
├── postcss.config.js
├── prisma/
├── src/
├── static/
├── svelte.config.js
├── tailwind.config.js
├── tsconfig.json
├── uno.config.ts
└── vite.config.ts
```

CLI スペースの package.json では`bin`を定義して、CLI を提供します。

```json
{
  "name": "stoc-cli",
  "bin": {
    "stoc": "index.js"
  },
  "type": "module",
  "dependencies": {
    "cac": "^6.7.14",
    "lowdb": "^6.0.1",
    "node-fetch": "^3.3.2",
    "sharp": "^0.32.5",
    "uuid": "^9.0.0"
  }
}
```

そして、本体のワークスペースの package.json でローカルのパッケージをインストールします。

```
{
  "devDependencies": {
    "cli": "file:cli"
  }
}
```

最終的な CLI のインターフェースは以下の通りに出来ました。

ドキュメントを追加するコマンド。`stoc doc`。

```
$ npx stoc doc --help

Usage:
  $ stoc doc <url>

Options:
  --tags <tags>  Document tags
  -h, --help     Display this message
```

タグを追加するコマンド。`stoc tag`

```
$ npx stoc tag --help

Usage:
  $ stoc tag <slug> <name>

Options:
  --icon <icon>  Tag icon
  -h, --help     Display this message
```

実際に使う様子は以下のような感じです。

```
npx stoc doc https://example.com --tags tag1,tag2
```

これは、すでに URL が登録されている場合には、タグをマージする挙動になります。

また、タグを追加するときには以下のような感じになります。

```
npx stoc tag tag1 "Tag 1" --icon path/to/icon
```

これによって、所定の場所に 32px\*32px の webp に変換された画像ファイルが出力されます。

## 3. あいまい検索

タグやドキュメントをあいまい検索したかったので、クライアントサイドでの検索を実現しました。
これには、Fuse を利用しました。React にはいい感じのライブラリが見つからなかったので、この点は一般的な Javascript のライブラリを利用することができる Svelte を採用してよかった点です。

https://github.com/krisk/fuse

```html
<script>
  export let q: string = "";
  export let searchResult: any = shuffle(data.docs).slice(0, 10);

  const fuse = new Fuse(data.docs, { keys: ["title", "description", "url"] });

  const search = () => {
    if (q) {
      searchResult = orderBy(
        fuse.search(q).map((t) => t.item),
        "refIndex",
        "desc"
      ).slice(0, 10);
    } else {
      searchResult = shuffle(data.docs).slice(0, 10);
    }
  };
</script>

<input
  placeholder="ドキュメントを検索"
  type="search"
  bind:value="{q}"
  on:input="{search}"
/>
```

## まとめ

GitHub での共同編集がしやすい JSON を採用しつつ、データ管理やデータ操作をいい感じにすることを実現することが出来た気がします。よかった。
