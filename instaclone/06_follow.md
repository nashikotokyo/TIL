# 06.フォロー機能を実装
### 内容
- フォロー機能を実装してください。

### 補足
- フォロー・アンフォローは非同期で行う。form_withを利用すること。
- 適切なバリデーションを付与する
- 投稿一覧画面について
  - ログインしている場合
    - フォローしているユーザーと自分の投稿だけ表示させること
  - ログインしていない場合
    - 全ての投稿を表示させること
- 一件もない場合は『投稿がありません』と画面に表示させること
- 投稿一覧画面右にあるユーザー一覧については登録日が新しい順に5件分表示してください
- ユーザー一覧画面、詳細画面も実装すること
  
# 作業の流れ
- ユーザの一覧画面と投稿一覧画面の最新5件のユーザ部分作成
- ユーザの詳細画面作成
- フォローアンフォロー用のルーティング設定
- relationshipモデルとテーブルを作成
- UserとRelationshipモデルのアソシエーションとバリデーションの設定
- Userモデルにfollow, unfollow, following?メソッドを定義
- _follow_area.html.slim, _follow.html.slim, _unfollow.html.slimのビューを追加
- relationshipsコントローラを作成
- create.js.slim, destroy.js.slimを作成
- フォローアンフォローボタンを表示させる部分のビュー修正(users/_user.html.slim, users/show.html.slim)
- 自分がフォロー敷いているユーザのみフィードに表示されるように変更(posts_controller.rbのindexアクション修正、Userモデルにfeedメソッド定義)
- 投稿が一件もない時、投稿がありませんと表示させる対応(index.htmlのフェード表示部分ロジック修正)

