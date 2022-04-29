# 10. 通知機能の実装
### 内容
- 通知機能を実装してください。
- タイミングと文言は以下の通りとします。（リンク）と書いてある箇所はリンクを付与してください。
  - フォローされたとき
    - xxx（リンク）があなたをフォローしました
    - 通知そのものに対してはxxxへのリンクを張る
  - 自分の投稿にいいねがあったとき
    - xxx（リンク）があなたの投稿（リンク）にいいねしました
    - 通知そのものに対しては投稿へのリンクを張る
  - 自分の投稿にコメントがあったとき
    - xxx（リンク）があなたの投稿（リンク）にコメント（リンク）しました
    - 通知そのものに対してはコメントへのリンクを張る（厳密には投稿ページに遷移し当該コメント部分にページ内ジャンプするイメージ）

- 既読判定も行ってください。通知一覧において、既読のものは薄暗い背景で、未読のものは白い背景で表示しましょう。
- 既読とするタイミングは各通知そのものをクリックした時とします。
- 不自然ではありますが通知の元となったリソースが削除された際には通知自体も削除する仕様とします。

### 補足
- ポリモーフィック関連を使うこと
- ヘッダー部分の通知リストには最新の10件しか表示させないこと

# 作業手順
今回やることが多く実装方法も複雑に感じたので、細かくやることを分けて確認してみた。

### テーブル作成、モデル設定など下準備
- マイグレーションファイルの作成、実行(rails g model Activities subject:references{polymorphic} user:references action_type:integer read:booleanでマイグレーションファイル作成, not nullなど追記して実行)
- ActivityとRelationship, Like, Commentモデルのポリモーフィック関連付けと、UserモデルとActivityモデルの関連付けの設定

### 通知(activity)機能の実装
- コールバックの設定(Comment, Relationship, Likeモデル)
- activity_typeのenumの設定(Activityモデル)

### ヘッダ部分のハートアイコン部分をクリックしマークダウンで通知一覧を表示する
- ビューの修正と作成(_header.html.slim, _header_activities.html.slim, _liked_to_own_post.html.slim, _followed_me.html.slim, _commented_to_own_post.html.slim)
- 翻訳ファイルで日時表示(config/locales/ja.yml)のフォーマットを設定をし、通知の各パーシャル内でlocalizeメソッドで呼び出す。

### マイページの通知一覧画面の作成
- routes.rbにマイページの通知画面のパスの設定(resources :activities, only: %i[index])
- _sidebar.html.slimに通知一覧の項目を追加
- Mypage::AccountsControllerのindexアクション作成(rails generate controller mypage/activities indexで作成、Mypage::BaseControllerを継承するように修正)
- マイページの通知一覧画面(mypage/activities/index.html.slim)の作成
- ヘッダの通知を最新10件まで表示にする(Activityモデルにscopeを定義、shared/_header_activities.html.slimを編集)

### 既読管理の作成
- Activityモデルにread, unreadのenumを設定
- routes.rbにactivitiesコントローラのreadアクション用のパスactivities/:id/read(.:format)を追記
- 通知の各パーシャルの一番外側にlink_toを設定、activitiesコントローラのreadアクションにつながるようにする
- mypage.scssとapplication.scssに.readを定義し、既読なら背景色がグレーになるよう通知の各パーシャルのlink_to部分にclassを設定
- activitiesコントローラのreadアクションを作成
- Activityモデルに  include Rails.application.routes.url_helpersとredirect_pathメソッドを定義

### 未読の通知数(バッジ)をアイコンの右上に表示
- バッジのパーシャルを作成(shared/_unread_badge.html.slim)
- 表示部分の実装(shared/_header.html.slim修正)

・最後にcss整える(共通scssを作って、mypage.scssとapplication.scssでimportするように変更)

# 学んだこと

