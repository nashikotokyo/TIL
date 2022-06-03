# 07.投稿の検索機能を実装する
### 内容
- 検索機能を実装してください
[![Image from Gyazo](https://i.gyazo.com/9417f6a1205725658e1663f0ba95f008.png)](https://gyazo.com/9417f6a1205725658e1663f0ba95f008)  
  
### 補足
- 全ての投稿を検索対象とすること（フィードに対する検索ではない）
- 検索条件としては以下の三つとする
  - 本文に検索ワードが含まれている投稿
    - こちらに関しては半角スペースでつなげることでor検索ができるようにする。e.g.「rails ruby」
  - コメントに検索ワードが含まれている投稿
  - 投稿者の名前に検索ワードが含まれている投稿
- ransackなどの検索用のGemは使わず、フォームオブジェクト、ActiveModelを使って実装すること
- 検索時のパスは/posts/searchとすること
  
# 作業手順
- 検索用のルーティング追加(routes.rb)
- 検索用のスコープ追加(Postモデル)
- 検索フォームのパーシャルの実装(posts/_search_form.html.slim)
- フォームオブジェクトクラスを実装する(app/forms/search_posts_form.rb)
- ApplicationControllerの追記(set_search_posts_formとsearch_post_paramsメソッド)
- postsコントローラーのsearchアクション実装
- 検索結果を表示させるビューの作成(app/views/posts/search.html.slim)と、検索フォームパーシャルを表示される部分の設定(app/views/shared/_before_login_header.html.slim, app/views/shared/_header.html.slim)
- 検索フォームにコメントとユーザー名の入力欄を追加し、本文の検索はor検索できるようにした。(applicationコントローラ、Postモデル、search_posts_form.rb、_search_form.html.slim&nbsp;の修正&nbsp;)
- ambiguousエラーの対応(Postモデルのscope :body_contain部分)
  
# 学んだこと
## フォームオブジェクトとは  
DBを使わないフォームや複数モデルを介す必要のあるフォーム(データベースのテーブルと1:1に紐づいていないフォーム)を実装する時に使う。  
form_withのmodelオプションにActive Record以外のオブジェクトを渡すデザインパターンのこと。  
form_withのmodelオプションに渡すオブジェクト自体もform objectと呼ぶ。  
  
**利点**  
- DBを使わないフォームでも、Active Recordを利用した場合と同じお作法を利用できるので可読性が増す
- 他の箇所に分散されがちなロジックをform object内に集めることができ、凝集度を高められる
  
https://tech.medpeer.co.jp/entry/2017/05/09/070758  
  
## Active Modelとは  
ActiveRecordのDBと連携しない版のこと。ActiveRecordはDBへの連携以外でも、validateや、attributeなど便利な機能があるが、DBへの連携は必要ないがViewに対するこれらの機能は使いたい、というケースの場合にはActiveModelを使う。また、Active Recordを利用したときと同じような記述をすることができる。  
(検索キーワードをcontrollerに送りたいだけでDBとは連携させない今回のケースには適切)  
  
## フォームオブジェクトクラスの定義
```
class SearchPostsForm
  include ActiveModel::Model
  include ActiveModel::Attributes

  attribute :body, :string
  attribute :comment_body, :string
  attribute :username, :string

end
```
クラスを定義し、`ActiveModel::Model`と `ActiveModel::Attributes`をincludeする。  
  
- ActiveModel::Model  
モデル名の調査（Naming)、変換(Conversion)、翻訳(Translation)、バリデーション(Validation)などの機能を提供。これによりActiveRecordモデルと対応していないオブジェクトでもモデルのような振る舞いを獲得できform_withやrenderなどのAction Viewヘルパーメソッドが使えるようになる。  
  
- ActiveModel::Attributes  
attributeメソッドを使って、属性と型を指定できる。attribute :bodyと書けば、object.bodyのように使える。  
また、RailsのActiveRecordにあるAttribute APIというActiveRecordを操作する際にクラス属性の型を意識しなくても、指定の型へ変換してくれるという機能を利用できるようになる。  
  
https://qiita.com/s_yanada/items/d856c11d074bcf98f73e  
https://ushinji.hatenablog.com/entry/2019/06/17/220611  
https://railsguides.jp/active_model_basics.html  
  
## 検索フォームのパーシャル内のform_withについて  
  
- **デフォルトのHTTPメソッドはPOST**  
フォームオブジェクトで使用する際にはdbには飛ばさないのでmethodをGETに変更する必要がある。  
```
= form_with model: search_form, url: search_posts_path, scope: :q, class: 'form-inline my-2 my-lg-0 mr-auto', method: :get, local: true do |f|
　〜略〜
```
　  
https://railsdoc.com/page/form_with  
- **scopeオプション**  
:scopeは各inputのname属性をグループ化するのに使う。これによりリクエストにのせて送られるパラメーターの構造が変わる。  
検索キーワードを入力するフォールドが一つだとほとんど意味が無いが、入力するフィールド数が複数だと下記のようにまとまってくれるので便利。  
  
▼scope: :qがない時のparams  
```
<ActionController::Parameters {"utf8"=>"✓", "body"=>"本文", "comment_body"=>"コメント", "username"=>"ユーザ名", "commit"=>"Search", "controller"=>"posts", "action"=>"search"} permitted: false>
```
▼scope: :qがある時のparams  
```
<ActionController::Parameters {"utf8"=>"✓", "q"=>{"body"=>"本文　本文", "comment_body"=>"コメント", "username"=>"ユーザ名"}, "commit"=>"Search", "controller"=>"posts", "action"=>"search"} permitted: false>
```
※form_withにモデルのインスタンスを与えた場合は、scopeを設定しなくても初めからグループ化されている。  
  
https://endoakak.hatenablog.com/entry/2020/08/29/200000  
  
## ルーティングのmemberとcollection  
どちらもresourcesでは自動で生成されない（サポートされていない）アクションへのルーティングを設定するときに使用する。  
  
**< memberとcollectionの違い >**  
生成するroutingに、:idが付くか付かないか。  
memberは付く（例) /posts/:id/search(.:format)）collectionは付かない(例/posts/search(.:format)）  
  
https://railsguides.jp/routing.html#コレクションルーティングを追加する  
  
## fetchメソッド  
fetchメソッドはrequireメソッドと似ているが、第二引数にデフォルトのバリューを設定することができる。  
```
#Applicationコントーラ
  
  def set_search_posts_form
    @search_form = SearchPostsForm.new(search_post_params)
  end
  
  def search_post_params
    params.fetch(:q, {}).permit(:body, :comment_body, :username)
  end
```
今回Applicationコントーラで上記のように`require(:q)`ではなく`fetch(:q, {})`としてパラメータを取得している。  
ApplicationコントローラでSearchPostsFormクラスのオブジェクトを生成する仕様にしているので検索機能を使わない時もいつでもこのメソッドは実行されるので検索していない時(Searchボタンを押してない時)はパラメータに:qがそもそもないことで、ActionController::ParameterMissingのエラーが出てしまうので、それを回避するため{}がデフォルト値として評価されるようにしている。  
  
https://www.sejuku.net/blog/58930  
  
## Postモデルのscope定義部分について  
条件に合致した投稿(posts)レコードを取得するscopeを事前に定義しておく  
```
#Postモデル
  scope :body_contain, ->(word) { where('posts.body LIKE ?', "%#{word}%") }
  scope :comment_body_contain, ->(word) { joins(:comments).where('comments.body LIKE ?', "%#{word}%") }
  scope :username_contain, ->(word) { joins(:user).where('username LIKE ?', "%#{word}%") }
```
- **プレースホルダ**  
`where('posts.body LIKE ?', "%#{word}%")`の部分の「?」はプレースホルダと言うもので、第2引数の値を「?」へ置き換えるための目印。  
プレースホルダーを使った書き方は、比較して条件を定義するときやandやor, LIKEを使うときにも使う。  
またSQLインジェクションなどのセキュリティリスクを防ぐ働きもある。  
  
- **SQLインジェクション**  
悪意を持つユーザーがSQL文などを不正な値を入れることで、サービスに不正な操作をさせることができる。ユーザーから入力された値を直接SQLに結合しないように実装する必要がある。  
  
whereメソッドは`モデル名.where("カラム名 = 検索値")`のように文字列指定する方法がある。  
しかしこの書き方だと検索値を直接渡してしまっていてSQLインジェクションを防げない。  
`where(カラム名: “検索値")のようにシンボル指定`、もしくは`where("カラム名 = ?, “検索値")のようにプレースホルダ`を使った方法で定義することで、特殊なSQL文字「'」「"」「NULL」「改行」などが入力されていてもエスケープしSQLインジェクションを防ぐことができる。  
　  
https://pikawaka.com/rails/where#SQLインジェクションとは  
https://railsguides.jp/security.html#sqlインジェクション  
https://rooter.jp/programming/ruby/rails_use_where_or_sanitize_sql_methods_to_avoid_sql_injection/  
  
- **whereとLIKEであいまい検索をする方法**  
`　モデルクラス.where("カラム名 LIKE ?", "検索したい文字列") `  
今回はscopeで定義して`Post.body_contain(検索したい文字列)`のように使っている。  
`"検索したい文字列"`の部分では%(0文字以上の任意の文字列)を活用して、様々なバリエーションで検索することができる。  
  
**前方一致(から始まる)：検索値%  
後方一致(で終わる)：%検索値  
部分一致：%検索値％**  
　  
https://www.sejuku.net/blog/71189  
  
- **joinsメソッド**  
関連するテーブル同士を結合(内部結合)してくれるメソッド。  
`モデル名.joins(:関連名)`  
1. joinsメソッドは、モデルのレコードしか取得しない。結合先のテーブルのレコードは取得しないので注意。取得したい場合は`select`を使う。  
1. モデルで定義した`アソシエーションの関連名`を引数に指定する事でテーブルを結合する事が出来ます。  
  
=> 関連名なので:usersでなく`:user`であることに注意。  
scope :username_contain, ->(word) { joins(`:user`).where('username LIKE ?', "%#{word}%") }  
  
https://railsguides.jp/active_record_querying.html#テーブルを結合する  
  
## フォームオブジェクトクラス(search_posts_form.rb)のsearchメソッドについて  
```
#app/forms/search_posts_form.rb
 
 def search
    scope = Post.distinct
    scope = splited_bodies.map { |splited_body| scope.body_contain(splited_body) }.inject { |result, scp| result.or(scp) } if body.present?
    scope = scope.comment_body_contain(comment_body) if comment_body.present?
    scope = scope.username_contain(username) if username.present?
    scope
  end

  private
  def splited_bodies
    body.strip.split(/[[:blank:]]+/)
  end
```
  
解答例で上記のように書いていて、いくつか疑問点があった。  
1. 最初にPost.distinctしてる意味は？Post.allじゃダメ？  
=>検索条件が1つ(postsのbodyのみ)の場合ならPost.allでも良いが、検索条件が複数出てきてor検索をする場合はdistinctにしないといけない。  
（今回のようにcommntsのbodyが検索の対象の場合、1つのpostに紐づく複数のcommentsの中で、例えばコメント検索をして2件ヒットしたらpostのレコードも同じものが2件取得されてしまう。）  
  
他受講生の質問で解決　https://tech-essentials.work/questions/93  
  
**・distinctメソッド**  
取得したレコードの重複をなくすメソッド  
https://qiita.com/toda-axiaworks/items/ad5a0e2322ac6a2ea0f4  
  
2. 最後のscopeって必要？  
=>ifは成立しない場合nilを返すので、1行目`scope = Post.distinct`で代入したscopeをちゃんと返すために必要。  
(seachメソッドの返り値がnilだと、postsコントローラで@search_form.search.includes~のように使っているので、includeがNoMethodErrorになる。)  
  
他受講生の質問で解決　https://tech-essentials.work/questions/138  
  
  **・後置ifやif節の条件不成立の時の返り値について**  
if 式は、条件が成立した節(あるいは else 節)の最後に評価した式の結果を返す。else 節がなくいずれの条件も成り立たなければnil を返す。  
  
https://ysk-pro.hatenablog.com/entry/if-return-value  
https://docs.ruby-lang.org/ja/latest/doc/spec=2fcontrol.html#if  
  
3. `scope = splited_bodies.map { |splited_body| scope.body_contain(splited_body) }.inject { |result, scp| result.or(scp) } if body.present?`の部分何やっているの？  
  
=>まず出てきたメソッドなどを調べる。  
  
- **stripメソッド**  
文字列の先頭や末尾に含まれる空白文字やタブを削除することができるStringクラスで用意されているメソッド。空白文字とタブだけでなく、改行コードや垂直タブなど(¥r,&nbsp;¥n,&nbsp;¥f,&nbsp;¥v)を削除する。ただし、全角スペースは削除しない。  
  
https://docs.ruby-lang.org/ja/latest/method/String/i/strip.html  
  
- **splitメソッド**  
文字列を分割して配列にするためのメソッド。  
区切り文字を引数に指定すると、そこで区切ってくれる。  
  
https://docs.ruby-lang.org/ja/latest/method/String/i/split.html  
  
- **.split(/[[:blank:]]+/)**  
全角スペースを含めたスペースを区切り文字に指定できる。(日本語を打つ時はスペースが全角なので)  
  
https://qiita.com/nao58/items/bf5d017a06fc33da9e3b  
  
- **mapメソッド**  
配列の入った変数.map {|変数名| 処理内容 }  
配列の要素の数だけ繰り返し処理を行うメソッド。  
mapは処理後に戻り値で配列を作成してくれます。元の配列が上書きされることはない。  
  
https://qiita.com/ecoyamas/items/07c9174d307e1ea66361  
  
**※mapとeachの違い**  
**each** => 元の配列を返す。(繰り返し処理に使う)  
**map** => 処理後の配列を返す。(繰り返し処理の結果を配列にしたいときに使う)  
処理結果を使いたい場合はeachではなくmapを使う  
  
- **injectメソッド**  
たたみ込み演算を行うメソッド  
  
https://docs.ruby-lang.org/ja/latest/method/Enumerable/i/inject.html  
  
- **orメソッド**  
https://railsguides.jp/active_record_querying.html#or条件  
  
=>結局何をやっているのか？  
他受講生の質問で解決　https://tech-essentials.work/questions/160  
  
mapとかが出てきたら分解して考える。  
`splited_bodies`&nbsp;→&nbsp;`["電車", "卵", "携帯"]`のような形式。  
`splited_bodies.map { |splited_body| scope.body_contain(splited_body) }&nbsp;`  
↓  
`&nbsp;["電車", "卵", "携帯"].map { |splited_body| scope.body_contain(splited_body) }`  
↓  
`[scope.body_contain("電車"), scope.body_contain("卵"), scope.body_contain("携帯")]`  
(厳密にはこれらが実行されて得られたActiveRecord::Relation))の形式  
　  
`[scope.body_contain("電車"), scope.body_contain("卵"), scope.body_contain("携帯")].inject { |result, scp| result.or(scp) }&nbsp;`  
↓  
&nbsp;`scope.body_contain("電車")or(scope.body_contain("卵")).or(scope.body_contain("携帯"))`  
の形式になる。つまり...  
```
  scope = Post.distinct
  scope = scope.body_contain("卵").or(scope.body_contain("電車")).or(scope.body_contain("携帯"))
  scope = scope.comment_body_contain(body) if comment_body.present?
  scope = scope.username_contain(username) if username.present?
  scope
```
最終的には下のように読める。  
```
  scope = Post.distinct
  scope = Post.distinct.where(‘users.body LIKE ?', “%卵%”).or(Post.distinct.where(‘users.body LIKE ?', “%電車%”)).or(Post.distinct.where(‘users.body LIKE ?', “%携帯%”)))
  scope = scope.comment_body_contain(body) if comment_body.present?
  scope = scope.username_contain(username) if username.present?
  scope
```
  
