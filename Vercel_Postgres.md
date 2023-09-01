---
title: Vercelがやりよった (Vercel Postgres)
date: '2023-05-02'
---

これの話っすね。
[Introducing storage on Vercel – Vercel](https://vercel.com/blog/vercel-storage)

Heroku で動いてたアプリと比べてみた。

## 差分

```diff
diff --git a/Procfile b/Procfile
deleted file mode 100644
index bc6eb1f..0000000
--- a/Procfile
+++ /dev/null
@@ -1,2 +0,0 @@
-release: npx blitz prisma migrate deploy
-web: npm run start:production
diff --git a/db/schema.prisma b/db/schema.prisma
index 0f8f549..3a35c9b 100644
--- a/db/schema.prisma
+++ b/db/schema.prisma
@@ -2,8 +2,9 @@
 // learn more about it in the docs: https://pris.ly/d/prisma-schema

 datasource db {
-  provider = "postgresql"
-  url      = env("DATABASE_URL")
+  provider          = "postgresql"
+  url               = env("POSTGRES_PRISMA_URL") // uses connection pooling
+  shadowDatabaseUrl = env("POSTGRES_URL_NON_POOLING") // used for migrations
 }

 generator client {
diff --git a/package.json b/package.json
index f6484ea..a1e0358 100644
--- a/package.json
+++ b/package.json
@@ -3,14 +3,13 @@
   "version": "1.0.0",
   "scripts": {
     "dev": "blitz dev",
-    "build": "blitz build",
+    "build": "blitz prisma migrate deploy && blitz build",
     "start": "blitz start",
     "studio": "blitz prisma studio",
     "lint": "eslint --ignore-path .gitignore --ext .js,.ts,.tsx .",
     "test": "vitest run --passWithNoTests",
     "test:watch": "vitest",
-    "prepare": "husky install",
-    "start:production": "blitz start --port $PORT"
+    "prepare": "husky install"
   },
   "prisma": {
     "schema": "db/schema.prisma"
```

## 結果

|             | Vercel               | Heroku        |
| ----------- | -------------------- | ------------- |
| Migration   | `npm run build`      | `Procfile`    |
| Deploy time | 1m30s                | 4m            |
| Performance | Good                 | Good          |
| Pricing     | Free (〜 1 Database) | $7.00 + $9.00 |

## 結論

よき。Vercel がんばれ。
