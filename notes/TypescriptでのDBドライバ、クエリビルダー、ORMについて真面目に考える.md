---
title: Typescript でのDBドライバ、クエリビルダー、ORMについて真面目に考える（pg、Kysely、Prisma）
date: "2023-05-06"
---

## はじめに

https://vercel.com/docs/storage/vercel-postgres

Vercel Postgres の発表を受けて、個人的に

- Typescript で Postgres を利用する**シーンが増えそう**
- Vercel が wrapper を用意している **Kysely** という**クエリビルダー**の存在が気になり始める
- **Prisma** は使っていたが、複雑なクエリになると辛いシーンもあった
- `vercel/@postgres` は `pg` の wrapper だった。そもそもクエリビルダーや ORM を使うという**前提を疑ってみたい**

というあたりが頭の中でつながりはじめ、今回の記事で、Typescript で SQL を利用する方法を改めて検討してみました。

## 登場するもの

### DB ドライバ

手動で TCP パケットを生成してデータベースに配信します。データベースへの接続やコネクションプールを処理することになります。このレベルでは、生の SQL 文字列を書いてデータベースに渡し、データベースから応答を受け取ることになります。

例）

- [pg - npm](https://www.npmjs.com/package/pg)
- [mysql - npm](https://www.npmjs.com/package/mysql)
- [sqlite3 - npm](https://www.npmjs.com/package/sqlite3)

### クエリビルダー

SQL のクエリを Typescript で記述することができます。

例）

- [knex - npm](https://www.npmjs.com/package/knex)
- [kysely - npm](https://www.npmjs.com/package/kysely)

### ORM

名前の通り、RDB のレコードを、アプリケーションのオブジェクトにマッピングします。

例）

- [sequelize-typescript - npm](https://www.npmjs.com/package/sequelize-typescript)
- [prisma - npm](https://www.npmjs.com/package/prisma)
- [bookshelf - npm](https://www.npmjs.com/package/bookshelf)

## 全体の流れ

1. とりあえず ORM vs クエリビルダー、クエリビルダー vs ドライバの順にそれぞれの意見を見ていきます。
2. 感想
3. 実践

## 🙋‍♂️ ORM を使う人たちの意見

### みんな使っている

なぜか世の中の情報が多い。ActiveRecord の功績？

### データベースを簡単に差し替えられる

例えば、Postgres と MySQL の両方に対応している ORM を利用していれば、"比較的"簡単に行き来することができる。（頻度はともかく）

### ORM はオブジェクト指向のクエリ手法を SQL に変換できる

- アプリケーションはオブジェクト指向で書かれているのでその文脈で SQL を記述できる
- インターフェイスの記述に重点を置かないことでビジネスロジックに集中できる
- データベーススキーマとオブジェクト（型）を DRY に管理できない

### SQL を厳密にマスターするのは大変

- しっかりとしたトランザクションサポートがある
- SQL インジェクションの脆弱性をガードする

### 生のクエリもサポートしている

これにより、開発者は生のクエリを書くための手段も併用できる。

### TypeScript の力を借りたい

- 補完
- 型チェック

## 🙋‍♂️ クエリビルダを使う人たちの意見

### SQL を生で書くのは動的なクエリで辛い

`pg`:

```ts
client.query(
  'INSERT INTO users(name, email) VALUES($1, $2) RETURNING *',
  [name, email],
  (err, res) => {
    /** .. */
  }
)
```

Kysely:

```ts
kysely.selectFrom('users').where('name', '=', name).where('email', '=', email)
```

### ORM では実現できないクエリがたまにある

巨人の ActiveRecord でさえも実現できないクエリがあり、結局生の SQL を書く、というケースがある。

### 複雑な ORM での記述は非効率になる場合がある（パフォーマンス）

Kysely:

```sql
SELECT
  post.*, user.name AS user_name, user.email AS user_email
FROM
  post
LEFT JOIN
  user ON user.id = post.user_id
WHERE
post.category_id = ?;
```

Sequelize:

```sql
SELECT
  "category"."id", "category"."name", "user"."id" AS "user.id",
  "user"."name" AS "user.name", "user"."email" AS "user.email",
  "user->post"."category_id" AS "user.post.category_id",
  "user->post"."user_id" AS "user.post.user_id",
  "user->post"."unit" AS "user.post.unit"
FROM
  "user" AS "user"
LEFT OUTER JOIN (
  "post" AS "user->post"
  INNER JOIN "user" AS "user" on "user"."id"  = "user->post"."user_id"
) ON "category"."id"  = "user->post"."category_id"
WHERE
"category"."id"  = ?
```

### 生のクエリもサポートしている

これにより、開発者は生のクエリを書くための手段も併用できる。

### TypeScript の力を借りたい

- 補完
- 型チェック

## 🙋‍♂️ データベースドライバを直接利用する人たちの意見

### 特定の ORM やクエリビルダーを学んだとしても、言語やライブラリを跨いで知識を活かせない

例えば、同じ Node.js のライブラリでも Sequelize と Prisma では微妙に記述が違う。

Sequelize:

```ts
const users = await user.findAll({ where: { name: 'Kevin' } })
const user = users[0]
const id = user.dataValues.id
```

Prisma:

```ts
const users = await db.user.findAll({ where: { name: 'Kevin' } })
const user = users[0]
const id = user.id
```

対して、SQL の方言はほんの数種類しかない。
生の SQL を使ってクエリを生成する方法を学ぶことで、この知識を異なる言語を簡単に行き来することができる。

## 感想

### 実感できたところ

- ORM をいちいち覚えたくない
- SQL に精通しておきたい
- Typescript で補完してほしい
- ORM だと痒い所に手が届かない

### 不安に感じたところ

- 生で SQL を書いた場合のトランザクションや SQL インジェクション対策
- スキーマと type (オブジェクト)で記述が重複するのはどうなんだろ...

### 困らないかもと思ったところ

- クエリビルダーの仕様ってシンプルなので覚えるのは簡単そう

### 一旦の結論

- クエリビルダーを使ってみたい
- Prisma で schema を定義するのはやっぱやりたい

## 💪 Typescript での実践（Kysely x Prisma）

とりあえず、Prisma でスキーマを定義しつつ、kysely でクエリを書いていきます！

こんなものもあるみたいですが、今回は自分で書いていきます。
[valtyr/prisma-kysely: 🪄 Generate Kysely types directly from your Prisma schema!](https://github.com/valtyr/prisma-kysely)

`schema.prisma`:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        Int       @id @default(dbgenerated("uuid()"))
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  email     String    @unique
  name      String?
  posts     Post[]
}

model Post {
  id          Int      @id @default(dbgenerated("gen_random_uuid()"))
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  title       String
  body        String?
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId      Int
}
```

ここで注意するところは、`@default`です。
Prisma では一般的に、以下のように定義しますが、**これは JS ベースの実装なので、Kysely を利用して insert した場合には、実際のデータベースに反映されません。**

```schema
@default(gen_random_uuid())
```

ということで、以下のように SQL ベースの UUID を利用します。

```diff
- @default(uuid())
+ @default(dbgenerated("uuid()"))
```

Kysely の初期化。
ここでは、`@prisma/client` から `User`、`Post` を Kysely に渡しています。

```ts
import { Pool } from 'pg'
import { Kysely, PostgresDialect } from 'kysely'
import { User, Post } from '@prisma/client'

interface Database {
  User: User
  Post: Post
}

const db = new Kysely<Database>({
  dialect: new PostgresDialect({
    pool: new Pool({
      connectionString: process.env.DATABASE_URL
    })
  })
})

export { db as kysely }
```

実際に操作してみる:

```ts
const posts = kysely.selectFrom('Post').where('userId', '=', 1).execute()
```

## まとめ

一旦は、schema とオブジェクトを Prisma に任せ、Kysely でクエリを組むということをやってみました。これでしばらく試してみたいと思います。補足やアドバイス等ありましたらお願いします！
