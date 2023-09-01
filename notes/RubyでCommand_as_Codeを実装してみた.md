---
title: Rubyで"Command as Code"を実装してみた
date: '2022-10-28'
---

こちらです。[cc-kawakami/shufu: Shufu is a simple Command as Code](https://github.com/cc-kawakami/shufu)

コマンドの引数やオプションの構成を定義しておいておきたい時があって、そのときに書いた気がします。
たしか、`gcloud` コマンドを YAML として定義することで、簡易的な Infrastructure as Code を実現しようとしたときに導出しました。

詳しく見ていきましょう。

基本はこうです。

```ruby
require "shufu"

# コマンドのスキーマの定義
schema = {
  git: {
    commit: {
      amend: :flag,
      author: :equal
    }
  }
}

# スキーマからコマンドのオブジェクトを生成
command = Shufu::Command.new(schema)

# to_s にスキーマに対応した値を渡すことで string になります！
p command.to_s({ amend: true, author: "cc-kawakami" })

# => "git commit --amend --author=cc-kawakami"
```

ここから発展して、例えば、`gcloud alpha storage buckets create` コマンドのスキーマを定義して、

```yaml
gcloud:
  alpha:
    storage:
      buckets:
        create:
          name: :param
          class: :equal
          bucket-level: :equal
          location: :equal
```

それに対する値も定義しておいて、

```yaml
name: foobarbox
class: regional
bucket-level: 'on'
location: us-central1
```

Shufu を使ってください。

```ruby
schema = YAML.load_file(schema_filepath)
values = YAML.load_file(value_filepath).map { |k, v| [k.to_sym, v] }.to_h
Shufu::Command.new(schema).to_s(values)
```

そうすれば、簡易的な IaC の完成です！
