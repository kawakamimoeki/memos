俺　　「Rails アプリ**速く**してえけど Heroku で**ラク**してえしなあ...。」
Fastly 「やあ」
俺　　「キャッシュなんかしたら**アプリが更新されなくなっちゃう**よ」
??? 　「HTML を**分解**するんじゃ」

---

↓ 先人の方勉強になります 🙏

https://dev.to/ben/making-devto-insanely-fast

![](https://storage.googleapis.com/zenn-user-upload/fc2d822e93e6-20230824.png)

## 0. Fastly のセットアップ

Heroku の場合は Fastly のアドオンが使えます。基本的に Heroku の紹介ページに従えば OK。

https://devcenter.heroku.com/articles/fastly

## 1. HTML を static/dynamic に分解する

### Static

ページにはアプリの DB との関連性がなく、**リクエストごとに違いが生じない静的なパーツ**があるはずです。これらを**CDN にキャッシュ**します。そうすれば最初のレスポンスを最速で返せるはず。ページはほとんどからかもしれないけど何かしらは表示できるので意外とストレスが少ない。リンクを押したときの反応もかなり向上させられる。例えば。

- ヘッダーにあるロゴはリクエストごとに変わらないはず
- フッターにあるリンクはリクエストごとに変わらないはず
- ナビゲーションメニューはリクエストごとに変わらないはず
- head で読み込んでいるメタタグやアセットは変わらないはず

### Dynamic

DB からデータをとってこないといけない場所や、ログインしてるかどうかで表示が変わる場所はキャッシュしてしまうと、正しく表示できなくなってしまう。例えば。

- ナビゲーションメニューのリストはログインした人としてない人で変わるかも
- 投稿されたつぶやきは随時更新してほしいかも
- 通知はリクエストごとに更新されて欲しい

![](https://storage.googleapis.com/zenn-user-upload/d6bb4ed00f7e-20230824.png)

### どうやって分割する？

Rails の場合は、[Turbo](https://turbo.hotwired.dev/)がいいかも。

```html:app/views/home/index.html.erb
<%= turbo_frame_tag :posts, src: posts_path %>
```

```html:app/views/posts/index.html.erb
<%= turbo_frame_tag :posts do %>
  <%= render @posts %>
<% end %>
```

こんな感じで、posts の部分だけ後からフェッチするようにできる。

### 補足

Fastly を通して Websocket を利用する場合には課金が必要。まあ別に**Fastly を通さなくても、Action Cable などの設定で、Heroku と直接やり取りできれば OK**。

```ruby
config.action_cable.url = "wss://example.herokuapp.com/cable"
config.action_cable.allowed_request_origins = [ "https://yourdomain.com" ]
```

## 2. キャッシュの設定

基本的に以下のような感じで、**Fastly にキャッシュの旨を伝えられれば**OK。

```
Cache-Control: public, no-cache
Surrogate-Control: max-age=2592000
```

具体的な実装だとこんな感じかな？気をつけないと、Turbo Stream の分をキャッシュしてしまうのでリクエストのフォーマットを確認する。

```ruby
def index
  if request.format == "text/html" # Turbo Streamの分とかまでキャッシュしないように
    set_cache_control_headers
  end

  # ...
end

private def set_cache_control_headers(max_age = 2592000)
  request.session_options[:skip] = true
  response.headers['Cache-Control'] = "public, no-cache"
  response.headers['Surrogate-Control'] = "max-age=#{max_age}"
end
```

## 3. デプロイごとにキャッシュをパージする

静的な部分の HTML を変更してデプロイしても、**今のままだと Fastly さんがキャッシュしたままなのでページが更新されません**。そこで、デプロイ時に、Fastly のキャッシュをパージすることにした。

ここでは、Fastly の Ruby クライアントを使う。

https://github.com/fastly/fastly-ruby

```ruby
desc "Fastly cache purge"
task purege_cache: :environment do
  api_instance = Fastly::PurgeApi.new
  opts = {
    service_id: ENV['FASTLY_SERVICE_ID'],
    fastly_soft_purge: 1
  }

  begin
    result = api_instance.purge_all(opts)
  rescue Fastly::ApiError => e
    Rails.logger.error("Exception when calling PurgeApi->purge_all: #{e}")
  end
end
```

Heroku だったら、このタスクを**リリースコマンド**に設定する。

```procfile:Procfile
release: bundle exec rake purge_cache # これ
web: bundle exec puma -C config/puma.rb
```

## 終わり

こちらのアプリで挙動を確かめられます ↓

https://www.timesy.dev/
