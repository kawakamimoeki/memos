---
title: Dockerのマルチステージビルドを利用してビルド時間を短縮する
date: '2022-11-01'
---

マルチステージビルドの背景にはイメージサイズの削減があります。

> イメージをビルドする際に取り組むことといえば、ほとんどがそのイメージサイズを小さく抑えることです。 Dockerfile 内の各命令は、イメージに対してレイヤーを追加します。 そこで次のレイヤー処理に入る前には、不要となった生成物はクリーンアップしておくことが必要です。 現実に効果的な Dockerfile を書くためには、いつもながらトリッキーなシェルのテクニックや、レイヤーができる限り小さくなるようなロジックを考えたりすることが必要でした。 つまり各レイヤーは、それ以前のレイヤーから受け継ぐべき生成物のみを持ち、他のものは一切持たないようにすることが必要であったわけです。

[マルチステージビルドの利用 — Docker-docs-ja 19.03 ドキュメント](https://docs.docker.jp/engine/userguide/eng-image/multistage-build.html)

ただ、私の場合は、イメージサイズよりもその**ビルド時間の大きさの方**が深刻です。
なぜなら、私がよく使う言語は**スクリプト言語**だからです。

ところが、このマルチステージビルド、うまく利用すればかなりビルド時間の短縮を実現することができるようです。

やってみましょう。簡単な Ruby on Rails と Node.js によるアセット環境を例に、まずはシングルステージの場合から初めて、そこから改善を試みていきます。

```Dockerfile
FROM ruby:3.1
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash - && apt-get install -y nodejs
RUN npm install --global yarn@1.22.19
RUN gem install bundler
RUN mkdir -p /rails
WORKDIR /rails
COPY . .
RUN bundle install
RUN yarn install
RUN RAILS_ENV=production bundle exec rails assets:precompile
CMD ["bin/rails", "s"]
```

最後に怒涛の

```Dockerfile
RUN bundle install
RUN yarn install
RUN RAILS_ENV=production bundle exec rails assets:precompile
```

が行われていますが、ほとんどの場合最も時間がかかるのはここです。
さらに厄介なのは、ソースのいずれかが変更された場合、

```Dockerfile
COPY . .
```

のレイヤーキャッシュが崩れるので、必ず

```Dockerfile
RUN bundle install
RUN yarn install
RUN RAILS_ENV=production bundle exec rails assets:precompile
```

が実行されます。
これはレイヤーキャッシュの仕組みにより必ずそのようになるのです。

> キャッシュが無効になると、次に続く Dockerfile コマンドは新たなイメージを生成し、キャッシュを使いません。

[Dockerfile のベスト・プラクティス — Docker-docs-ja 20.10 ドキュメント](https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html#add-copy)

この問題を解決するために、次は `builder` ステージを用意してみます。

```Dockerfile
FROM ruby:3.1 as builder
WORKDIR /tmp
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash - && apt-get install -y nodejs
RUN npm install --global yarn@1.22.19
RUN gem install bundler
COPY Gemfile Gemfile.lock ./
RUN bundle install
COPY package.json yarn.lock ./
RUN yarn install
COPY app/assets app/assets
COPY app/javascript app/javascript
COPY bin bin
COPY config config
COPY Rakefile vite.config.ts ./
RUN RAILS_ENV=production bundle exec rails assets:precompile

FROM ruby:3.1 as app
RUN mkdir -p /rails
WORKDIR /rails
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --from=builder /tmp/public/vite public/vite
COPY . .
CMD ["bin/rails", "s"]
```

注目すべきは、ここです。

```Dockerfile
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --from=builder /tmp/public/vite public/vite
COPY . .
```

`--from` では他のステージからファイルをコピーすることができますが、**問題は `COPY` の順番**です。
まず、時間のかかる可能性が高い、

```Dockerfile
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --from=builder /tmp/public/vite public/vite
```

が先に記述され、その後

```Dockerfile
COPY . .
```

が実行されます。これで、`Gemfile` `Gemfile.lock` のいずれかが変更されない限り、`bundle install` は実行されません。

ただ、問題は残されていて、
`Gemfile` `Gemfile.lock` のいずれかを変更してしまうと、`yarn install` まで実行されてしまいます。

では、ここで今の所の最終形態に行っていましょう。

```Dockerfile
FROM ruby:3.1 as builder
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash - && apt-get install -y nodejs
RUN npm install --global yarn@1.22.19

FROM ruby:3.1 as bundler
WORKDIR /tmp
RUN gem install bundler
COPY Gemfile Gemfile.lock ./
RUN bundle install

FROM node as yarn
WORKDIR /tmp
COPY package.json yarn.lock ./
RUN yarn install

FROM builder as assets
WORKDIR /tmp
COPY --from=bundler /usr/local/bundle /usr/local/bundle
COPY --from=yarn /tmp/node_modules node_modules
COPY app/assets app/assets
COPY app/javascript app/javascript
COPY bin bin
COPY config config
COPY Rakefile Gemfile Gemfile.lock package.json yarn.lock vite.config.ts ./
RUN RAILS_ENV=production bundle exec rails assets:precompile

FROM ruby:3.1 as app
RUN mkdir -p /rails
WORKDIR /rails
COPY --from=bundler /usr/local/bundle /usr/local/bundle
COPY --from=assets /tmp/public/vite public/vite
COPY . .
CMD ["bin/rails", "s"]
```

👍

この `Dockerfile` では、

- **実行プロセスが依存しているファイルの変更のみを捕捉し、キャッシュを制御している**

を実現しています。これで、例えば `Gemfile.lock` が変更されても `yarn install` は実行されることはありません。

**さらに**

**`yarn` ステージと `bundler` ステージは、並列処理される**ことにお気づきでしょうか。

実行の構造としては、

- `COPY --from=bundler /usr/local/bundle /usr/local/bundle`
  - `RUN bundle install`
- `COPY --from=bundler /usr/local/bundle /usr/local/bundle`
  - `--from=yarn /tmp/node_modules node_modules`
    - `RUN yarn install`

となり、`--from` を解決しつつ、必要な処理を並列で実行することができるようになりました！

以上。

- **マルチステージビルドで、レイヤーキャッシュを最大限に活かすことができる**
- **マルチステージビルドで、並列処理を実現できる**

でした。
