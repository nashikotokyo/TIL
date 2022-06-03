# 09.プロフィール編集機能
### 内容
- プロフィールの編集機能を実装してください
### 補足
- 編集画面は/mypage/account/editというパスとする
- アバターとユーザー名を変更できるようにする
- アバター選択時（ファイル選択時）にプレビューを表示する
- image_magickを使用して、画像は横幅or縦幅が最大400pxとなるようにリサイズする
- 以降の課題でもマイページに諸々追加するのでそれを考慮した設計とする（ルーティングやコントローラやレイアウトファイルなど）
  
# 作業手順
- アバター用のカラム追加（bundle exec rails g migration AddAvatarToUsers）
- プロフィール編集用のルーティング追加(routes.rb)
- アバターアップロード用のcarrierwaveの設定(Userモデル、avatar_uploader.rb)
- アカウント編集のコントローラ作成(app/controllers/mypage/accounts_controller.rb, app/controllers/mypage/base_controller.rb)
- config/initializers/assets.rbでmypage.jsをmypage.cssコンパイル対象に追加する設定
- mypageのビュー(app/views/layouts/mypage.html.slim、appllication.htmlのような大元の、共通で使うビュー)の作成
- サイドバーのパーシャルの作成(app/views/mypage/shared/_sidebar.html.slim)
- プロフ編集画面の作成(app/views/mypage/accounts/edit.html.slim)
- mypage.jsとmypage.scssの作成
- ロケールファイルの追記、プロフィール編集画面へのリンク(ボタン)を表示、 プレースホルダーだった箇所を変更、未ログイン時にを考慮して修正、などなど
  
# 学んだこと
## namespaceについて
- 今回、マイページに編集画面だけではなく他にも様々な機能を追加する予定なので、namespaceを使い、マイページ専用のコントローラーやファイル構造でビューを作成した方がいい。
```
#confin/routes.rb

Rails.application.routes.draw do
  namespace :mypage do
    resource :account, only: %i[edit update]
  end
end
```

