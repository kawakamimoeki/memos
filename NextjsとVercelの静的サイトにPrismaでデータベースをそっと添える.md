---
title: Next.jsとVercel の静的サイトにPrismaでデータベースをそっと"添える"
date: '2022-11-08'
---

先日、自分のウェブサイトに「いいね！」ボタンを付けました。

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/61v5i2tn2mc2gmk8u1up.png)

こんなやつ。

これまでは、静的な Web サイトにちょっとしたデータベース処理を加えようとすると、アプリの実装を静的サイトジェネレータから Ruby on Rails などのサーバサイドフレームワークに変更するか、別の DB サーバを用意して API を叩くしかありませんでした。**機能によってはその追加がアーキテクチャ全体に影響を及ぼしていた**のです。

しかし、今回の実験で、Next.js と Vercel がアーキテクチャを大きく変えずにウェブサイトの機能を拡張できる、非常に**アプリケーションファースト**な思想を持っていることを実感しました。
これは、Vercel は Serverless Function で Rest API を受け取ることができることが大きく寄与しています。

見ていきましょう。

まず、「いいね！」の UI を実装します。

```jsx
import { useState } from 'react'
import { MdThumbUp } from 'react-icons/md'

export default function Component(props) {
  // props.likeCountをserver sideから取得
  const [likeCount, setLikeCount] = useState(props.likeCount)
  const [liked, setLiked] = useState(false)
  const createLike = async () => {
    if (liked === false) {
      // TODO: create like
      setLikeCount(likeCount + 1)
      setLiked(true)
    }
  }
  return (
    <>
      <span className={liked ? 'text-blue-500' : undefined}>
        <span
          onClick={createLike}
          className="ml-4 mb-4 align-bottom cursor-pointer inline-block bg-white border-2 border-gray-800 rounded-full px-4">
          <MdThumbUp className="inline mr-2" />
          {likeCount}
        </span>
      </span>
      <span className="inline-block ml-4 mb-4 align-bottom">
        {liked && <span>Thank you!</span>}
      </span>
    </>
  )
}
```

今回は ORM として、Prisma を利用したいと思います。

```
yarn add prisma
```

これがスキーマファイルである `prisma/schema.prisma` です。
今回は非常にシンプルなモデルです。

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Like {
  id    Int     @id @default(autoincrement())
}
```

そして、マイグレーションを生成して適用します。

```
yarn run prisma migrate dev
```

次に、データベースにアクセスするための prisma クライアントを生成します。

```
yarn run prisma generate
```

次に、実際にデータベースにアクセスする部分を実装します。

あるページから `getServerSideProps` (サーバーサイドレンダリング) という関数をエクスポートすると、Next.js は `getServerSideProps` が返すデータを用いて、リクエストごとにこのページをプリレンダリングします。ここで Like をカウントして props に渡します。

```javascript
// pages/index.js

import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()

export default function Home(props) {
  //...
}

export async function getServerSideProps(context) {
  const likeCount = await prisma.like.count()
  return {
    props: { likeCount: likeCount }
  }
}
```

続いて、Like のレコードを作成するための post エンドポイントを作成します。

```javascript
// pages/api/like/create
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()

export default async function handler(req, res) {
  const like = await prisma.like.create()
  res.status(201).json(like)
}
```

で、いいねボタンを押したときに発動する`ceateLike`callback に API を叩く処理を追加します。

```javascript
const createLike = async () => {
  if (liked === false) {
    fetch('/api/like/create')
    setLikeCount(likeCount + 1)
    setLiked(true)
  }
}
```

これで実装は終了です。

次に、Vercel で Prisma を使うための設定と、PostgreSQL サーバーを用意しましょう。

PostgreSQL サーバは[render](<(https://render.com/)>)で用意しました。

Vercel のドキュメントに Prisma を Vercel へのデプロイ方法をまとめてくれているので、これに沿ってやってみます。

[How to deploy a Prisma app to Vercel](https://www.prisma.io/docs/guides/deployment/deployment-guides/deploying-to-vercel)

```js
// package.json

{
  "scripts": {
    "vercel-build": "prisma generate && prisma migrate deploy && next build"
  }
}
```

次に、Prisma から Vercel への環境変数を設定します。

render の PostgreSQL から URL を取得して、
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2ceug6vaav6yerkxga0p.png)

Vercel にセットします。
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/njb5go3nvkfusb5ma8gy.png)

あとは環境変数のセットを契機に勝手にデプロイが走るので、それで終わりです。

大きなアーキテクチャの変更をすることなく、データベースの機能を追加することができました。とても自由な開発体験で、Next.js と Vercel がやろうとしていることが少し分かったような気がします。

お読みいただきありがとうございました。
