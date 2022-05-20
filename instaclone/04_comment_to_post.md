# 04.コメント機能
### 内容
- 投稿に対するコメントのCRUD機能を実装してください。

### 補足
- 編集・更新はモーダルを表示させ非同期で行う。form_withを利用すること。
- 文字列長など適切なバリデーションを付与する
- shallowルーティングを使用する
  
# 作業の手順
- コメントモデルの作成、マイグレーションファイルの作成、実行
- post, userモデルのアソシエーション設定、commentモデルのバリデーション設定
- routesに追記 (commentをshallowオプションにする)
- commentsコントローラの作成
- コメント入力部分の作成(createアクションとcomment_paramsメソッド、_form.htmlの作成、_show.html修正、post#showアクションの修正など)
- コメント一覧表示部分の作成（create.js.slim、_comments.html、_comment.htmlの作成、show.html、postコントローラshowアクションの修正など)
- コメント編集部分の作成(commentsコントローラeditアクションの追加、_comment.htmlの修正、_modal_form.html.slim、_modal.html.slim、edit.js.slimの作成、application.htmlの追記など)
- コメント編集更新の作成 (commentsコントローラupdateアクションの追加、update.js.slimの作成など)
- コメント削除部分の作成(commentsコントローラdestroyアクションの追加、destroy.js.slimの作成など)
- i18nの更新(ja.ymlにコメント関連の記述を追加)
- commentコントローラのログイン制御(require_login)
  
# 学んだこと
## 浅いネスト shallowオプション

今回、commentコントローラのcreateアクションはpostのidの情報もパラメータに必要なので` /posts/:post_id/comments`のようなルーティングが必要。  
また、edit/update/destroyアクションは、postの情報は必要なくcommentのidのみあればいいので`/comments/:id/edit`や `/comments/:id`のようなルーティングが作れればいい。  
それを実現するのがshallowオプション  
  
※今回使っていないindex/newアクションに関してもcreateと同様に親のスコープの下でルーティングが生成される。  
showアクションはedit/update/destroyアクションと同様に浅いネストで生成される。  
  
