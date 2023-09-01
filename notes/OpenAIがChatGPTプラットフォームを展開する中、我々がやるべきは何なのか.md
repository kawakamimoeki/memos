---
title: OpenAI が ChatGPT プラットフォームを展開する中、我々がやるべきは何なのか
date: "2023-04-28"
---

OpenAI は ChatGPT API をリリースして以来、続々と派生サービスを展開しています。なぜリリースして以来というのを強調したかというと、悪意があるように感じてしまうくらいのタイミングだと思うからです。つまり、そのタイミングとは、API を利用してプロダクトを開発している人たちがアイデアを形にしようとしているタイミングだということです。

ここで一旦 OpenAI の動きを振り返ってみましょう。

参考:

[hollobit/GenAI_LLM_timeline: ChatGPT, GenerativeAI and LLMs Timeline](https://github.com/hollobit/GenAI_LLM_timeline)

### 2022-11-30: ChatGPT リリース

このタイミングで、OpenAI がどこまで想定していたのか分かりません。ユーザーがプロンプトという概念に慣れていく序章に過ぎないという感じだったのでしょうか。

### 2023-03-01: ChatGPT API リリース

このリリースのタイミングは絶妙だったように記憶しています。人々が LLM の概要を理解し、アイデアを溜めていた状態でした。リリース後 1 週間は API を利用したサービスがたくさんリリースされましたが、傾向としては、

- 既存プラットフォームでの利用
- 特定のユースケースのプロンプトを GUI に
- PDF などのデータを入力としてチャットボットを生成

あたりで、まだまだシンプルなものが多かったと思います。

### 2023-03-14: OpenAI の利用規約更新

[Usage policies](https://openai.com/policies/usage-policies)

ここでは、ChatGPT API を利用したやり取りは学習に利用しないということが明言されました。ここでさらに API を利用したプロダクトが一気に増えた印象がありました。

### 2023-03-23: ChatGPT plugins 発表

[ChatGPT plugins](https://openai.com/blog/chatgpt-plugins)

ChatGPT の GUI から外部サービスとの連携を実現できる plugins が発表されました。次世代のモバイルアプリになるのではないかという予想を立てていましたが、まだ利用には制限があり、どうなるか分かりません。

ただし、外部サービス x プロンプト UI のプロダクトについては、この plugins で完結できることは間違いなく、開発者の工数が減り、それと同時に ChatGPT 本体にエンゲージメントを取られる形となると思われます。

また、プロンプト型の UI で実現できるものは plugins に吸収されることは明らかだと思います。

### Microsoft や Google が LLM を利用したサービスを発表

この辺りでは、業務特化型の既存サービスは Microsoft Office や Google Workspace で完結できる傾向があることが判明しました。

### 2023-04-26: OpenAI Brand guidelines

[Brand guidelines](https://openai.com/brand)

ここでは、〇〇 GPT というプロダクト名を推奨しないことが明言され、今後は ChatGPT を利用したサービスの存在感がこれまでよりも弱くなる傾向が予想されます。

### 2023-04-27: ChatGPT Business 発表

[New ways to manage your data in ChatGPT](https://openai.com/blog/new-ways-to-manage-your-data-in-chatgpt)

ここでは、ChatGPT の Web UI でも企業のデータを学習に使わないプランを考えていることが発表されました。つまり、ChatGPT API のポリシーがそのまま適用されます。これで、企業向けに ChatGPT の導入を進めていたプロダクトにとって不利な状況になったと思われます。

## OpenAI の考えの考察

OpenAI は ChatGPT の Web UI をプラットフォームとして展開することがユーザーにとってより良い体験につながるという考えを持っていると感じています。これは、プロダクトを開発している立場としてはとても納得できます。ユーザーにとってサイトを行き来したり、サービスを使い分けることがいいことだとは考えにくいです。

我々（開発者）は何をするべきか？

これまでの流れと今後の展開の予想から以下のことに注力するべきだと思っています。

1. これまでと同様に、ChatGPT を主体としないプロダクトを発想する（AI 時代に必要なものとは何なのか？）
2. 既存プロダクトと LLM との連携について考える
3. ChatGPT plugins を開発する
4. ChatGPT 単体もしくは、既存プラットフォーム（Microsoft / Google）では解決できない「UI」を考える

4 で強調した「UI」とは、つまり、プロンプト UI でできてしまうことは、plugins に吸収されてしまうからです。

以上です。補足などがあればよろしくお願いします。
