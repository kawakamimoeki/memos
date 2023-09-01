---
title: 'ミニマリストのための管理画面フレームワーク'
date: '2022-11-01'
---

[thoughtbot/administrate](https://github.com/thoughtbot/administrate) は管理画面用のよく知られたフレームワークですが、開発者にとっては使いづらいものでした。私は自分用の Administrate テンプレートファイルを実装しましたが、新しい Administrate のバージョンがリリースされるたびに、**テンプレートファイルが変更されるため、更新が難しくなります。**

しかし、管理画面は共通化が非常に容易なため、フレームワークやライブラリを使用したいと思っています。

そこで、これを実現しました。

[cc-kawakami/micro-admin: A minimal Administration dashboard parts.](https://github.com/cc-kawakami/micro-admin)

私は [trailblazer/cells](https://github.com/trailblazer/cells) を使用して実装したため、Ruby on Rails または他のフレームワークでも使用できます。

`ApplicationDashboard` を定義できます。

```ruby
class ApplicationDashboard < MicroAdmin::Dashboard::Base
    # もし Rails を使うなら
    self.view_paths = [Rails.root.join("app", "views", "dashboards")]
end
```

そして、それを拡張して、モデルごとにダッシュボードクラスを定義し、各画面に表示する属性を定義します。

```ruby
class UserDashboard < ApplicationDashboard
    def index_attributes
        [:id, :name]
    end

    def show_attributes
        [:id, :name]
    end

    def form_attributes
        [:name]
    end

    def model_class
        User
    end
end
```

重要な点として、MicroAdmin にはフォームアイテム用のテンプレートがないため、入力や選択などのテンプレートファイルを書く必要があります。

```html:/app/views/dashboards/user_dashboard/new/name.erb
<input type="text" name="name" class="form-control">
```

`value` 変数は、編集テンプレートに渡されます。

```html:/app/views/dashboards/user_dashboard/edit/name.erb
<input type="text" name="name" value="<%= value %>" class="form-control">
```

以下のように呼び出すことでレンダリングできます。

```ruby
smith = User.new(id: 1, name: "Smith")
james = User.new(id: 1, name: "James")
dashboard = UserDashboard.new
```

```erb
<%= dashboard.index([smith, james]) %>
```

```erb
<%= dashboard.show(smith) %>
```

```erb
<%= dashboard.new(errors: ["Name is required!"]) %>
```

```erb
<%= dashboard.edit(id: 1, values: {name: "Smith"},
```