参考: [Railsガイド](https://railsguides.jp/routing.html#浅いネスト)  
  
## ストロングパラメータのmergeメソッド

commentコントローラのcreateアクションで使われているcomment_paramsでは、パラメータから送られてきたpost_idの値(params[:post_id])を取得しmergeメソッドでストロングパラメータに追加している。  
  
これの理由: コメントをcreateする際にpost_idカラムにコメントに紐づくpost_idの情報を保存したいが、、その値は別にフォーム上でユーザーに記入してもらうわけではない（ユーザーからフォームにのっとってパラメーターで送信されてくるわけではない）ので、こちら側でparamsから引っ張ってきて登録してあげる必要がある。  
mergeメソッドを使えば、直接的にユーザーから受け取った情報にはないが、レコード作成時に追加したい値を合わせて処理してくれる。  
```
class CommentsController < ApplicationController
  before_action :require_login, only: %i[create edit update destroy]

  def create
    @comment = current_user.comments.build(comment_params)
    @comment.save
  end
　 〜略〜
  
　def update
    @comment = current_user.comments.find(params[:id])
    @comment.update(comment_update_params)
  end
  
  〜略〜
  
private

  def comment_params
    params.require(:comment).permit(:body).merge(post_id: params[:post_id])
  end

  def comment_update_params
    params.require(:comment).permit(:body)
  end
end
```
一方で、commentコントローラのupdateアクションで使われているcomment_update_paramsについては、updateアクションで取得した@commentにすでにpost_idの情報はあるのでmergeメソッドでpost_idの情報を追加する必要がない。  
  
参考: https://qiita.com/sa_tech0518/items/e63f409c8d16daea9cf7  
  
## _comments.html.slimと_comment.html.slimのパーシャルに分けている理由  
  
質問もさせて頂いたが、解答例のコードで、一度_comments.html.slimを介して_comment.html.slimを作成している理由として、comments一覧全体に対して何か記述を加えたい場合（例えば、「コメント一覧」という記載があった後に、各コメントが連なってほしい場合など）に、綺麗に表現をしやすい点、また各ファイルの見通しをよくするためという点が挙げられた。  
  
参考: https://tech-essentials.work/questions/459  
  
## remote: trueでのAjax対応  
  
**ajax(非同期通信)と同期通信とは...**  
  
=>同期通信だとクライアントからサーバーに対してページ全ての情報を返すようリクエストが送られているためリクエストを送ったクライアントは、サーバーからの応答があるまでその結果を待機し結果を受け取った後に画面全体を切り替える処理を行う。  
=>非同期通信では、クライアントからサーバーに対してページの一部の情報を返すようリクエストが送られているためクライアントからのリクエスト送信後、サーバーの処理中にも、他の作業を行うことができる。  
つまりユーザーに処理待ちの時間を与えずに、裏では随時通信が行われているといったサービスの設計が可能となる。  
  
**remote: true**
  
link_toメソッドやform_withに`remote: true`オプションを追加することによりajax通信を実装可能。  
=> remote: trueとすると、リクエストを送信する際にHTML形式ではなく`JS形式`で送信する。  
その場合にレンダリング処理で呼び出されるビューファイルは`アクション名.js.erb(slim)`となる。  
このjs.erb(slim)ファイル内でclassやidで場所を指定し、「この部分だけ色を変える」「この部分だけ描画し直す」といった処理をjs(とruby)で記述することで、ajax処理を行うことが可能となる。  
  
参考: https://railsguides.jp/working_with_javascript_in_rails.html  
  
## alert()メソッド  
  
jsのメソッド。`alert( 画面に表示させたい値 )`で以下のようポップアップ画面を画面上部に表示させる。  
[![Image from Gyazo](https://i.gyazo.com/dbd977bcd51429d01aebb3ef3bc91054.png)](https://gyazo.com/dbd977bcd51429d01aebb3ef3bc91054)  
  
参考: https://techacademy.jp/magazine/5486  
  
## joinメソッド  
  
配列の要素を繋げて文字列に変換することができる  
  
参考: https://docs.ruby-lang.org/ja/latest/method/Array/i/join.html  
  
※以下のファイルを最初、`alert("#{@comment.errors.full_messages}");`のように書いて実行したら文字化けがアラート画面に出た。  
```
#create.js.slim

- if @comment.errors.present?
  | alert("#{@comment.errors.full_messages.join('\n')}");
〜略〜
```
文字化け↓  
```
[&quot;コメントを入力してください&quot;]
```
=> これは@comment.errors.full_messagesには配列で`[“コメントを入力してください”]`と入ってるのでそれが化けた形。(※「`&quot;`」は「`”`」の文字化けらしい。)  
errors.full_messagesは配列なのでalertの引数で使用する際、join(‘\n’)することで各要素に改行を追加し文字列に変換している。そのため文字化けになっていない。  
  
## $の意味  
  
JavaScriptにおいて「$」はなんら特別の意味は持たない。慣例的にjQueryのコードでは、ある変数がjQueryオブジェクトへの参照を代入する変数名であることをわかりやすくするため、$を接頭辞として使っているに過ぎない。  
  
参考: https://qiita.com/weedslayer/items/57b6ed8643395c95c258  
  
## prepend()メソッド  
  
セレクタで指定した要素（やクラスやid）内の子要素の先頭に引数で指定した要素（や文字列など）を追加するjsのメソッド。部分テンプレートを呼び出して、ページの一部を更新する事も出来る。  
これと対になるのが`append()メソッド`  
  
参考: https://www.sejuku.net/blog/46746  
  
## escape_javascriptメソッド  
  
`アクション名.js.erb(slim)`ファイルの中で部分テンプレートをrenderする際に’’、""があるとうまく実行されないのでjavascriptではエスケープが必要。それを解決するためのメソッド。  
`j`はescape_javascriptのエイリアス。  
  
参考: https://qiita.com/techman/items/8807bb9eb40fd2663e1a  
  
## .val(‘’)
    
テキストボックスの value 値を空欄にする操作  
  
参考: https://www.sejuku.net/blog/45297  
  
## html()メソッド  
  
任意のHTML要素を取得したり意図的に要素を追加・書き換え(上書き)をすることができるjsのメソッド。  
  
参考: https://www.sejuku.net/blog/38267  
  
## remove()メソッド  
  
対象となる要素や子要素などを削除することができるjsのメソッド。  
  
参考: https://www.sejuku.net/blog/39300  
  
## アクション名.js.erb(slim)内で|(パイプ)を使う理由  
  
こちらみけたさんの過去の質問で書いてあり助かりました。  
※以下引用  
以下のようなjs.slimがあった場合(一部抜粋)  
```
- else
  / 該当のコメントを更新する
  | $("#comment-#{@comment.id}").html("#{j render('comments/comment', comment: @comment)}");
```
まずRails（Ruby）に関係するところが処理される。
```
| $("#comment-1").html("<p class=\\"yeah\\">いえーい！<\\/p>");
```
その後slimで解釈された結果、以下がtext/javascript形式でクライアントに送信される。  
なのでslim形式のファイルにおいてはパイプ（ これ→ | ）が必要になる。  
```
$("#comment-1").html("<p class="yeah">いえーい！</p>");
```
slim形式のファイルにおいては、通常 - や = といったものから始まることが多く、slim側では何らかの解釈を要するものが書かれている前提で読み込みが行われる。ただ、それでは単純なテキスト自体を書き込むことができなくなるので、その解釈をして欲しくない場合、パイプを使う。  
参考: https://tech-essentials.work/questions/146  
  
## モーダルウィンドウとは  
  
元のウィンドウの上に別枠で表示されるウィンドウのこと。指定された操作を完了、もしくはキャンセルするまでずっと表示され続け、他のウィンドウに移ることができない。  
  
- **この課題でのモーダルウィンドウの実装方法**  
今回の課題でのコメント編集のモーダルウィンドウの作り方、手順。  
- link_toのオプションで`remote: true`を記載、ajax通信が実装できるようにする。  
- commentsコントローラの追記。`edit`, `update`アクションを追加。  
- `_modal_form.html.slim`(モーダルウィンドウ部分のファイル)でコメント編集画面のビューを実装。  
　bootstrapの公式のテンプレートを見ながら実装できる。(https://getbootstrap.jp/docs/4.2/components/modal/)  
 `edit.js.html`でモーダルウィンドウを表示する操作をする目印のために`#comment-edit-modal`というidを設定しておく。  
- `views/shared/_modal.html.slim`に`#modal-container`とだけ書いて、他のモーダルウィンドウを作りたいときも対応できるようにモーダルウィンドウの箱を用意しておく。  
- `application.html`のmainの一番下に `= render ‘shared/modal’`として上記の_modal.htmlを呼び出しておく。(`edit.js.html`が実行されれば表示されるし、実行されなければ何も表示されないという作り)  
- `edit.js.slim`を実装する。  
```
#edit.js.slim
| $("#modal-container").html("#{j render('modal_form', comment: @comment)}");
| $("#comment-edit-modal").modal('show');
```
=>新しく入力された@commentを渡して`_modal_form.html.slim`をrenderしたものを`#modal-container`(モーダル箱のid)に対してhtmlメソッドを使って追加する。  
`#comment-edit-modal`(コメント編集画面のモーダルファイルのid)に対してmodal('show')メソッドでモーダルを表示させる操作を記述。  
=>つまりコメント編集アイコンが押されてeditアクションが実行されたらedit.js.slimが実行され、上記2点が実行されコメント編集モーダルが表示される。  
- `update.js.slim`を実装し編集したコメントが反映されるようにする。  
```
#update.js.slim
- if @comment.errors.present?
  | alert("#{@comment.errors.full_messages.join('\n')}");
- else
  | $("#comment-#{@comment.id}").html("#{j render('comments/comment', comment: @comment)}");
  | $("#comment-edit-modal").modal('hide');
```
=>編集された@commentを渡して`_comment.html.slim`(コメント表示のパーシャル)をrenderしたものをhtmlメソッドで`#comment-対象のコメントのid`に対して上書きする。  
`#comment-edit-modal`(コメント編集画面のモーダルファイルのid)に対してmodal('hide')メソッドでモーダルを閉じる操作を記述。  
  
# 感想
以前ajax対応を初めてやった時jQuery(JavaScript)周りが理解が追いつかなくてすごく不安があったが、今回いい復習になり、仕組みを理解することができたので少し安心した。  
jQuery(JavaScript)に付随してhtml, bootstrap関連のこともいろいろ調べたり理解を深めることができたのでよかったです。  
  
モーダルの実装は初めてだったけど自分がwebサイトなどでよく見る機能を1つずつ作れるようになっていく感覚はやっていて楽しいです。  
  
解答例を見つつ実装しましたが、全体的にrails関連のコーディングは割と答えを見なくても書けたり、正解に近いようなコードを導き出すことは出来たり、書けなくても解答を見れば理解できることがほとんどでしたが、ビューファイルはまだ自分ではなかなか同じように作るまでには及ばないし、読み解くのに時間がかかってしまうのでポートフォリオ作成の時少し不安です。  
なので、html, css周りのコードも写経するだけでなく、調べつつ、また実際に画面上での動作の変化などを手を動かして確認するようにしています。  
