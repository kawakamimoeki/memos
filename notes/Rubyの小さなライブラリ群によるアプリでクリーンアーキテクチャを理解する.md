---
title: Rubyの小さなライブラリ群によるアプリでクリーンアーキテクチャを理解する
date: '2022-11-01'
---

クリーンアーキテクチャですが、[hanami/hanami: The web, with simplicity.](https://github.com/hanami/hanami) と出会ってから 5 周回くらいして、ようやく腹落ちしつつあるのでまとめてみます。

#### フレームワークを利用しつつ理解するのは難しい

フレームワークを利用した場合、例の図の Usecase の外堀をグルッと埋める形になります。MVC なら Controller、View、Model です。Hanami の場合は Repository が来ます。個人的にはこの内側から 3 層目が全て一体化していると、どうしてもそれらが中心であるように感じてしまうんですよね。

そこでアプリケーションのそれぞれの責務を別のライブラリを利用して実装することを試してみました。

これです。 [cc-kawakami/clean-architecture-minimal-app: A minimal Clean Architecture app](https://github.com/cc-kawakami/clean-architecture-minimal-app)

利用したライブラリはこれらです。

- Usecase: [utils/interactor.rb at v1.3.8 · hanami/utils](https://github.com/hanami/utils/blob/v1.3.8/lib/hanami/interactor.rb)
- Controller: [sinatra/sinatra: Classy web-development dressed in a DSL (official / canonical repo)](https://github.com/sinatra/sinatra)
- Serializer: [procore/blueprinter: Simple, Fast, and Declarative Serialization Library for Ruby](https://github.com/procore/blueprinter)
- Object Mapper: [rom-rb/rom: Data mapping and persistence toolkit for Ruby](https://github.com/rom-rb/rom)
- DB: [sparklemotion/sqlite3-ruby: Ruby bindings for the SQLite3 embedded database](https://github.com/sparklemotion/sqlite3-ruby)

#### 「Entity あれ。」

アーキテクチャの中心から攻めていきます。まずは Entity。

```ruby
class User
  attr_reader :id, :name, :email

  def initialize(id:, name:, email:)
    @id = id
    @name = name
    @email = email
  end
end
```

普通のクラスですね。ビジネス上でどんな情報を扱うかを定義しています。今回はユーザーを管理することを想定して User entitiy を用意しました。

#### 次にアプリのビジネスロジック

このアプリではユーザーを ID で検索できるようにします。

```ruby
class FindUser
  include Hanami::Interactor

  expose :user

  def initialize(repository:, serializer:)
    @repository = repository
    @serializer = serializer
  end

  def call(id)
    begin
      #ユーザーを探す
    rescue => e
      error!(e.class.name)
    end
  end
end
```

こんな感じの Usecase を用意しました。これがアプリがビジネス上のどんな目的を果たすかを定義できました。Repository, Serializer は Entity をとってくるところと出力するところの Adapter ですね。これを Usecase が利用します。

#### あとは Adapter を拵えていく

Repository。

```ruby
class UserRepository < ROM::Repository[:users]
  commands :create

  def find(id)
    users.by_pk(id).map_to(User).one
  end
end
```

Controller。

```ruby
get "/users/:id" do
  find = FindUser.new.call(params["id"])

  if find.success?
    if find.user
      status 200
      body = find.user
    else
      status 404
      body = { error: "Not found!" }
    end
  else
    status 500
    body = { error: find.error }
  end

  json body
end
```

Serializer です。

```ruby
class UserSerializer < Blueprinter::Base
  identifier :id

  fields :name, :email
end
```

これで揃いました。

あとは、さっき出てきた Usecase の詳細な記述をしていきます。

```ruby
class FindUser
  include Hanami::Interactor

  expose :user

  def initialize(
    repository: UserRepository.new(App.new.rom),
    serializer: UserSerializer
  )
    @repository = repository
    @serializer = serializer
  end

  def call(id)
    begin
      user = @repository.find(id)
      @user = @serializer.render_as_hash(user)
    rescue => e
      error!(e.class.name)
    end
  end
end
```

#### 実行する

```bash
$ curl http://127.0.0.1:9292/users/1
{"id":1,"email":"smith@exmaple.com","name":"Smith"}
```

OK。ここまでの流れで、HTTP リクエスト・レスポンスは、FindUser のビジネスロジックの**入出力の詳細の一つ**に過ぎないことが体感できましたでしょうか。
同じ様に、DB は User を生成するための**実装の一つ**でしかありません。**DB や HTTP は詳細**なのです。

#### 責務を混ぜることが難しい = 責務が混ざらない

この設計なら、Controller に Entity の検索が記述されることは、よほど工夫しない限り起こらないということが分かるでしょう。DB や HTTP、HTML、JSON の詳細は Usecase を起点として、円のそれぞれの方向へ分散される、という設計に自然と誘導されます。

以上です。ありがとうございました。
