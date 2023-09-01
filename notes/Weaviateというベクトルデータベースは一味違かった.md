---
title: Weaviate というベクトルデータベースは一味違かった
date: '2023-05-09'
---

## はじめに

ベクトルデータベースざっと調べた感じいくつか候補がありました。

- [Weaviate](https://weaviate.io/)
- [Elasticsearch](https://medium.com/gsi-technology/vector-databases-made-easy-51483c4e44d7)
- [Deep Lake](https://www.deeplake.ai/)
- [Pinecone](https://www.pinecone.io/)
- [Milvus](https://milvus.io/)
- [Qdrant](https://qdrant.tech/)

## 基準として何を気にしていたか

- ローカル環境を立ち上げられること
- データ構造がいい感じか
- upsert ができるか
- API がいい感じか

## どれも同じかと思ったら Weaviate が一味違かった

1. Docker イメージが用意されている
2. GraphQL API が用意されている
3. 内部で Embedding の module が用意してあってベクトル化もしてくれる
4. RDB の構造をそのままベクトルとして扱える

全部やばいが、特に 4 がやばい。他のベクトルデータベースは、文字列のベクトル + メタデータという感じだが、
Weaviate は、Class という概念とそれに付随する Properties という概念があって、例えば Author という Class の Properties を全部ベクトル化してくれる。
しかも、Author に紐づく Article とかもアソシエーションを定義できる。
なんでこの圧倒的違いが話題になっていないのか分らないけど個人的にはグッときた。

例えば、こんな感じのデータがって、

```json
{
  "name": "Alice Munro",
  "age": 91,
  "born": "1931-07-10T00:00:00.0Z",
  "wonNobelPrize": true,
  "description": "Alice Ann Munro is a Canadian short story writer who won the Nobel Prize in Literature in 2013. Munro's work has been described as revolutionizing the architecture of short stories, especially in its tendency to move forward and backward in time."
}
```

これをそのままベクトルデータに渡すことができる。

```json
{
    "id": "779c8970-0594-301c-bff5-d12907414002",
    "class": "Author",
    "properties": {
        "name": "Alice Munro",
        (...)
    },
    "vector": [
        -0.16147631,
        -0.065765485,
        -0.06546908
    ]
}
```

しかもアソシエーションも定義できる。

```json
{
    "id": "779c8970-0594-301c-bff5-d12907414002",
    "class": "Author",
    "properties": {
        "name": "Paul Krugman",
        ...
        "writesFor": [
            {
                "beacon": "weaviate://localhost/32d5a368-ace8-3bb7-ade7-9f7ff03eddb6",
                "href": "/v1/objects/32d5a368-ace8-3bb7-ade7-9f7ff03eddb6"
            }
        ]
    },
    "vector": [...]
}
```

これで何が実現できるかというと、
「A さんが投稿している記事に多く登場する単語は？」みたいなデータを跨いだセマンティック検索が可能。
これまでは、構造化データどうすんねんと思ってたけどこれで解決できそう。
