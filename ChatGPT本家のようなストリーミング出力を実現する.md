---
title: ChatGPT 本家のようなストリーミング出力を実現する
date: '2023-04-28'
---

今回の例では、Langchain を利用してますが、多分本家の API だけでも対応できると思います。

[Welcome to LangChain | 🦜️🔗 Langchain](https://js.langchain.com/docs/)

まずはサーバーサイド。

```typescript
import { ChatOpenAI } from "langchain/chat_models"
import { HumanChatMessage } from "langchain/schema"
import { CallbackManager } from "langchain/callbacks"

const api = (req, res, ctx) => {
  if (req.method !== "POST") return
  const input = req.body.input

  const chat = new ChatOpenAI({
    openAIApiKey: OPENAI_API_KEY,
    streaming: true
    callbackManager: CallbackManager.fromHandlers({
      async handleLLMNewToken(token: string) {
        res.write(token)
      },
    }),
  })

  await chat.call([new HumanChatMessage(input)])

  res.end()
})
```

ポイントは、`streaming: true`でストリーミングを有効にして、[`write()`](https://nodejs.org/api/stream.html#writable-streams) でストリーミングを有効にします。これは Node.js の Writable streams というもので、ファイルやレスポンスに対して有効になります。

全部 Langchain の処理が終わったら `end()` で終了してあげます。

次はクライアントサイドです。

```typescript
const reply = async (input) => {
  const decoder = new TextDecoder()

  const res = await fetch(ENDPOINT, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      input
    })
  })
  const reader = res?.body?.getReader()

  let output = ''

  while (true) {
    if (reader) {
      const { value, done } = await reader.read()
      if (done) break
      output += decoder.decode(value)
    }
  }
}
```

ポイントは [`getReader()`](https://developer.mozilla.org/ja/docs/Web/API/ReadableStream/getReader)ですね。あと、decode しないといけないのでその処理を入れておきます。

これで OK。
