---
title: Sanity CMS
date: '2023-05-05'
---

## Vercel の Visual Editing

なんかブラウザで CMS のコンテンツを編集できるやつです。

https://twitter.com/vercel/status/1653791607609761799?s=20

[Visual Editing | Vercel Docs](https://vercel.com/docs/workflow-collaboration/visual-editing)

> Visual Editing, currently in a limited private beta, is accessible for Enterprise Teams using Sanity as their content editing workflow. To participate in this or future private betas, contact us for more information.

> Visual Editing は現在、限定的なプライベートベータ版で、Sanity をコンテンツ編集ワークフローとして使用しているエンタープライズチームがアクセスできます。このプライベートベータや今後のプライベートベータへの参加をご希望の方は、弊社までお問い合わせください。

とのことなので、CMS [Sanity](https://www.sanity.io/) を試しに使ってみようと思います。

## Sanity Web でプロジェクトを用意

この辺はぽちぽちやってけば大丈夫です。

最後に、 local で立ち上げる sanity の管理画面 （studio と呼ばれているらしい） をインストールするコマンドを教えてくれました。

```
npm create sanity@latest -- --template get-started --project *** --dataset production --provider google
```

というわけで、Sanity にはクラウドの管理画面がないみたいです。
この studio をどこかに deploy すれば、コラボレーションもできるようです。

https://www.sanity.io/solution/collaboration

#### 追記: 2023/05/06

公式のドキュメントに deployment の紹介があるようです！

https://www.sanity.io/docs/deployment

## Next.js の環境にインストール

```diff
  ./
  ├── pages/
  │   ├── ...
+ │　　　　　　└── studio/
+ ├── sanity/
+ │   ├── lib/
+ │   ├── schema.ts
+ │   └── schemas/
+ ├── sanity.cli.ts
+ ├── sanity.config.ts
  ├── package-lock.json
  ├── package.json
  ├── ...
```

- `./sanity/` - なんかデータの schema やらなんやら
- `./pages/studio/` - `localhost:3000/studio` で管理画面が開ける

## 実際にデータを編集する

画面はこんな感じ。別に使いやすくはなかったです。

![](https://storage.googleapis.com/zenn-user-upload/4e2ac678da93-20230505.png)

## Next.js から呼び出す

大まかには以下のような感じ。

```ts
import { createClient } from 'next-sanity'
import { apiVersion, dataset, projectId } from '@/sanity/env'

const client = createClient({
  projectId: projectId,
  dataset: dataset,
  apiVersion: apiVersion,
  useCdn: false
})

export const getStaticProps = async () => {
  const posts = await client.fetch(`*[_type == "post"]`)
  // ...
}
```

気になるところは後述します。

## 辛かったところ

### クエリが独自

とにかくクエリを覚えないといけないのが微妙でした。Copilot がなかったら補完も効かない。

[How GROQ Queries Work - Step by Step Guide](https://www.sanity.io/docs/how-queries-work)

```ts
client.fetch(`*[_type == "post" && slug.current == ${slug}]`)
```

#### 追記: 2023/05/06

一応 GraphQL サポートがあるようです。

https://www.sanity.io/docs/graphql

### ~~ブログの Body とかが、HTML 文字列じゃなくて、ブロック構造になっているが、Parser は自前で用意しろスタイル~~ Portable Text という規格を採用してるみたい

こんな感じで、結構気を遣って構造化してくれているんですが、Parser が用意されてないので、HTML に直すところでめんどくさくなり諦めました。

```ts
body: [
  {
    style: 'h2',
    _key: 'f8a7d4e3e019',
    markDefs: [],
    children: [Array],
    _type: 'block'
  },
  {
    markDefs: [],
    children: [Array],
    _type: 'block',
    style: 'normal',
    _key: '78957f0d9762'
  }
  // ...
]
```

#### 追記: 2023/05/06

Portable Text という規格に則っているようです。

https://github.com/portabletext/portabletext

公式でライブラリが紹介されていたので試しに使ってみようと思います！

https://www.sanity.io/docs/presenting-block-text

### せっかく Schema as Code なのに、Type や Interface がない

ので、自前で用意しました。

#### 追記: 2023/05/06

有志がライブラリを作ってくれているみたいです！

https://github.com/ricokahler/sanity-codegen

## まとめ

ドキュメントを一通り読みましたが、抜け漏れがあるかもしれません。補足等お願いします。

#### 補足: 2023/05/06

色々補足していただいて、懸念点が払拭されつつありますので改めて使ってみようと思います！
特に Portable Text あたりは勉強になりました。
