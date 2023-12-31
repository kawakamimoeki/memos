はじめに
今回は、UX 改善の取り組みの例を分かりやすいコンバージョンが存在する EC サイトで紹介しようと思います。

対象者
ウェブサイトの設計に関わる人
登場するキーワード
タスク - ユーザーがウェブサイトで実行する行動です。
ステップ - タスクを構成する行動単位です。小さなタスク。例えば、ボタンを押すや、フォームの項目を入力する、など。
目標達成 - 大きなタスク。ユーザーがウェブサイトで最終的に達成したいゴールです。例えば商品の注文など。
目標達成プロセス - 目標達成がどのようなステップで構成されているかを定義したもの。
PV - ページが読み込まれた数。
イベント - ユーザーがウェブサイトで行ったあらゆる行動。
想定する課題
ある EC サイトにおいて、以下のような課題を想定します。

ユーザーにとって嬉しいコンテンツと使いやすい機能を定義できていない
「嬉しい」「使いやすい」の基準が分からない
ユーザーのタスクの完了率を最大化する設計ができていない
タスクとして PV しか定義できていない
完了率として PV しか計測できていない
ユーザーにとって「嬉しい」「使いやすい」を定義する
一般的にユーザビリティの高いタスクは以下のような指標において優秀な成績を収めているとされています。

ここでタスクとは、コンテンツ読了、機能利用などユーザーが目標に到達するために行う行動を指します。

タスクの完了率 - 商品の検索に成功したか？
タスクにかかった時間 - 商品の検索にどのくらい時間がかかったか？
検索の利用率 - そもそもどのくらい検索窓が使われているか？
5 UX KPIs You Need To Track - Every Interaction

目標到達プロセスを設定する
目標達成プロセスは、いくつかのステップによって構成され、ユーザーにどのようにサイトを使ってもらいたいかを定義します。

各ステップがタスクとなりますが、目標達成も大きなタスクと言うことができます。

より詳細にサイトを設計するためのプロセス
ステップはできるだけ詳細に定義してください。

例えば、一般的にウェブサイトの PV 測定がなされたりしますが、PV ではなくあらゆるイベントを単位として考えることで、

サイト構成だけでなく、ページ内のコンテンツ・機能について議論することができるようになります。

Bad：ページビューをステップとする
Good：検索フォームの項目入力をステップとする
例えば、Google Analytics 4 を利用している場合には、この「ステップ」は、PV だけではなく、あらゆるイベント、たとえばスクロールやクリックとして定義できます。

ステップとして利用したいイベントを取得するための方法は後述します。

目標達成プロセスは複数存在しても OK
目標達成プロセスが一通りであるウェブサイトの方が少ないでしょう。

例えば、ユーザーの趣向や時期、広告などによってプロセスは変わることがあります。

あるユーザーは商品を買い物かごに入れて注文する、あるユーザーは商品を返品することが目標達成になる可能性もあります。

その場合は、ユーザータイプの定義や経路の定義を行い、それに対応したプロセスを設定してください。

ステップに従ってページを設計
ユーザーに期待するステップを元に、ページ構成・サイト構成を設計してください。

プロセスが複数ある場合は、そのすべてを満たすような設計にする必要があります。

プロセスのバッティングが起きた場合には、優先度の高い方にとって効果的な設計を採用してください。

これを実践すると、プロセスに影響を与えないコンテンツは必要のないコンテンツということができてしまいます。

しかし、ユーザーの行動パターンをすべて網羅することは不可能ですから、完璧を目指す必要はありません。

ただし、コンテンツを追加する場合には、本当に目標達成にとって効果的なものなのか？という問いを忘れないようにしてください。

例えば、商品を購入するためのウェブサイトなのに、運営会社のブログがずらずらと並んでいても目標達成に影響しないか、妨げになりますよね。

また、

タスクにかかった時間
検索の利用率
の指標に則って、

ステップ数を最小限にする - 検索窓は隠さずに表示しておく
ユーザーの入力はできるだけ補完する - よく注文している商品のキーワードをサジェストする
検索ではなくリンクを用意する - 商品のカテゴリのリンクを並べる
などを意識してください。

ステップの成功率を計測する
実践として Google Analytics 4 の場合には「探索」>「目標達成プロセス」という機能を利用してください。 ここでは、

ステップを定義できる
ステップの完了率/放棄率を見られる
を実現することができます。

また、Google タグマネージャと GA4 を組み合わせて以下のようなイベントを計測することができます。

すべての HTML 要素のクリックイベント
スクロール率
ある HTML 要素の表示
ステップの成功率を向上させる
各ステップの成功率を向上させるような施策を検討してください。

特に成功率の低いステップに重点を置いたり、施策のしやすいところから始めたりすると、何から手を付けてよいか迷うことが少なくなります。

例えば、検索結果画面のスクロール率が低い場合には、商品リストの配置を変更したりすることができます。

おすすめ商品に横スクロールを採用すると商品に特別感が出るかも

プロセス変更の検討
どうしても、ステップの成功率を向上させることが難しいと判断した場合には、プロセスの変更を検討してください。

例えば、どうしても検索結果から商品詳細への遷移率が上がらない場合には、検索画面に人気商品へのリンクを並べてしまいショートカットしてしまう手もあるかもしれません。

例えば特集のバナーを載せたり

まとめ
ウェブサイトにおける UX 改善の実践方法についてご紹介しましたが、Google Analytics などを使った測定方法のイメージがなかなか湧きにくいかもしれません。

次回以降別の記事でご紹介したいと思います。
