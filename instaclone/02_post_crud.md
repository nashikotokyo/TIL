# 02. 投稿のCRUD機能を実装
### 内容
投稿のCRUD機能を作ってください  
  
### 補足
- ユーザーとポストのシードファイルを作る  
- その際fakerを使ってダミーテキストを生成する  
- 画像のアップロードにはcarrierwaveを使用する  
- image_magickを使用して、画像は横幅or縦幅が最大1000pxとなるようにリサイズする  
- 画像は複数枚アップロードできるようにする  
- Swiper使って画像をスワイプできるようにする
- 諸々のアイコンにはfontawesomeを使用する
  
# 作業手順
- swiperの導入
- fontawesome導入
- carrierwaveの導入とuploadファイルをgit管理対象外に設定(gitignore)
- fakerの導入
- seedファイルの作成
- ヘッダ部分にfontawesomeのアイコンを表示
- application.htmlのmetaタグに文字コードの設定とレスポンシブ対応
- routes.rbにpostsのルーティングを追加、annotateの更新
- postテーブル(マイグレーションファイル)の作成(imagesカラム(JSON型で複数画像対応), textカラム, 外部キーのカラムの設定、外部キー制約やreference型、indexについて確認)
- postモデルの設定(belongs_to :userやバリデーション)
- userモデルの方にもアソシエーション設定(has_many :posts, dependent: :destroy)
- carrierwaveの設定(アップローダークラス設定、複数画像を扱う設定など)
- postコントローラとビューの作成(indexアクションとindex.html、_post.html、new、createアクションとnew.html、_form.html、edit updateアクションとedit.html(と_form.html)、showアクションとshow.html、destroyアクション)
- swiperの設定(application.js, application.scss上の設定、swiper.jsの作成、_post.html, show.htmlの修正など)
- ルートパスやログイン後、ログアウト後のリダイレクト先を変更、ヘッダ部分のリンク設定を修正など
  
# 学んだこと  
## seedファイルとFaker
ユーザを生成するseed内でFakerを使用するときユニークなデータを生成したい場合は
```
10.times do
  user = User.create!(
    email: Faker::Internet.unique.email,
    username: Faker::Internet.unique.user_name,
    password: 'password',
    password_confirmation: 'password'
  )
```
のようにuniqueをつける  
- シードファイルdb/seeds/posts.rb内の、画像のアップロード部分で、carrierwaveでリモートロケーション(ネット上のurl)から画像をアップロードする方法↓  
carrierwaveではマウントしたカラムがstring型(1枚)なら`remote_[マウントしたカラム名]_url`というメソッド(属性)が、マウントしたカラム名がarray型(複数枚)なら`remote_[マウントしたカラム名]_urls`というメソッド(属性)が使えるようになる。そこにダウンロードしたい画像URLを指定してsaveするとアップロード処理が実行される  
https://blog.hatappi.me/entry/2018/04/05/231828  
```
# db/seeds/posts.rb

puts 'Start inserting seed "posts" ...'
User.limit(10).each do |user|
  post = user.posts.create!(
    body: Faker::Hacker.say_something_smart,
    remote_images_urls: %w[https://picsum.photos/350/350/?random https://picsum.photos/350/350/?random]
```
=> 今回imagesカラムはarray型なのでremote_images_urlsを使う、ここにダミー画像のurlを格納することでリモートからダウンロードできる。  
ダミー画像urlは`Lorem Picsum`を使用している。  
   
## 外部キー制約について
1. 存在しない値を外部キーとして登録することはできない
2. 子テーブルの外部キーに値が登録されている親テーブルのレコードは削除できない
3. 外部キー制約をつける場合、インデックスは自動で付与されるので、index: trueは不要になる
※つけるには`add_foreign_key :対象のテーブル名, :指定先のテーブル` と書く  
もしくは`reference型`で`foreign_key: true`と書く。  
  
## reference型を使うメリット
2つのメリットがある  
1. userではなくuser_idというカラム名を作成してくれる
2. インデックスを自動で張ってくれる
※t.reference :userだけでは外部キー制約はつかないので`t.references :user, foreign_key: true`のように書く必要がある  
https://qiita.com/ryouzi/items/2682e7e8a86fd2b1ae47  
  
## dependent: :destroyとは
- 親モデルを削除する際に、その親モデルに紐づく「子モデル」も一緒に削除できるようになる。  
  
- `dependent: :destroyとforeign_key`について  
=>基本的にアソシエーションで外部キーにはforeign_keyを貼るので、親モデルを参照している小モデルがあれば、子モデルに親モデルの外部キーが残ってる限り削除できないというエラーが起きる。(外部キー制約)  
これを防ぐ為に先に親モデルの外部キーidを参照している子モデルのレコードを全て削除してから親モデルのレコードを削除する`dependent: :destroy`が必要になる。  
https://qiita.com/Tsh-43879562/items/fbc968453a7063776637  
  
## CarrierWaveで複数画像を扱う方法
CarrierWaveで複数画像を扱う場合..  
  
