---
title: Spiral of decentralized and centralized
date: '2022-11-26'
---

この記事は、以下の Podcast を要約したものです。

[29. The Aesthetic of Technology Selection (2) w/ twada | fukabori.fm](https://fukabori.fm/episode/29)

この Podcast では、コンピューティングが発展してきて、分散コンピューティングと集中コンピューティングのトレンドが来たり行ったりしていること、そして、この広い歴史的なトレンドを要約している。

著者は、2019 年以降業界に関わっているという前提で話をしている。

この記事では、用語をより詳しく見ていく。

## Good Old Web

最初に、ビジネスシステムなどが[Visual Basic](https://ja.wikipedia.org/wiki/Visual_Basic)と SQL サーバーを使用していることについて話している。これは、2000 年頃の話で、画面と SQL だけの 2 層アーキテクチャーであると説明されている。これは、[Next.js](https://nextjs.org/)の開発体験に似ているように感じた。

次に、@t*wada 氏がモノリシックな[LAMP(Linux, Apache, MySQL, PHP)](<https://en.wikipedia.org/wiki/LAMP*(software_bundle)>)の不動産検索サービスを開発したことが聞かれた。PHP でレンダリングされた HTML を表示するだけのブラウザが行っていた単純な作業の時代であり、Java Servlet なども主流であった。これは 2002 年頃である。

ここで、Visual Basic と SQL の 2 層アーキテクチャから LAMP の 3 層アーキテクチャへの移行があった。これは、集中コンピューティングから分散コンピューティングへの流れの始まりである。そして、分散コンピューティングへのトレンドは次のセクションでさらに強化された。

## EJB や SOAP を使った分散コンピューティング

ビジネスシステムは複雑で再利用可能な部品が多く、複数のシステムを接続する需要がありました。

このような問題を解決する文脈で、ビジネスロジックを実装するコンポーネントである[EJB（Enterprise Java Beans）](https://ja.wikipedia.org/wiki/Enterprise_JavaBeans)が登場しました。

[SOAP（Simple Object Access Protocol）](<https://ja.wikipedia.org/wiki/SOAP_(%E3%83%97%E3%83%AD%E3%83%88%E3%82%B3%E3%83%AB)>)は、[XML](https://ja.wikipedia.org/wiki/Extensible_Markup_Language)ベースの[RPC](https://ja.wikipedia.org/wiki/%E9%81%A0%E9%9A%94%E6%89%8B%E7%B6%9A%E3%81%8D%E5%91%BC%E5%87%BA%E3%81%97)プロトコルの実装です。異なる言語やオペレーティングシステムで実行されるプロセスが、XML を介して通信することができます。仕様は次第に単純でなくなり、SOAP は Simple Oriented Access Protocol の略称ではないという興味深い声明があったとのことです。

これらのアイデアは、[SOA（Service-Oriented SOA（Service-Oriented Architecture）](https://ja.wikipedia.org/wiki/%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E6%8C%87%E5%90%91%E3%82%A2%E3%83%BC%E3%82%AD%E3%83%86%E3%82%AF%E3%83%81%E3%83%A3)に基づいています。SOA とは、単一のビジネスプロセスに対応するソフトウェア機能をサービスとして考慮し、これらのサービスをネットワーク上でリンクし、全体のシステムを構築することです。これはクラウドコンピューティングの台頭で再確認されました。具体的には、ソフトウェアを単一のビジネスプロセスに相当するユニットで構成し、オープンで標準化された技術仕様を使用してサービスインタフェースを定義する場合、SOA と見なされます。

[What is Service Oriented Architecture? Service Oriented Architecture Explained - AWS](https://aws.amazon.com/jp/what-is/service-oriented-architecture/)

## Ruby on Rails のモノリシックへの移行がもたらした影響

また、EJB や SOAP に基づく SOA の問題点は、様々なシステムを接続するために大量の設定を書かなければならないことであったが、Ruby on Rails はその反対を提唱していた点が注目されています。

実際、Ruby on Rails 以前から [Spring Framework](https://ja.wikipedia.org/wiki/Spring_Framework) や [Seasar](https://ja.wikipedia.org/wiki/Seasar) などのフレームワークがありましたが、これらのフレームワークも Ruby on Rails の登場後は影響を受け、慣例に基づいた設計がなされるようになりました。

## ウェブを基盤とする分散システム

Ruby on Rails のようなモノリシックなフレームワークは、分散システムは実装が困難であるという考え方から生まれました。しかし、身の回りに成功している分散システムがあることに気づきました。それが実際には World Wide Web でした。

分散システムの設計は、RPC から Web ベースに移行し始めました。JSON を使用したサーバー-クライアントおよびサーバー-サーバーの相互作用のアーキテクチャが増加しました。さらに、クラウドコンピューティング、コンテナ技術、継続的デリバリーの登場により、マイクロサービスなどの分散システムへの動きが再び現れています。

さらに、[GraphQL](https://graphql.org/) や [gRPC](https://grpc.io/) に象徴される、サーバー駆動型 API 定義からクライアント駆動型 API 定義へのシフトは、マイクロサービスの実現を強化しました。この点から見ると、GraphQL はサーバー側とクライアント側の合意不足を強調しています。

## 感想

知らないことが多く、継続的に進化する技術に対する印象を明確にしたと思います。今後、他の視点からも学んでいきたいと思います。