## Ambiguousエラー  
検索ワードを本文とコメント両方に入力しsearchボタン押したらAmbiguousエラーが出た  
```
#エラー内容
ActiveRecord::StatementInvalid at /posts/search
Mysql2::Error: Column 'body' in where clause is ambiguous: SELECT COUNT(DISTINCT `posts`.`id`) FROM `posts` INNER JOIN `comments` ON `comments`.`post_id` = `posts`.`id` WHERE (body LIKE '%テスト%') AND (comments.body LIKE '%修正%')
```
- 「Mysql2::Error: Column ‘カラム名’ in where clause is ambiguous」は複数テーブルに渡って同じカラムが存在していた場合に発生する =>今回はusersのbodyとcommentsのbody  
- JOINでテーブル結合を行い、加えてWHEREでカラム指定をする場合はそのテーブル先も指定しなければならない  
  
=> WHERE (body LIKE ‘%テスト%’)となっていたので`users.body`にしなければならない。  
=> `scope :body_contain, ->(word) { where(‘posts.body LIKE ?', "%#{word}%") }`に変更。  
  
https://techtechmedia.com/join-error-solution/  
  
## 質問済み  
https://tech-essentials.work/questions/464  
  
## 感想
フォームオブジェクトを使用するのが初だったのでまず概念の部分から理解するのが大変で、searchメソッドの定義部分やscopeの書き方で出てきたメソッドも初見のものも多く、複雑だったので結構時間がかかりました。。アウトプットのボリュームも大きくなってしまいました。笑  
でも知識的にもたくさん学ぶことができたし、色々手を動かしてコードを変えてみたりしてどう動いているのか確認することが多かったので為になりました！  
今後余裕ができたら、検索機能以外のフォームオブジェクトの使い方もどこかで確認してみたいと思います。  
