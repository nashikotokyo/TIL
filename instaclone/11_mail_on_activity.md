# 11. メール通知の実装
### 内容
メール通知機能を実装してください。タイミングと文言は以下の通りとします
- フォローされたとき
![Image from Gyazo](https://i.gyazo.com/54082216ed23b2aa31cdf61e38790222.png)
- コメントされた時
![Image from Gyazo](https://i.gyazo.com/91f139a69df72e5174eda7e335099e08.png)
- いいねされた時
![Image from Gyazo](https://i.gyazo.com/cef7164bc671f9c99a11db0001556db3.png)

### 補足
- default_url_optionsの設定値はconfigというgemを使い定数として設定すること
- 今後定数に関してはconfigを使う方針とする

# 作業手順
- letter_opener_web(gem)を導入
- config(gem)を導入、 bundle exec rails g config:installで設定ファイルを生成
- bundle exec rails g mailer UserMailerでメイラーやその他必要なファイルの生成
- 開発環境の環境設定ファイル(config/environments/development.rb)にてメイラー関連の設定
  - メール配信方法(letter opener web)の設定(config.action_mailer.delivery_method = :letter_opener_web)  
  - host情報を定義しメイラー内でいちいち定義せず使えるようにする(config(gem)を使いconfig/settings/development.ymlに定数として定義)  
- letter_opener_webのルーティング設定(`http://localhost:3000/letter_opener`にて送信メールを確認できるようにする)
- メール機能の実装
  - app/mailers/application_mailer.rbのデフォルト部分編集
  - user_mailer.rbの作成(like_post, comment_post, followメソッドの作成)
  - メイラー（UserMailer）の呼び出し部分の定義。(comments, likes, relationshipsコントローラのcreateアクション内を追記、修正)
  - メールの本文(メイラービュー)を作成(app/views/user_mailer/like_post.html.slim, app/views/user_mailer/comment_post.html.slim, app/views/user_mailer/follow.html.slim）


# 学んだこと
## ActionMailer
Railsに組み込まれているメール送信機能のこと。Railsからメールを送信できる。  
[Action Mailer の基礎(Railsガイド)](https://railsguides.jp/action_mailer_basics.html)  
[Action Mailer でメール送信機能をつくる](https://qiita.com/annaaida/items/81d8a3f1b7ae3b52dc2b)
<br>

## letter_opener_web (gem)
擬似的にメールの受信や閲覧ができるようにするgem。  
メール送信機能はRailsのデフォルトでできるが、内容の確認はコンソールしか確認ができずそれだと面倒なので、これを導入すると実際のメールに似た読みやすい画面で表示できる。  
letter_opener_webの場合、ブラウザはデフォルトのブラウザに限らず、`localhost:3000//letter_opener`が開けるところなら、どこでもデモサイトのような画面を開いてくれる。   
[デモサイト](http://letter-opener-web.herokuapp.com/)  
[gem letter_opener を試してみる](https://qiita.com/tanutanu/items/c6193c4c2c352ac152ec)
<br>

## config (gem)
環境ごとの定数を便利に管理することができるようになるgem。  
アプリを開発していく中で本番環境と開発環境で値を変えたいといったケース(ホストの情報を管理したい時など)が出てくる。  
(例)  
開発環境・・・localhost:3000  
本番環境・・・techessentials.jp  
=> この場合`if Rails.env.development?`や`Rails.env.production?`などを使い条件分岐でhostを定義するのは管理しづらくあまり良くない。  
configを使い、config/settings以下のymlファイルに定数を定義しておくと以下のようにスッキリ取得ができる。
（例）`host = Setting.host`  
[gem configを理解する ~ configを使った定数管理の方法](https://qiita.com/tanutanu/items/8d3b06d0d42af114a383)  
<br>
## 開発環境の環境設定ファイルでhost情報を設定している理由
`config/environments/development.rb`で以下のように設定している。
```rb
config.action_mailer.default_url_options = Settings.default_url_options.to_h
```
メイラーは通常のコントローラとは異なっていて、メイラーのインスタンスは送られてくるリクエストについてのコンテキスト(ホスト情報とか)を持っていない。  
なのでメイラーのビューでURLを書く上では、`url_for`や`設定したroutes`を使ってurlを生成することはできるが、自分でホスト情報(:hostパラメータ)をメイラーのビューに明示的に指定する必要がある。  
ただ、`= users_url(host: "localhost:3000")`のように毎回わざわざ書くのは面倒であり、:hostに指定する値は環境ごとにそのアプリケーション内で共通なのでグローバルに利用できるように上記のように設定している。  
  
※`_pathヘルパー`は、この動作の性質上メイラービュー内では一切利用できない点に注意。メールでURLが必要な場合は、`_urlヘルパー`を使う。  
[Railsガイド Action MailerのビューでURLを生成する
](https://railsguides.jp/action_mailer_basics.html#action-mailer%E3%81%AE%E3%83%93%E3%83%A5%E3%83%BC%E3%81%A7url%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B)  
[Qiita action_mailer.default_url_optionsの意味](https://qiita.com/minoriinoue/items/393d61b854a34358d102)

## Action Mailer関連のメソッド
### ・UserMailer.withとは
[この参考サイト](https://qiita.com/annaaida/items/81d8a3f1b7ae3b52dc2b)ではUserMailer内のアクションにコントローラ側から必要情報を引数で渡しているが、今課題ではUserMailer.withを使っている。  
今回以下のように定義している。 ※いいね部分のみ例として抜粋。フォロー、コメントに関しても同じ考え方
```rb
# likesコントローラ

  def create
    @post = Post.find(params[:post_id])
    current_user.like(@post)
    UserMailer.with(user_from: current_user, user_to: @post.user, post: @post).like_post.deliver_later if current_user.like(@post)
  end
~ 略 ~
```
```rb
# メイラー(user_mailer.rb)

class UserMailer < ApplicationMailer
  def like_post
    @user_from = params[:user_from]
    @user_to = params[:user_to]
    @post = params[:post]
    mail(to: @user_to.email, subject: "#{@user_from.username}があなたの投稿にいいねしました")
  end
 ~ 略 ~
```
withに渡されるキーの値は、メイラーアクションでは単なるparamsになる。つまり、今回`with(user_from: current_user, user_to: @post.user, post: @post).like_post`と書いているので、メイラーアクション(like_post)内で`params[:user_from], params[:user_to]やparams[:post]`を使えるようになる。  
ちょうどコントローラのparamsと同じ要領。  
  
※コントローラの場合と同様、メイラーのメソッド内で定義されたすべてのインスタンス変数はそのままメイラービュー(メールのフォーマット)で使える。  
  
※実装の際、最終的にメイラービュー内や送信先アドレスや件名内で使いたい情報(今回でいうと、1.いいねした人、2.された人、3.いいねされた投稿 の情報)を先に確認し、コントローラ内のUserMailer.withで必要なパラメータを渡す。  
  
### ・deliver_laterメソッド
Active Jobによるメールキューにメールを登録できる。これにより、コントローラは送信完了を待たずに処理を続行できる。  

### ・mailメソッド 
mailメソッドでは通常のメールと同じようにto(メールの送信先アドレス)などを指定可能。  
[![Image from Gyazo](https://i.gyazo.com/549cfd3cb873d2a90dbc58f535ec5af8.png)](https://gyazo.com/549cfd3cb873d2a90dbc58f535ec5af8)  
[【Rails入門】Action Mailerのメール送信を初心者向けに基礎から解説](https://www.sejuku.net/blog/48739)  
  
※メイラーにはto:オプションに配列を指定することもできるがそれは避けた方が良い。受信者に「自分以外に誰に送信されたのか」がバレてしまうので。面倒でもループで回して一人一人にメールを送信すべき。  
これと同じような問題が起こってしまう。  
https://www.lancers.co.jp/news/info/21226/  
  
# 感想
letter_opener_webを使ったメール送信機能の実装(パスワードリセット関連の実装)は以前やったことあったので、思い出しながら復習できました。  
Railsガイドやその他参考サイトもわかりやすく、コードリーディングの理解に特に問題は無さそうです。  
深掘りした[参考サイト](https://qiita.com/fursich/items/bb75a06714bcad6a0afb)などもありましたが、今回特に深掘りはせず、概要と実装の大体の流れだけ掴んだ感じです。  
基本Railsガイドに載っていたのでそれさえあればなんとかなるかなと思いました。  