- 画像情報を格納するカラム名を `複数形`にする。(image => images)
  
- データベースがjsonのデータ型をサポートしている場合（PostgreSQL,&nbsp;MySQLなど）はデータ型を`JSON`にする。
 ```
# マイグレーションファイル
  def change
    create_table :posts do |t|
      t.json :images, null: false
      t.text :body, null: false
      t.references :user, foreign_key: true
      
      t.timestamps
  end
```
データベースがjsonのデータ型をサポートしていない場合（SQLiteなど）はカラムをstring型にし、モデル側で`serialize :images, JSON`と記載する。(MySqlはこの方法でもOKだった。）  
```
# Postモデル

class Post < ApplicationRecord
  belongs_to :user
  mount_uploaders :images, PostImageUploader
  serialize :images, JSON
  validates :images, presence: true
  validates :body, presence: true, length: {maximum: 1000}
end
```
  
- モデルとアップローダーとの紐付けの設定の際、モデルに `mount_uploaders(複数形) :images(カラム名複数形), PostImageUploader(アップローダー名)`のように複数形にして記載する。 
  
- コントローラ上のストロングパラメータも画像の部分は`params.require(:post).permit(:body, images: [])`のように`空の配列`を記載する。
  
- ビュー上の画像を選択するフィールドには`multiple: true`で複数選択できるようにする。
  
```
(例)
= f.file_field :images, multiple: true, class: 'form-control'
```
https://qiita.com/tanutanu/items/47f8a229ef52cae3c251  
  
## 部分テンプレートについて
復習ポイント  
- `partialオプション`は「呼び出しているのは部分テンプレートだよ」と強調したいだけなので、つけなくても構わない。  
しかしlocalsオプションを使用した時はつけないとエラーが出る。partialを省略して書きたければ下記のように必ずlocalsオプションも省略して書く必要あり。  
```
<%= render 'hoge', hoge: hoge %>

<!-- 下記はlocalsオプションを記述しているのでエラー -->
<%= render 'hoge', locals: { hoge: hoge } %>
```
  
- `collectionオプション`  
部分テンプレートを繰り返しrenderしたい場合（indexでの投稿の一覧表示など）以下のように書いても間違いではない。  
しかし、これだと@hogesが持つ要素の個数分renderメソッドが実行されて、レンダリング(パーシャルファイルが実行される)されるのでパフォーマンスが悪くなる。  
```
<% @hoges.each do |hoge| %>
  <%= render partial: 'hoge', hoge: hoge %>
<% end %>
```
上記の3行はcollectionオプションを使用し下記1行に書き換え可能。collectionオプションに指定した変数の要素の分だけ部分テンプレートが繰り返し表示されるという結果も同じとなる。  
```
<%= render partial: 'hoge', collection: @hoges %>
```
  
- `パフォーマンスの問題`に関して  
上記のようにcollectionオプションを使用して記述する方法とeachで回す方法ではパフォーマンスに違いがある。  
  
**collectionオプション**  
=> 部分テンプレートがrenderで呼び出され、読み込むのが1回のみで済む。  
  
**eachで回す**  
=> @hogesの個数分部分テンプレートをrenderで呼び出し読み込むのでパフォーマンスが悪い。  
`※collectionオプションを使用する時はpartial:を記述しないとエラーになるので気をつける！`  
  
- 下の3つの条件を全て満たしている時、下記の書き方は省略できる。  
```
<%= render partial: 'hoge', collection: @hoges %>
```
↓　省略可能  
```
<%= render @hoges %>
```
1. 呼び出す部分テンプレートがviewsフォルダ内にあるhogesフォルダに存在する
2. 部分テンプレート名が_hoge.html.erbである
3. 部分テンプレート内で使う変数がhogeである
https://pikawaka.com/rails/partial_template  
    
## own?メソッド
- 編集、削除アイコンのリンクを、ログインユーザ自身の投稿かどうか判定して表示させる場合、判定式はviewで条件分岐せず、モデルのメソッドを利用している方がいい。
理由としては、viewにロジックを書かないことで、条件変更時にモデルメソッド1箇所の変更で済むので保守性が良くなる。
```
#userモデル 
def own?(object)
    self.id == object.user_id
end
```
```
#indexのビュー
if current_user&.own?(post)
```
  
- ownよりown?のがいいらしい（可読性）
  
- (object)は(post)でもいいが、汎用性向上のためobjectにしている
  
- `&`はぼっち演算子。
レシーバ(current_user)がnilであった場合でもエラーが発生しなくなる。
  
## buildメソッド
基本的にnewメソッドと同じだが、モデルの関連付けをした際は暗黙の了解で、buildを使うらしい。  
```
#postモデル

def create
    @post = current_user.posts.build(post_params)
    if @post.save
      redirect_to posts_path, success: '投稿しました'

    else
      flash.now[:danger] = '投稿に失敗しました'
      render :new
    end
  end

```
  
## edit update destroyアクションは必ずcurrent_user.hoges.find(params[:id])で対象データを取得する
edit update destroyアクションで対象データを`@post = Post.find(params[:id])`でも取得できるが、`@post = current_user.posts.find(params[:id])`にしているのは、current_user.postsでログインユーザの投稿したpostsのみに絞ることで他人の作成した投稿を変更できないようにするため。  
Post.find(params[:id])だと全てのpostsから探すことになり、万が一パラメータを変えられた場合に他人の作成した掲示板を変更できてしまう。  
  
また、以下のようなコードは、暗に当該idのポストが存在することを悪意あるユーザーに伝えてしまうので、current_user.hoges.findにすれば404を出してくれるのでそのリスクは軽減される。  
```
@post = Post.find(params[:id])
if @post.user_id != current_user.id
 #エラーを出す
end
```
  
Rails初心者とバレる書き方  
https://tech-essentials.work/questions/456  
  
## destroy! create!について
destroy!やcreate!は処理失敗時に、例外をエラーとして返す。  
destroyやcreateは処理失敗時に、例外は発生させずに実行結果をfalseとして返す。  
 => 成功・失敗をハンドリングして表示を出し分けないのであれば、destroy!を使うべき。  
  
## swiperの導入の注意点
- Yarnでswiperを導入し、CSS・JSの適用をしhtml(ビュー)の実装をして使用する流れだが、v5.4.5以降node_modulesディレクトリ配下に作成されるswiperというフォルダ内にインストールされるファイルやディレクトリ構成やファイル名が変更されたため、過去の記事を参照するとファイル構造が違うので動かない！  
=>下記の記事を参考にして実装した。  
https://qiita.com/miketa_webprgr/items/0a3845aeb5da2ed75f82  
  
- 更に、今回バージョン7を使用したがバージョン6とバージョン7では動作に関わるクラス名が「swiper-container」から「swiper」に変わっているので注意する。  
https://kopa-diary.com/swiper-js-how-to/  
  
- assets/javascript/application.js内に  
```
//= require swiper/swiper-bundle.js
//= require swiper.js
# この順番を間違えると上手く動かないらしい
```
のように記載する必要があるが、この`swiper.js`はYarnでインストールしてnode_modules配下に置かれるJSファイル(swiper-bundle.js)とは別に、新しく自分で作成する必要があるJSファイル。  
ここで作成するJSファイルが、設定したswiper/swiper-bundle.jsを参照し、スライダー機能を各クラスに適用させるような仕組みとなっている。  
  
- app/assets/javascripts/swiper(任意の名前).js作成時に気をつける点  
application.html内でHEADタグ内にJSのスクリプトを置くように設定している場合、BODYタグ内のDOM（HTMLの構造、どこに何クラスがあるかなど）を読み込む前にSwiperが発動してしまい、SwiperはJSの適用先が分からないのでSwiperによるスライド機能は当然実装されない。  
  
=>解決策として  
`$(function() {...})` を活用して、swiper.jsの処理を一旦予約状態で止めておいてHTMLの読み込みが全て完了した後にswiper.jsが実行されるようにする。  
https://qiita.com/bakatono_super/items/fcbc828b21599568a597  
  
```
#例
$(function() {
  new Swiper('.swiper', {
    pagination: {
        el: '.swiper-pagination',
    },
  })
})
```
  
## ログイン制御に関して
- require_login  
ログインをしていないユーザーをアクション単位で弾く。アクセスしようとしたURLをセッションに格納し、not_authenticatedを実行するメソッド。  
以下のようにbefore_actionで指定する。  
```
# コントローラ
before_action :require_login, only: %i[new create edit update destroy]
```
アクションごとに変える場合は、only: %i[  アクション ]をつける。  
アクション内の分岐など、もっと細かい単位で弾きたい場合はlogged_in?を使う。  
  
- not_authenticated
先ほどのrequire_login内で、このメソッドも実行される。 デフォルトではredirect_to root_path（自動的にルートに飛ばされる）と定義されているが、カスタマイズしたい場合はapplication_controllerで上書きをする。  
```
# application_controller

  def not_authenticated
    redirect_to login_path, warning: 'ログインしてください'
  end
```
https://qiita.com/aiandrox/items/65317517954d8d44d957  
  
## n + 1問題
SQLが必要以上に実行されてしまいパフォーマンスが落ちる問題。  
今回でいうと画像投稿の一覧を表示する際に画像に紐づくユーザーの名前を取得するために画像投稿の数だけSQLが発行されてしまう。  
  
SQLが発行されるタイミングは  
**画像投稿の一覧を取得する際（1）**  
**ユーザーの情報を取得する際(投稿数分)（N）**  
の回数になるので『N+1問題』と呼ばれている。  
  
=> 解決法  
画像投稿(@posts)を取得の際にincludesを使うとSQLが  
**画像投稿の一覧を取得（1）**  
**ユーザーの情報をまとめて取得（1）**  
の2回になりN+1 問題が解決する。  
```
# Postコントローラ
  def index
    @posts = Post.all.includes(:user).order(created_at: :desc)
  end
```
