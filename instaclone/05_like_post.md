# 05.投稿に対するいいね機能を実装
### 内容
- 投稿に対するいいね機能を実装してください
  
### 補足
- link_toを使って非同期処理として実装する
- like, unlike, like?メソッドをユーザーモデルに実装してそれを利用する形にする
- Likeモデルに適切なバリデーションを付与する
  
# 作業手順
- bundle exec rails g model like user:references post:referencesでLikeモデルとマイグレーションファイルを作成=>マイグレーションにユニーク制約を記載=>rails db:migrate
- likeモデルにもユニーク制約を記載
- user, postのモデルにアソシエーションを追記(has_many :likes, has_many :like_posts, has_many :like_users)
- userモデルにlike, unlike, like?メソッドを定義
- routes.rbを追記
- いいねボタンのビューファイル(_like_area.html.slim, _like.html.slim, _unlike.html.slim)、_post内のいいねボタン表示部分追記
- likesコントローラの作成(create/destroy)
- create.js.slimとdestroy.js.slimを作成
- show.html.slim内にもいいねボタン表示部分追記
  
# 学んだこと
## ＜カラムのペアにunique制約を付ける方法＞
同じユーザが2回以上同じ投稿にいいねはできないのでlikesテーブルのレコードはuser_idと:post_idのペアが1意になる必要がある  
  
**マイグレーション側の設定**  
`t.index [:user_id, :post_id], unique: true`  
重複したuser_idとpost_idのペアが登録されようとしたら、例外を発生させることができる。  
  
(https://qiita.com/TO-TO/items/dc34e3416df07ca1bc1d)  

**モデル側の設定**  
`validates :user_id, uniqueness: { scope: :board_id }`  
もしくは  
`validates :board_id, uniqueness: { scope: :user_id }`  
  
(https://railsguides.jp/active_record_validations.html#uniqueness)  
  
## ＜アソシエーションに別名をつける方法＞  
対象のユーザーが『いいね！』した投稿一覧を表示したい時  
**userモデル：**  
`has_many :like_posts, through: :likes, source: :post`  
=> @user.like_boards のような形で@userがいいねしているpostのcollectionを取得できる。  
  
対象の投稿を『いいね！』したユーザ一覧を表示したい時  
**postモデル :**  
`has_many :like_users, through: :likes, source: :user`  
=> @post.like_users のような形で@postをいいねしているuserのcollectionを取得できる。  
  
(https://railsguides.jp/association_basics.html#has-manyのオプション-source)  
  
## ＜Userモデルにlike, unlike, like?メソッドを定義する意味＞  
**like, unlike**  
=>「ユーザーが対象の投稿をいいね(いいね解除)する」という処理をコントローラ上により簡潔に直感的に記載できる。(createやdestroyを使っていいねレコードを作成/削除する方法より読みやすい)  
```
(例)
current_user.like(@post)
current_user.unlike(@post)
```
**like?**  
=>自分の掲示板かどうかを判定する条件分岐のロジックをビューに記載すると判定箇所が増えるたびにメンテナンス漏れが発生したり、ファイルの見通しが悪くなるため。  
  
## ＜<<演算子＞  
likeモデルでのlikeメソッドでいいねの処理を`<<演算子`を使用して下記のように書いている。  
```
#Userモデル
  def like(post)
    like_posts << post
  end
```
この演算子では`ユーザー.likes.create!(post_id: post.id)`と同様の処理が行われて、CREATE文のSQLが発行される。has_manyの設定によって記載できる記法。  
  
(https://railsguides.jp/association_basics.html#has-manyで追加されるメソッド-collection-object)  
  
## ＜deleteとdestroyの違いについて＞  
**deleteメソッド**  
=>条件に一致したレコードを、SQLを直接実行することで削除する。つまり、モデルを経由せずに削除していて関連づけしているレコードは削除されない。  
  
**destroyメソッド**  
=>destroyメソッドはモデルつまりActiveRecordを経由してるため、関連したバリデーション機能も有効ですし、関連づけされたレコードの削除も行われる。  
  
(https://qiita.com/tochisuke221/items/1d5e54f9697af85c62bc)  
  
## ＜include?メソッド＞  
配列の中に引数で渡した値が含まれているかどうかを判別しtrueかfalseで返す。  
```
a = [ "a", "b", "c" ]
a.include?("b")       #=> true
a.include?("z")       #=> false
```
(https://docs.ruby-lang.org/ja/latest/method/Array/i/include=3f.html)  
  
## ＜link_toのHTTPメソッドのデフォルトはGET＞  
link_toメソッドのHTTPメソッドには、GETがデフォルトで指定されている。  
今回ルーティングは以下のようになっている。  
```
#routes.rb

#likes POST   /likes(.:format)                           likes#create
#like DELETE /likes/:id(.:format)                      likes#destroy
```
これまではlink_toを使用した際は画面遷移の流れだったので、デフォルトの`GET`でよかったが、今回のいいね機能に関しては`POST`と`DELETE`なので、いいねアイコン部分のビューのlink_toメソッド内で、`method: :post`と`method: :delete`としてHTTP通信の種類を指定する必要がある。  
  
(https://railsdoc.com/page/link_to)  
  
## ＜link_toに任意のパラメータを付与する＞  
link_toにはlink_toに任意のパラメータを付与できる。  
今回`_like.html.slim`(いいねするためのボタン(♡)が表示される方)に以下のようにある。  
```
= link_to likes_path(post_id: post.id), method: :post, remote: true do
  = icon 'far', 'heart', class: ‘fa-lg’
```
```
※routes.rbの設定
#likes POST   /likes(.:format)        likes#create
```
=>`link_to likes_path(post_id: post.id)`に部分ではpost_idに対象postのidを渡し｀`href=“/likes?post_id=10”`(10は仮)のようなパスを生成しコントローラ側でこのパラメータを使えるようにしている。
  
(https://310nae.com/linkto-param/)  
  
## ＜Ajax通信の際のビューの実装ポイント＞  
- Ajax通信でjsで操作（切り替える）するのでいいねアイコン(likeとunlikeのボタン)の部分のビューはパーシャルに分けたほうがコードが簡潔で見やすい。
- Ajaxの処理の目印となるように、表示を変えたい部分にid属性を付与する必要あり。
（今回はlike_areaに記載）  
  
## ＜解答例の - if current_user && !current_user.own?(post)の部分＞  
以下を質問させて頂きました。  
(https://tech-essentials.work/questions/462)  
  
# 感想
以前ブックマーク機能を実装したことがありましたが、細かい部分を忘れてしまっていたので再度復習できてよかったです。  
特に、Userモデルにlike, unlike, like?メソッドを定義してコントローラやビューで使う方法や、Userモデルにhas_many :like_posts, through: :likes, source: :postと定義して@user.like_boards のような形で@userがいいねしているpostのcollectionを取得する方法はもっと慣れておきたいなと思います。  
  
前回コメント機能もajaxだったのでajaxの機能実装や流れには慣れてきたかなと感じます。  