## ポリモーフィック関連 
- オブジェクト指向の概念の1つ。複数のモデルに紐づけるようなモデルを実装したい場合に使う。また、一つのモデル(クラス)を同じインターフェース(メソッド)を持ったクラス(モデル)が扱う場合に便利。<br>
  [参考: Railsガイド](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)<br>
  [Railsのポリモーフィック関連とはなんなのか](https://qiita.com/itkrt2y/items/32ad1512fce1bf90c20b)<br>
  <br>
  今回の場合に置き換えて言うと、それぞれいいねをされた時、コメントされた時、フォローされた時に通知レコード(activitiesモデル)が作成されるようにしたい。つまり、activities、likeｓ、commentsモデルという複数モデルをactivitiesモデルと紐付けたい。<br>
  また、今回は各3つのモデル(クラス)内にactivityレコードを作成する`create_activities`という共通のメソッドが定義する(インタフェースが統一されている)ケースなので、ポリモーフィックを使う。<br>
  インタフェースが統一されているので`activity.subject`などで呼び出す時に呼び出す側はfollow・like・commentどのクラスなのかを意識する必要がなく、これができていない場合(ポリモーフィックを使わない場合)はcase文を使ってfollow・like・commentを判別して実装しなければならず不便。<br>

- ポリモーフィックを使わない例として、通知モデルにcomment_id,like_id,post_idなどの外部キーを追加する方法がある<br>
  [参考:【Rails】通知機能を誰でも実装できるように解説する【いいね、コメント、フォロー】](https://qiita.com/nekojoker/items/80448944ec9aaae48d0a#%E3%83%95%E3%82%A9%E3%83%AD%E3%83%BC%E3%81%97%E3%81%9F%E3%81%A8%E3%81%8D)<br>
    
    => しかしこれは、新しく通知モデルと関連付けしたいモデルが発生した場合テーブルの変更（列の追加）を行う必要がある点やnullのデータが作成される点、case文を使う点などがあまり良くない([参考: Railsで通知機能を実装してみた](https://qiita.com/P9eQxRVkic02sRU/items/a41e4136c3f4c656c72d))<br>
       これらの理由からポリモーフィックを使った方がいい。<br>

- 今回のケースとは違うが、「あるテーブルが複数のテーブルに対して、自身が多、紐付き先が一で関連する場合のテーブル設計」について、以下の記事でポリモーフィックを含む4つのアプローチがあると言っている<br>
  [複数のテーブルに対して多対一で紐づくテーブルの設計アプローチ](https://spice-factory.co.jp/development/has-and-belongs-to-many-table/)<br>
  複数のテーブルに対して紐づけたい時に必ずしもポリモーフィックを使った方がいいというわけではなく、その都度状況に応じて考える必要があることがわかった。<br>
  
## 通知機能の実装方法
やりたいこと:  
通知をクリックしたら、通知が未読(unread)の場合は既読(read)になるようにし、指定されたリンク先に飛び(いいね、フォロー、コメントによってそれぞれ違う遷移先)再度通知を見た時既読のものは背景色がグレーになるようにする。  
  
<これを実現するために具体的にやること>
1. クリックした時にActivitiesコントローラのreadアクションを実行するようにする  
(Activityモデルにenumの設定、routes.rbにパス(read_activity PATCH /activities/:id/read)を設定、shared/XXXXXのそれぞれの通知のパーシャルの一番外側にlink_to(aタグ)を設定しそのパスに遷移するようにする)  
  
2. readアクション内では対象のactivityのreadカラムをunread(未読)の場合にはread(既読)に更新し、通知をクリックした際の課題指定の遷移先に飛ぶようにする  
(Activityモデルにredirect_pathメソッドを定義してそれぞれの通知タイプ(action_type)に応じて遷移先を設定しておく)  
  
3. 通知を見た時既読のものは背景色がグレーになるように設定する  
(scssファイルに.readは背景をグレーにする定義をしておき、それぞれの通知のパーシャルのlink_toのクラス定義部分に#{‘read' if activity.read?}”のようにして既読ならグレーのcssが当たるように設定しておく)  
  
## Activityモデル(テーブル)の設計
```rb
# マイグレーションファイル
class CreateActivities < ActiveRecord::Migration[5.2]
  def change
    create_table :activities do |t|
     　　　　# t.stringの"subject_type"とt.bigintの"subject_id"が生成される。subject_typeは関連するモデル名（follow、like、comment）、subject_idは関連するモデルのレコードのidが入る。
      t.references :subject, polymorphic: true
      　　# references型で外部キーつける(user_idが生成、indexが貼られる)。通知を送る対象のuser_idが入る。
      t.references :user, foreign_key: true 
      　　# フォローした場合は0（followed_me）、いいねした場合は1（liked_to_own_post）、コメントをした場合は2（commented_to_own_post）が保存されるようモデル側でenumを設定する。
      t.integer :action_type, null: false
      　　# 未読か既読かが入る。default: falseでデフォルトは未読に設定。モデル側で未読（unread）の場合はfalse、既読（read）の場合はtrueと定義する。
      t.boolean :read, null: false, default: false 

      t.timestamps
    end
  end
end
```

```rb
# Activityモデル
class Activity < ApplicationRecord
  belongs_to :subject, polymorphic: true
  belongs_to :user

  enum action_type: { commented_to_own_post: 0, liked_to_own_post: 1, followed_me: 2 }
  enum read: { unread: false, read: true }
```

```rb
# Commentモデル
class Comment < ApplicationRecord
  has_one :activity, as: :subject, dependent: :destroy
```
=> Like, Relationshipモデルにも同様にアソシエーションを設定。<br>


## has_one について
今回の事例においては、一つのフォロー登録・いいねアクション・コメント投稿をした際に通知が飛ぶのは必ず１件だけ(フォロー、いいね、コメントのレコードが作成されたら通知のレコードも1件作成される)なので、has_manyではなく、has_oneを使用している。<br>


## モデル同士のアソシエーションについて
ポリモーフィック関連付けは、Relationship, Like, CommentのそれぞれとActivityモデルでしていて、ポリモーフィックな関連付けとは別の話として、通知を受け取る対象のuserの情報が必要なのでActivityモデルはUserモデルと紐付けをする必要がある（通知一覧取得にあたって必要。例えばユーザid=1の人に紐づく通知レコードを取得して表示する）<br>


## コールバックの設定について
[Active Record コールバック](https://railsguides.jp/active_record_callbacks.html#%E3%82%B3%E3%83%BC%E3%83%AB%E3%83%90%E3%83%83%E3%82%AF%E3%81%AE%E6%A6%82%E8%A6%81)

コメントされた場合、いいねされた場合、そしてフォローされた場合に、通知レコードが作成されるようにしたい。つまり、activitiesレコードは、commentsレコード、likesレコード、relationshipsレコードが作成された際に連動して作成したい。<br>
この場合、各3つのコントローラのcreateアクションにactivityレコードを作成する記述を追加する方法もあるが、各モデルのコールバックを活用すると書く量も少なくDRYなので便利。<br>
ただ、以下のサイトにあるようにコールバックは背景で何が起こっているか見えづらい為、使用には注意が必要らしい。
今回はコールバックの使い方もシンプルで、逆にコントローラ側に実装する方が見づらいので、コールバックの方がいいと思われる。<br>
[苦しめられてやっと理解できたRailsコールバックの使い方](https://tech.kitchhike.com/entry/2018/06/30/232400)<br>


## after_create_commitについて
[トランザクションのコールバック](https://railsguides.jp/active_record_callbacks.html#%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%AB%E3%83%90%E3%83%83%E3%82%AF)<br>

after_commitのエイリアスメソッド。after_saveコールバックと似ているが、以下の違いがある。<br>
・ `after_save`  : オブジェクトをDBに保存した直後で実行。INSERT、UPDATE両方で実行<br>
・ `after_commit`:	after_save後のDBにCOMMITされた直後に実行<br>
[ActiveRecordのコールバック早見表](https://morizyun.github.io/ruby/active-record-callback.html)

=> commitした後の発火の方が安全なのでafter_create_commitを使用。<br>


## enumについて
１つのカラムに指定した複数個の定数を保存できる様にする為のモノ。enumを使うと指定した複数個の定数以外の値は保存できない様にしたり、カラムに指定した定数が入っているレコードを取り出すのが容易なる便利なメソッド。<br>
[【Rails】 enumチュートリアル](https://pikawaka.com/rails/enum)<br>

Activityモデルにaction_typeとreadについて以下のように定義している
```rb
  enum action_type: { commented_to_own_post: 0, liked_to_own_post: 1, followed_me: 2 }
  enum read: { unread: false, read: true }
```
DBに実際に保存されるのは0,1,2の整数やfalse, trueの真偽値だが`activitiy.action_type`や`activity.read`で取得できるのは定数の方の情報。<br>
また、enumは以下のような使い方ができる。<br>
・ `activity.read?` => そのactivityオブジェクトがread(既読)なのか判定できる<br>
・ `activity.read! if activity.unread?` => activityオブジェクトがunread(未読)ならreadに更新する。</br>
こうすることでコード量も減り、可読性も高まる。<br>


## action_typeカラムの使い方について
今課題ではactivityテーブルにaction_typeカラムを設けて、Activityモデルで`enum action_type: { commented_to_own_post: 0, liked_to_own_post: 1, followed_me: 2 }`のようにenumを設定している。<br>
そして各モデル(Comment, Like, Relatiopnship)のcreate_activitiesアクションでactivityレコードを作成する際に上記のaction_typeがコメント、いいね、フォローに応じてそれぞれ入るように設定している。<br>
コメント、いいね、フォローの際の通知のビューをそれぞれ`_commented_to_own_post`, `_liked_to_own_post:`, `_followed_me`に分けてパーシャルsharedディレクトリに作成しておき、呼び出し元のビュー`_header_activities.html.slim`にて以下のようにしている。
```rb
current_user.activities.each do |activity| 
    = render "shared/#{activity.action_type}", activity: activity
```
=> current_user.activitiesで取得したactivityレコードの1つ1つをactivityに入れeachで回し、activity.action_typeでは各レコードに応じて`commented_to_own_post`, `liked_to_own_post`, `followed_me`のいずれかを取得でき、`shared/#{activity.action_type}`とすることで`shared/commented_to_own_post`のように変換され、eachで回す各レコードに応じて用意していたパーシャルがそれぞれrenderされるようになっている。<br>
   action_type(enum)とパーシャルを連動させるテクニック！<br>


## localizeメソッド
通知パーシャルの日時の表示部分が何も設定しないと`2022-04-23-17:30:33 +0900`のように詳細な情報まで表示されて見づらい。そこで、locale.jaに翻訳を記載し、localizeメソッドを使い呼び出すことでフォーマットを整えることができる。
localizeメソッドは`I18n.l`のように使用するが、`ControllerやviewファイルではI18n.l()をl()に省略して記述する`ことが出来る。※コンソールでは、省略することが出来ないので注意。

```rb
  time:
    formats:
      default: "%Y年%m月%d日(%a) %H時%M分%S秒 %z"
      long: "%Y/%m/%d %H:%M"
      short: "%m/%d %H:%M"
```
[Railsで日時をフォーマットするときはstrftimeよりも、lメソッドを使おう](https://qiita.com/jnchito/items/831654253fb8a958ec25)<br>
[Railsガイド - localize](https://railsguides.jp/i18n.html#%E3%83%AD%E3%82%B1%E3%83%BC%E3%83%AB%E3%81%AE%E8%A8%AD%E5%AE%9A%E3%81%A8%E5%8F%97%E3%81%91%E6%B8%A1%E3%81%97)<br>


##  link_toでアンカーを設定する
例えばアンカータグが設定された`コメント`というリンクをクリックしたとしたらページに遷移後、対象のコメントの部分まで画面がジャンプするのでいちいちスクロールしなくて済む。<br>
link_toで使うには以下のようにする。
```rb
= link_to 'テキスト', リンク先のpath(anchor: 'リンク先のid', ~~~
```
[Rails link_toでアンカーを設定する](https://qiita.com/tatsuya1156/items/595fe0df912c6c89f991)  

## resourcesとresourceの違い
routes.rbで以下のように設定している。
```rb
  namespace :mypage do
    resource :account, only: %i[edit update]
    resources :activities, only: %i[index]
  end
```
resourcesは今回でいう通知オブジェクトのようにアプリケーション上に「複数存在する」リソースを扱う場合に用いる。  
自身のプロフィールの様に、ログインユーザーからみてアプリケーション上、１つしか存在しない様なリソースは、REST的なルーティングを定義したい場合には、resourceを定義する。  
[Railsのresourcesとresourceついて](https://qiita.com/Atsushi_/items/bb22ce67d14ba1abafc5)  
[単数形リソース](https://railsguides.jp/routing.html#%E5%8D%98%E6%95%B0%E5%BD%A2%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9)  

## routes.rbのRESTfulな書き方
route.rbで以下のようにRESTfulに定義している。
```rb
# mypage/acitivitiesになる。URLを見ただけで、mypageのacitivitiy一覧なんだろうなと推測できる
# 今回は一覧を見る画面をmypage以下で見るという仕様にしているので、以下のようにするのが良い
namespace :mypage do
    resources :activities, only: %i[index]
end

# activities/id/readになる。これを見るとacitivityのid番目をreadしたんだなあと推測できる
# readアクションでは対象のactivityを既読処理をしたいだけで、mypageとは無関係なので以下のように別で定義するのが良い
resources :activities, only: [] do
    patch :read, on: :member
end
```
みけたさんの質問の回答がとてもわかりやすかったので、参考にさせて戴きました。  
https://tech-essentials.work/questions/131  

#### ※on: :member
デフォルトで作成されるRESTfulなルーティングはresourcesで指定した場合index,new,show,create,update,edit,deleteの７つだが、新しい独自のルーティング(アクション)を追加したい場合に使う。  
[メンバールーティングを追加する](https://railsguides.jp/routing.html#%E3%83%A1%E3%83%B3%E3%83%90%E3%83%BC%E3%83%AB%E3%83%BC%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%92%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)  
[Railsのroutesにつけるmemberってやつ](https://qiita.com/ryuuuuuuuuuu/items/607bf3ce92d80ceb9057)

## aタグ内でaタグを利用したい時はobjectタグを使う
shared/_commented_to_own_post.html.slimで以下のようにobjectタグを使っている。  
(followed_me, liked_to_own_postパーシャルも同様)
```rb
= link_to read_activity_path(activity), class: "dropdown-item border-bottom #{'read' if activity.read?}", method: :patch do
  = image_tag activity.subject.user.avatar.url, class: 'rounded-circle mr-1', size: '30x30'
  object
    = link_to activity.subject.user.username, user_path(activity.subject.user)
  | があなたの
  object  
    = link_to '投稿', post_path(activity.subject.post)
  | に
  object
    = link_to 'コメント', post_path(activity.subject.post, anchor: "comment-#{activity.subject.id}")
  | しました
  .ml-auto
    = activity.created_at
```
全体をリンクにしてるけど、中の一部は違う場所にリンクを貼りたい場合に、aタグをaタグで囲うとうまく変換されないので中の部分をobjectタグで囲う。  
[aタグの中にaタグを書きたい時のtips](https://qiita.com/fukamiiiiinmin/items/7412b21c6df5de31cab1)

## Railsの不特定ModuleやClass(Modelなど)で`_path`を使う場合
Railsではpath/to/fooを表示するためにfoo_pathのような便利なメソッド(Url Helper)がある。  
しかしこの機能はController、HelperとViewでしか使えないので、modelや自作moduleで使いたい場合には`include Rails.application.routes.url_helpers`で取り込む必要がある。  
今回はActivityモデルの`redirect_path`メソッド内で`_path`を使用したい為、上記設定をincludeしている。  
[Railsの不特定ModuleやClass(Modelなど)で`_path`を使う](https://qiita.com/jerrywdlee/items/f91c9ea01055cb74083c)  

## redirect_pathメソッドでaction_type.to_symとシンボルに変換している理由
質問させて頂きました。
https://tech-essentials.work/questions/466

## Bootstrap関連
[ドロップダウンの作り方](https://getbootstrap.jp/docs/4.2/components/dropdowns/)  
[バッジの作り方](https://getbootstrap.jp/docs/4.3/components/badge/)  

# 感想
全体を通してポリモーフィック関連の概念も馴染みなく、実装も自分にとっては難しく、複雑だったので細かく分けて1つ1つ確認する必要があり時間がかかりました。  
でも、以前Runteqの応用でポリモーフィックに触れた際に、駆け足で終わらせ全然理解できてなかったしそもそも覚えてなかったので、今回理解を深められて、また理解するところまで頭を持っていけて少しですが前より成長したのかなと感じられて良かったです！  
いろんなケースによって色々な設計方法があり、その最善を選択していくのがいかに難しいかを実感しております。(コードリーディングだけで精一杯・・)  
いつも時間がかかるのはアソシエーションに関連する部分で、まだ苦手意識があり、実際のテーブルはどうなっていてテーブルがどう紐づいていてどうデータを取得しているのかの図式が頭の中で想像するだけでは難しく、いろんな参考サイトを渡り歩き関連図を見たりしました。SQLの知識もかなり重要なんだなあと感じました。  
おいおい、アソシエーションのまとめみたいなのをTILにできたらいいなと思います。