# 学んだこと  
## フォロー機能のアソシエーションに関して  
- Railsチュートリアルが説明も詳しく載っていたので参考にした。  
( https://railstutorial.jp/chapters/following_users?version=4.2 )  
  
- 理解のポイント  
中間テーブルを使う点はいいね機能と似ているが、フォロー機能は中間テーブルのカラム両方ともにユーザのidが入る点が違う。そのためカラム名をuser_idではなく、区別するために`follower_id`や`followed_id`としている。そして常にユーザを`フォローする人(follower)`と`フォローされる人(followed）`に分けて考えアソシエーションを設定する必要がある。  
[![Image from Gyazo](https://i.gyazo.com/f2ed62c097f1f0ee6138274d11ada759.jpg)](https://gyazo.com/f2ed62c097f1f0ee6138274d11ada759)  
  
また、`フォローする側(follower)から見たrelationship`と`フォローされる側(followed)から見たrelationship`を分けて考えるため(フォローする側(follower_idのユーザ)からたどってフォロー中のユーザ(followed_idのユーザ)を取得する時と、その逆のフォローされている側(followedr_idのユーザ)からたどってフォロワー(followed_idのユーザ)を取得する時と分けて考えるため)  
relationshipsテーブルを`active_relationships`と`passive_relationships`に分けて考えactiveとpassiveの2方向のアソシエーションを設定するイメージ。  
※実際のテーブルはrelationshipsテーブルだが、仮のテーブル名としてつけている  
[![Image from Gyazo](https://i.gyazo.com/4cb4731bf924b27248f2a7fa932c48db.png)](https://gyazo.com/4cb4731bf924b27248f2a7fa932c48db)  
[![Image from Gyazo](https://i.gyazo.com/7a49d93c367171597995b92e1c21b3da.png)](https://gyazo.com/7a49d93c367171597995b92e1c21b3da)  
  
## scopeを使用して特定の(最新5件)レコードを取得  
以下のようにモデルに定義することで最新5件の登録ユーザを取得できる。  
コントローラ側で長ったらしいメソッドチェーンを書くことなくスッキリとした可読性の高いコードになる。  
```
#Userモデル
scope :recent, ->(count) { order(created_at: :desc).limit(count) }
```
```
#postsコントローラ
@users = User.recent(5)
```
## following_idsメソッド  
Userモデルのfeedメソッド内で使用。  
`following_idsメソッド`は、Active Recordによってhas_many :following関連付けから自動生成されたもの。これにより、user.followingコレクションに対応するidを取得できる。つまりそのユーザがフォローしているすべてのユーザーのuser_idを取り出し、配列を作り出している。  
```
#Userモデルのfeedメソッド
def feed
    Post.where(user_id: following_ids << id)
  end
```
( https://railstutorial.jp/chapters/following_users?version=4.2#code-initial_working_feed )  
  
## Userモデルのfeedメソッドについて  
投稿一覧画面で、ログイン済の場合フォローしてるユーザの投稿のみを表示するためにUserモデルに`feedメソッド`を定義している。  
```
#Userモデルのfeedメソッド
def feed
    Post.where(user_id: following_ids << id)
end
```
※上記↑ Post.where(user_id: self.following_ids << self.id)のselfが省略されている  
```
#postsコントローラのindexアクションで使用
  def index
    @posts = if logged_in?
      current_user.feed.includes(:user).order(created_at: :desc).page(params[:page])
             else   
               Post.all.includes(:user).order(created_at: :desc).page(params[:page])
             end
    @users = User.recent(5)
  end
```
=>Post.where(user_id: following_ids << id)でどういうことをやっているのかなと思い、binding.pryで確認してみた。  
```
#Userモデルでbinding.pryした結果
    70: def feed
    71:   binding.pry
 => 72:   Post.where(user_id: following_ids << id)
    73: end
[1] pry(#<User>)> current_user
NameError: undefined local variable or method `current_user' for #<User:0x00007faf92784148>
from /Users/natsumimatsuda/workspace/menta!/insta_clone/insta_clone_by_me/vendor/bundle/ruby/2.6.0/gems/activemodel-5.2.6/lib/active_model/attribute_methods.rb:430:in `method_missing'
[2] pry(#<User>)> self
=> #<User:0x00007faf92784148
 id: 11,
 email: "user@ex.com",
 crypted_password:
  "$2a$10$d/W0jhT6g7F72zVOPDpd6OZFM8a6A2t.edjPSD57BD2j2mBhToUOm",
 salt: "tm_ZFSXcs8GLwtSM6xGP",
 username: "user",
 created_at: Mon, 21 Feb 2022 16:49:13 JST +09:00,
 updated_at: Mon, 21 Feb 2022 16:49:13 JST +09:00>
[3] pry(#<User>)> following_ids
   (2.1ms)  SELECT `users`.`id` FROM `users` INNER JOIN `relationships` ON `users`.`id` = `relationships`.`followed_id` WHERE `relationships`.`follower_id` = 11
  ↳ (pry):3
=> [1, 3, 5, 6, 7, 9]
[4] pry(#<User>)> following_ids << id
=> [1, 3, 5, 6, 7, 9, 11]
```
=> `following_ids`でログインユーザがフォローしているユーザのid(usersテーブルのid)を取得、そこに`<<`を使ってログインユーザのidを追加している。  
そして`Post.where`を使って、postテーブルのuser_idが上記で取得したユーザのidに合致する投稿(posts)を取得している。  
  
feedメソッドを定義することでコントローラ側のコードの可読性も上がりスッキリする。  
一度定義すれば繰り返し使えるので便利。  
  
## 解答例のuserモデルのfollow, unfollowメソッドの書き方について質問  
https://tech-essentials.work/questions/463  
  
# 感想
中間テーブルを用意し、非同期でjsファイルを使用し実装する点などはいいね機能と似てたので、実装の流れは理解しやすかった。  
ただ、アソシエーションの設定の仕方がいいね機能とは違って複雑だったので最初はこんがらがり、いろんな解説サイトを見てみましたが、結局Railsチュートリアルの説明が一番わかりやすかったです。上記の学んだことでも書いたように、フォローする側とされる側で分けて考え、2方向(2パターン)の設定が必要なことがわかってからは理解しやすくなりました。  
  
まだ理解したレベルなので定着はしてないですが、今回初めて実装する機能だったので勉強できてよかったです！  