　※ `resource :account`と単一リソースで設定している。アカウントはログインユーザーにとって一つしか存在しないから単一リソースを使用する。  
( https://railsguides.jp/routing.html#コントローラの名前空間とルーティング )  
  
- まずは、mypageで区切られる名前空間のコントローラに共通する親コントローラ`Mypage::BaseController`を作成。こうすることでmypageの名前空間で区切られるコントローラに共通でやりたいことなどまとめたりできるので便利。  
プロフィール編集のコントローラ`Mypage::AccountsController`では`Mypage::BaseController`を継承するようにする。  
  
- Railsのroutingにおけるscope / namespace / module の違い  
  
[![Image from Gyazo](https://i.gyazo.com/78bf5b43348eb322b3e9ca09c7cb5903.png)](https://gyazo.com/78bf5b43348eb322b3e9ca09c7cb5903)  
  
( https://qiita.com/ryosuketter/items/9240d8c2561b5989f049 )  
  
## config/initializers/assets.rbでの設定について  
assetに関する追加設定などができるファイル。  
  
`Rails.application.config.assets.paths << Rails.root.join('node_modules')`  
=>これは探索パスの設定。デフォルトのassetsディレクトリ以外のディレクトリを追加できる。初期状態ではyarn(パッケージマネージャー)のnode_modulesディレクトリが追加されるようになっている。  
  
( https://railsguides.jp/asset_pipeline.html#アセットの編成 )  
  
`Rails.application.config.assets.precompile += %w( mypage.js mypage.css )`  
=>プリコンパイルするファイルを配列で追加指定できる。デフォルトでは`application.js`、`application.css`、app/assets配下のJS、CSS以外の全てのファイルがデフォルトで対象となっているのでそれ以外のファイルをプリコンパイル対象としたい場合にこの設定をいじる。  
今回は `mypage.js`と`mypage.css`を追加したいので上記のように追記する。  
  
( https://railsguides.jp/asset_pipeline.html#アセットをプリコンパイルする )  
  
## javascript_include_tagとstylesheet_link_tag  
css, jsファイルをインポートするためのメソッド。  
`application.html`で`application.scss`, `application.js`をインポートするとき、今回だと`mypage.html`で`mypage.scss`, `mypage.js`をインポートするときに使っている。  
```
= stylesheet_link_tag 'mypage', media: 'all'
= javascript_include_tag 'mypage'
```
  
( https://railsguides.jp/action_view_helpers.html#javascript-include-tag )  
  
## yieldメソッド
共通プログラムと個別プログラムで分割してコーディングするために使用するRubyのメソッド  
 
` application.html.erb(今回はmypage.html) → HTMLやHEADタグなどの共通レイアウト`  
`アクション名.html.erb(今回はmypage/accounts/edit.html) → BODYタグ内の個別レイアウト` => bodyタグ内でyeildメソッドを使う。  
   
( https://railsguides.jp/layouts_and_rendering.html#yieldを理解する )  
  
## application.jsで//= require_tree .しない方がいい  
application.jsで`//= require_tree .` とすると同じ階層以下すべてのJSファイルを読み込む。  
読み込む順序は指定できない。そのため、利用しているJSファイルが特定の読み込み順に依存している場合に不都合がある。  
今回、application.jsと**同じ階層**に`mypage.js`を作っている。application.jsでrequire_tree .してしまうと、application.jsでmypage.jsを読み込んでしまう 。つまり、**mypage画面でしか使わないはずのファイルまで、app/assets/javascripts/application.jsで読み込んでしまう**。それを避けるために、require_tree .は使わずに、必要なファイルを個別に必要な順序で読み込む必要がある。  
  
## パーシャルに渡す変数をインスタンス(@user)ではなくf.objectを使っている件  
今回プロフ編集画面でエラーメッセージのパーシャルを以下のようにrenderしていた。  
```
# app/views/mypage/accounts/edit.html.slim
= form_with model: @user, url: mypage_account_path, local: true do |f|
  = render 'shared/error_messages', object: f.object
```
今までは `object: @user`としていた。挙動はどちらも変わらないが、 `object: @user`だと引き渡す変数はなにか毎回考える必要があるが、`object: f.object`とすると、ただ、コピペするだけで使いまわせる。  
( https://tech-essentials.work/questions/121 )  
  
## アバター画像のプレビューの作り方に関して  
- **プレビューを表示する** = **まだデータベースに保存していないアバターデータに対して選択された画像ファイルを表示**するということ(そこでJavaScriptが必要)  
  
- **イベントハンドラ** => イベント(ユーザーによるボタンのクリックやメッセージキューによるメッセージ受信など) が発生したときにJavaScriptなどで特定の処理を与えるための仕組み。  
今回で言うと`onchangeがイベントハンドラ`、`previewFileWithId()はイベントハンドラ名`で、`画像ファイルの選択がイベント`に該当する。  
  
**<プロフィール編集画面>**  
```
app/views/mypage/accounts/edit.html.slim

= form_with model: @user, url: mypage_account_path, local: true do |f|
  = render 'shared/error_messages', object: f.object
  .form-group
    = f.label :avatar 
    = f.file_field :avatar, onchange: 'previewFileWithId(preview)', class: 'form-control', accept: 'image/*' ...①
    = f.hidden_field :avatar_cache ...②
    = image_tag @user.avatar.url, class: 'rounded-circle', id: 'preview', size: '100x100' ...③
  .form-group
    = f.label :username
    = f.text_field :username, class: 'form-control'
  = f.submit class: 'btn btn-primary btn-raised'
```
  
① アップロード画像の選択フィールド部分について
```
= f.label :avatar
= f.file_field :avatar, onchange: 'previewFileWithId(preview)', class: 'form-control', accept: ‘image/*’
```
=> ユーザーがファイルの選択操作をした後には、`changeイベント`が発生する。changeイベントを捕えるには、`onchangeイベントハンドラ`を使う。ここに別途mypage.jsで定義した処理previewFileWithIdを設定しておく。引数にpreview(画像表示フィールド部分のid: ‘preview')を渡している。(←後ほど下で解説)  
accept: ‘image/*’でファイルの種類を指定。  
  
( https://developer.mozilla.org/ja/docs/Web/API/GlobalEventHandlers/onchange )  
  
② **= f.hidden_field :avatar_cache**について  
=>`hidden_field`は非表示のフォームを作成(隠しフィールドの生成)する。  
アップロードに失敗してrenderされた場合でも、画像をキャッシュしておいてファイルが消えないようにできる。**コントローラのストロングパラメータに:avatar_cacheを追加する**のも忘れない。  
※プレビューの処理とはまた別件で、carrierwaveを使っているとできる機能  
  
( https://github.com/carrierwaveuploader/carrierwave#making-uploads-work-across-form-redisplays )  
  
③ 画像表示部分について  
```
= image_tag @user.avatar.url, class: 'rounded-circle', id: 'preview', size: '100x100'**
```
=> id: 'prevew'はJavaScript(mypage.jsのpreviewFileWithId())と紐づけるため設定。  
(プレビュー画像を表示する処理をどこに対して与えるか教えるための目印)。  
@user.avatar.urlのパスでは、画像ファイル選択前はデフォルト画像が表示される。ファイルが選択されたらイベントハンドラが起動して選択した新しい画像がこの部分に表示される。  
  
★呼び出したmypage.jsのpreviewFileWithIdで定義した処理を実行させることで選択された画像ファイルでブラウザのDOMを差し替えることができる。  
  
## プレビュー画像を表示するjsファイルの書き方  
application.jsは、個別のCSSやJavaScriptファイルをどう管理していくか記述していくファイル(マニフェストファイル)なので、コードを記載しない。別途**mypage.js**を作成する。  
```
# mypage.js

//= require jquery3
//= require popper
//= require rails-ujs
//= require bootstrap-material-design/dist/js/bootstrap-material-design.js

function previewFileWithId(id) {     ...①
  const target = this.event.target;  ...②
  const file = target.files[0];      ...③
  const reader  = new FileReader();  ...④
  reader.onloadend = function () {   ...⑤
    id.src = reader.result;          ...⑥
  }
  if (file) {
    reader.readAsDataURL(file);      ...⑦
  } else {
    id.src = "";
  }
}
```
  
① ユーザが画像が選択したら`onchange: 'previewFileWithId(preview)`の設定によりこのjsファイルのpreviewFileWithIdメソッド(id='preview'が引数に渡されている)が実行される。  
② イベント元のオブジェクト(クリックした箇所のオブジェクト要素)を読み込んでtargetに代入している。  
　今回は、**file_fieldで生成されたinputタグ**を取得している。  
③ 選択された画像の**fileオブジェクト**を取得しfileに代入  
④ **FileReaderオブジェクト**を作成、readerに代入。  
⑤ **onloadend**ではfileオブジェクトが⑦のreadAsDataURL(file)で読込完了したら発火する処理を書いておく。  
⑥ ⑦のreadAsDataURL(file)での読込完了後に**result属性**に格納されている新しく読み込んだファイルデータの文字列で、id (id='preview')の**src**を書き換えている。つまり新しく選択した画像がプレビューとして表示される。  
⑦ もしfileオブジェクトがあれば(画像ファイルが選択されていたら)、**readAsDataURL**でそのfileオブジェクトを読み込み、**動作が終了すると読み込み終了と認識し⑤のreader.onloadendが発動**する。**読み込んだファイルデータを表す文字列(data: URL&nbsp;の文字列)はresult属性に格納される。**  
もし画像ファイルが選択されてなければsrc属性を空にしている。  
  
( https://javascript.keicode.com/newjs/how-to-read-file-with-file-api.php#1-4 )  
( https://lab.syncer.jp/Web/API_Interface/Reference/IDL/FileReader/onloadend/ )  
( https://lab.syncer.jp/Web/API_Interface/Reference/IDL/FileReader/readAsDataURL/ )  
  
## HTML内でIDをつけた要素はJavaScriptのグローバル変数に格納される  
**id属性**に値を設定すると、jsでグローバル変数(プログラムのどこからでもアクセスができる変数)になる。  
```
~ 略 ~
  = f.label :avatar
    = f.file_field :avatar, onchange: 'previewFileWithId(preview)', class: 'form-control', accept: 'image/*'
    = f.hidden_field :avatar_cache
    = image_tag @user.avatar.url, class: 'rounded-circle', id: 'preview', size: '100x100'
  .form-group
~ 略 ~
```
id: ‘preview’とすると別の要素内のidでもpreviewFileWithId(preview)のように引数として渡せる。  
  
( https://qiita.com/nakajmg/items/c895105afae95bfa8fae )  
( https://tech-essentials.work/questions/157 )  
  
# 感想
carrierwaveはインスタクローン課題では使うのが2回目なので少しずつ慣れてきた気がします。  
プレビュー機能の実装は以前1度やったことがありましたが、その時はjsファイルの中身をじっくり調べて理解するところまでいかなかったので、今回はmypage.jsのファイル内でどのようなことが起こっているのかをちゃんと確認できたのでよかったです。(あくまで、このファイル内で使われているメソッドやオブジェクトなどについて調べた程度なので、JavaScript自体に慣れたかと言われればまだわからないことだらけと言う感じですが・・)  
あと、namespaceも以前admin機能の実装でやったことがあったので、細かい実装方法までは覚えてなかったですが、コンセプトはわかっていたので進めやすかったです。admin以外の使い方を知れてよかったです。  
