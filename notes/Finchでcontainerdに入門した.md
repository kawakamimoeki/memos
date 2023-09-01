---
title: 'Finchでcontainerdに入門した'
date: '2022-12-02'
---

AWS が Finch を発表した。

[コンテナ開発用のオープンソースクライアント「Finch」のご紹介 | Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/introducing-finch-an-open-source-client-for-container-development/)

筆者は Mac を使っていて、Docker Desktop または、[colima](https://github.com/abiosoft/colima)　 x 　 Docker CLI を利用して、Docker の開発環境を実現している。Dcoker Desktop は内部で[HyperKit](https://github.com/moby/hyperkit)（macOS hypervisor）を利用して Linux の VM を立ち上げ、その中で dockerd を実行している。Docker 導入当初は Docker Desktop を利用していたが、Docker Desktop の GUI は使っていなかったので、よりシンプルな colima に移行した。colima は Lima をベースに

- Lima の設定ファイルを生成して Linux 環境を準備
- Linux on Lima の環境に Docker をインストール
- ホスト側(つまり macOS 側)の Docker CLI と Docker on Linux on Lima を透過的に接続できる環境を用意

ということを行なって、Docker 開発のサポートをしてくれる。この時点で、Lima の存在は認知していたが、AWS の記事を見ると Lima 自体結構盛り上がっているようだ。

今回の Finch の発表、つまり Alternative Docker の参入の背景はどんなものがあるのだろうか？正直言うと Kubernetes は実際に利用したことがなく疎いため、Kunernetes が Docker をやめた話を今回の Finch をきっかけに以下の記事を見つけ始めて認識した。

参考：[Docker から containerd への移行 (NTT Tech Conference 2022 発表レポート) | by Akihiro Suda | nttlabs | Medium](https://medium.com/nttlabs/docker-to-containerd-4f3a56e6f2b6)

詳細を見てみると、従来、Kubernetes は分散環境上のコンテナ管理のために Docker を触っていたが、近年は[containerd](https://containerd.io/)を直接利用するようになったらしい。

![](https://miro.medium.com/max/1400/1*HL7tRfSRwv8fLleLRDvDXQ.webp)
https://speakerdeck.com/ktock/dockerkaracontainerdhefalseyi-xing?slide=7

containerd を直接利用？containerd？コンテナ技術何も分かっていなかったことが分かった。
containerd は元々、Docker の下でコンテナの基本的な管理機能を提供するデーモンとして、2015 年に Docker 社により開発された。containerd のスコープは徐々に拡大して、現在では、Docker Engine のほぼ全てをカバーしているようだ。例えば、[nerdctl](https://github.com/containerd/nerdctl)は containerd の CLI だ。UX は Docker CLI とほぼ同じで、Docker Compose も対応している（`nerdctl compose`）。

Kubernetes に話を戻すと CHANGELOG によれば、dockershim という Docker の CRI（Container Runtime Interface）　のメンテナンス上の問題が起きていた模様。

> Docker support in the kubelet is now deprecated and will be removed in a future release. The kubelet uses a module called "dockershim" which implements CRI support for Docker and it has seen maintenance issues in the Kubernetes community. We encourage you to evaluate moving to a container runtime that is a full-fledged implementation of CRI (v1alpha1 or v1 compliant) as they become available.

[kubernetes/CHANGELOG-1.20.md at master · kubernetes/kubernetes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#deprecation)

整理すると、containerd が Docker から生まれ派生し、独自に領域を増やすと同時に、Kubernetes のメンテナンス上の問題によって Docker ではなく containerd が選ばれ、containerd を直接管理するための方法も理由も揃ってきたということになるか。そして、これらを背景にして、Finch の登場となった。Finch は containerd, nerdctl と Lima を組み合わせたツールだ。Finch 登場以前は、containerd を利用する場合は、Lima を直接利用する場合がほとんどだっただろう。

さらに Finch の記事には[BuildKit](https://github.com/moby/buildkit)についても触れられているのでこちらも抑えておこう。これまでと同様に Docker との関係性を中心に理解していこうと思う。私も身に覚えがあって、`DOCKER_BUILDKIT=1 docker build .`のようなものを目にする機会があった。これは Docker で BuildKit を利用する方法だ。BuildKit には Docker デフォルトのビルドに対して以下のようなメリットがあるようだ。

- 複数ステージのビルドを並列実行できる
- 機密情報をビルド成果物(キャッシュを含む)に残さない仕組み(Dockerfile 内の RUN --mount コマンド)が使える
- ローカルファイルをビルド時のみに使える
- リモートファイルをビルド時に SSH 接続して取得できる
- ビルドキャッシュのインポート/エクスポートが出来る
- 分散ビルドが出来る

なるほど。ビルドキャッシュや並列実行は強力そうだ。Finch ではこれらの機能をデフォルトで利用できるという認識で良いかと思う。

Finch の意義を理解できたところで、実際に使ってみよう。

```
brew install finch
```

まず、finch の構成を設定することをしてみた。README に沿って以下のように設定してみた。

```yaml:~/.finch/finch.yaml
cpus: 8
memory: 8GiB
```

そして vm を初期化する。

```
finch vm init
```

これで vm が立ち上がった。
次は Docker CLI や nerdctl と同じように、Docker Compose を利用してみる。

```
finch compose up
```

ここで、Docker と containerd との挙動の違いにいくつか出会ったので紹介しておく。

まず、port の省略的な記述が無効だった。

```diff:compose.yaml
  db:
    ports:
-     - 5432
+     - 5432:5432
```

あとは、context の外のファイルを読み込もうとするとエラーになった。今回はファイルの置き場所を変えることで対応した。

```diff:compose.yaml
  app:
     volumes:
-      - ~/.gitconfig:/root/.gitconfig
+      - ./.gitconfig:/root/.gitconfig
```

それ以外は、Docker の場合と同じだった。

以上、Finch をきっかけに containerd やコンテナ周辺の技術についての理解を深めることができた。今後は徐々に Finch への移行を進めていった方が潮流に乗ることができるような気がしている。
