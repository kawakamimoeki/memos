---
title: GitBookが楽しかった
date: "2023-05-01"
---

https://guide.uxai.dev で [GitBook](https://www.gitbook.com/) を使ってみました。

構成としては、

https://github.com/moekidev/ai-conversational-ux-guide/tree/main/guides

```
guides
├── en
└── ja
```

です。

GitBook には、team, collection と space と言う階層があって、

- team
  - collection
    - spaceA
    - spcaeB
    - ...

みたいな感じになります。

今回は、

- UXAI.DEV
  - AI Conversational UX Guide => `guide.uxai.dev`
    - en => `/v/en`
    - ja => `v/ja`

みたいな構成にしました。

team, collection, space どれにも custom domain が設定できてその辺りがややこしかったです。

以上。