---
title: 'Prismaでのテスト'
date: '2022-11-05'
---

タイトル：Prisma を使ったテスト

日付：2022 年 11 月 5 日

最近、Next.js と Prisma を使ったフルスタックアプリ開発を試しています。ちなみに、Prisma のドキュメントを見てみてください。

[Prisma を使った統合テスト](https://www.prisma.io/docs/guides/testing/integration-testing)

> 現実の環境をシミュレートする方法の一つは、Docker を使用してデータベースとテストデータをカプセル化することです。これは、テストと共にスピンアップしてダウンすることができるため、プロダクションデータベースから独立した環境として機能します。

> このガイドでは、マシンに Docker と Docker Compose がインストールされていること、そしてプロジェクトに Jest がセットアップされていることを前提としています。

そうですが、本当にテストのためにそこまで準備する必要がありますか？もっと簡単にテストを実行したいです。

そして、これは重要です。ドキュメントには、テストケースごとにデータベースを簡単にクリーンアップする方法が書かれていないため、テストが独立しておらず、テストの順序に応じて動作が変わってしまいます。

では、やってみましょう。

今回は、次のスキーマを準備しました。

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DB_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String
}
```

```
npx prisma generate
```

最初に、Dotenv を使用して、テスト用のデータベースの URL を指定します。

```
npm i -D dotenv
```

```
# .env.test
DB_URL=postgresql://user:password@127.0.0.1:5432/example-test
```

次に、テストプロセスを定義します。ここでは、スクリプトの「pre」と「post」関数を使用しましょう。

```json
{
  "scripts": {
    "pretest": "pg_ctl -D db/pgdata start && dotenv -e .env.test -- prisma migrate dev",
    "test": "dotenv -e .env.test -- mocha",
    "posttest": "pg_ctl -D db/pgdata stop"
  }
}
```

ここでポイントとなるのは、Dotenv の API を使って`.env.test`を使用することです。

最後に行うべきことは、最初に申請したデータベースのクリーンアップです。mocha の場合、beforeEach、afterEach を使用してこれを行います。

```javascript
import { PrismaClient } from '@prisma/client'

export class PrismaCleaner {
  constructor(prisma = new PrismaClient()) {
    this.prisma = prisma
    const propertyNames = Object.getOwnPropertyNames(prisma)
    this.modelNames = propertyNames.filter((name) => this.isModelName(name))
  }

  async clean() {
    console.log(`Database cleaning...`)
    return Promise.all(
      this.modelNames.map((modelName) => this.prisma[modelName].deleteMany())
    )
  }

  /**
   * @param {String} name
   * @returns {Boolean}
   */
  isModelName(name) {
    return !name.match(/^(_|\$)/)
  }
}
```

```javascript
import * as assert from 'assert'
import { PrismaCleaner } from '../index.js'
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()
const cleaner = new PrismaCleaner()

describe('prisma-cleaner', () => {
  beforeEach(async () => {
    await cleaner.clean() # ここ
  })

  afterEach(async () => {
    await cleaner.clean() # ここ
  })

  describe('first creation', () => {
    it('creates a user', async () => {
      const user = await prisma.user.create({
        data: {
          name: 'John Lenon',
          email: 'john@example.com'
        }
      })
      assert.equal(user.name, 'John Lenon')
    })
  })

  describe('second creation', () => {
    it('creates a user', async () => {
      const user = await prisma.user.create({
        data: {
          name: 'John Wick',
          email: 'john@example.com'
        }
      })
      assert.equal(user.name, 'John Wick')
    })
  })
})
```

以上で完了です。

ちなみに、このデータベースのクリーンアップのためにいくつかのライブラリが存在することがわかりましたが、どれも完璧に機能しなかったようです。

- [emerleite/node-database-cleaner: The simplest way to clean your database after tests](https://github.com/emerleite/node-database-cleaner)
- [blazing-edge-labs/node-postgres-cleaner: Extremely simple way to clean your postgres database](https://github.com/blazing-edge-labs/node-postgres-cleaner)

とりあえず、自分でライブラリを公開することにしました。

- [cc-kawakami/prisma-cleaner: Prisma Cleaner is a utility for cleaning database with Prisma in testing. You can use in Jest or Mocha, etc.](https://github.com/cc-kawakami/prisma-cleaner)

Prisma の開発体験は予想していたほど悪くないと感じましたが、テスト環境はまだ準備ができていないと感じました。それに取り組みたかったので、この記事を書きました。

読んでくださってありがとうございました。
