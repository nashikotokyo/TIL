# 01. ログイン機能の実装
### 内容
- ログイン機能を実装してください。
- その他初期設定を行ってください
  - generateコマンド時に生成されるファイルを制限する
    - ルーティング、JS、CSS、テストが自動生成されないようにする
  - タイムゾーンの設定
  - etc
  
### 補足
- git-flowをhomebrewで導入しGitフローでの開発フローとする( https://github.com/DaichiSaito/insta_clone/wiki/git-flow%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6 )
- Rubyのバージョンは2.6.4とする(解答例が2.6.4なので。本来は新しいバージョンを使用すべき。)
- Railsのバージョンは5.2.6とする(解答例は5.2.3だがmimemagicのエラーが出るので5.2.6が望ましい)
- DBはMySQLとする
- turbolinkとcoffee-scriptは使わないようにする
- slim-railsを導入してビューテンプレートはslimを使う
- sorceryを導入してログイン機能を実装する
- rubocopを導入してLintチェックを行う
  - .rubocop.ymlは以下をダウンロード・展開したものを使う。ファイル名の先頭に「.」をつけるのを忘れないこと。
    (https://github.com/DaichiSaito/insta_clone/files/3991350/rubocop.yml.zip)
- redis-railsを導入してセッションの管理方法をクッキーストアではなくredisにする
- rails-18nを導入して国際化に対応する（メンタリングの都合でモデル名, カラム名のみ国際化対応することとする）
- annotateを導入してモデルが作られるたびに自動的にスキーマ情報がファイルに記載されるようにする
- better_errorsを導入してエラー画面を使いやすくする
- binding_of_callerを導入してエラー画面を使いやすくする
- pry-byebugを導入してデバッグ可能な状態にする
- pry-railsを導入してデバッグ可能な状態にする
- bootstrap material designを導入（gemだとうまく動かないのでyarnで導入）してビューを整える

# 作業手順
以下のような流れで作業しました。

- git-flowをインストールしてプロジェクトの初期化をし、featureブランチ(feature/01_login_logout)を作り一度リモートにpush(initial commit)し、開発を進める。
- slimの導入
- rubocopの導入 & [Fix] rubocopの指摘修正
- better_errorsとbinding_of_callerを導入
- pry-railsとpry-byebugを導入
- Material Design for Bootstrapを導入
- redis-railを導入しsession_storeをredis_storeに変更
- annotateの導入と設定ファイルの作成
- generators時に不要なファイルを生成しないよう設定
- gitignoreにvendor以下を管理しないよう追記
- タイムゾーンの設定
- rails-i18nの導入と基本設定
- sorceryの導入
- userモデルのvalidationを設定
- routesの定義
- bundle exec rails g controller usersでusersコントローラを作成, new createアクションを作成
- flashメッセージ関連を作成
- 新規ユーザ登録画面new.htmlを作成（form_with, bootstrap, slimの書き方等復習）
- error messageのパーシャルの作成とrenderで呼び出しの設定
- bundle exec rails generate controller user_sessionsでログインログアウトのコントローラとビュー作成
- user_sessions_controllerでログインログアウトのアクションの記載
- app/views/user_sessions/new.html.slimでビューの記載
- i18n（config/locales/ja.yml）の設定
- ログイン前とログイン後のヘッダの表示の設定(部分テンプレートを2パターン用意してapplication.htmlで切り替えする、navbarの使い方など)
  
# 学んだこと
## git flowとは
=>Gitの機能であるブランチを活用したGitの開発手法でもあり、ツールの名前でもある。チームで開発をおこなう場合、運用ルールを決めずにGitを採用してしまうと、コンフリクトが頻繁に起こったりマージのミスが発生したりといった問題が起キルが、それを回避し、最大限にGitを活用することができる。
developから作成され、開発者が直接コードを修正してコミットするブランチ。次のような流れで開発をする。  
基本的に以下の流れで進める  
`1. developブランチからfeatureブランチを作成する`
`2. featureブランチで機能を実装する  `
`3. GitHubにプッシュし、developブランチに対してプルリクエストを送る  `
`4. レビューを受けてdevelopブランチにプルリクエストをマージする`
  
## turbolinkとは
=>ページ遷移をAjaxに置き換えることで、JavaScriptやCSSのパースを省略することで高速化するgemのこと。Railsにデフォルトで入っている。turbolinksを使うことで、ページ遷移する際に通常のページ移動ではなく、ajaxで取得したhtmlでbodyを入れ変えることでページ遷移を実現してくれ普通のページ遷移より素早く遷移ができるのが利点。  
しかしturbolinksを使うと(document).ready()などが発動しないといったこともあり、一部の人に煙たがられているらしい。  
turbolinksをデフォルトでオフにする場合は以下のようにrails newをする。  
`rails new appname --skip-turbolinks`
  
## slimとは
=>slimはRubyで使うテンプレートエンジンの一つ。htmlよりもコードを早く綺麗に、そして非常にシンプルに作ることができる。またerbと同じでRubyのコードを埋め込むことができる。  
まだ慣れていないので以下の変換サイトを利用してみました。  
https://erb2slim.com/  
  
## sorceryとは
Railsに認証機能の実装を行うためのライブラリです。 似たようなライブラリにdeviseなどがあるがsorceryの方がシンプルで、カスタマイズ性に富んでいるらしい。  
Userモデルに`validates :password, confirmation: true`と記載することでsorceryに、それがcrypted_passwordカラムの内容であることを認識させで、passwordというDBに存在しない仮想的な属性(virtual attributes)が追加され「仮想的な」passwordフィールド(passwordとpassword_confirmation)を使用できるようになり、データベースに暗号化される前のパスワードをビューで扱えるようになる。※このpassword属性はカラムに対応していないため、平文のパスワード情報がDBに保存されることは無い。  
  
よく使うメソッド  
`loginメソッド`…認証処理が行われる。  
@user = login(params[:email], params[:password])でemailによるUser検索、パスワードの検証を行い、正常に処理できるとセッションデータにUserレコードのid値を格納する、という処理が行われている。  
  
`logoutメソッド`…セッションをリセットする。  
redirect_back_or_toメソッド・・・例えば、掲示板ページにアクセスしようとしたユーザにログインを要求する場合、require_loginメソッドでユーザをログインページに誘導し、ログインが成功したら、最初に訪れようとしていた掲示板ページにリダイレクトさせるということが可能になる。  
  
`auto_login(user)`…その名の通りオートログイン。メールアドレスやパスワードを使わずuserとしてログインする。  
  
## rubocopとは
- Rubyのコーディングチェックツールです。誰でも簡単にソースコードの品質を維持できるようになる。  
- bundlerでインストールしたgemをコマンドラインで使う場合、文頭にbundle execを付ける必要がある。`% bundle exec rubocop`
- rubocop導入の際は  
```
gem 'rubocop', require:false
gem 'rubocop-rails', require:false
```
と入れておく！  
bundlerでは、Gemfileに書いたgemをまとめて自動でrequireする仕組みになっているがrubocopは、ソースコードが規約に沿っているか確認するために、ターミナル等でコマンドを実行するのでbundlerによってアプリ側に自動で読み込む必要がない。よって、require: falseにする。  
  
`bundle exec rubocop -a`でオートコレクトしてくれる
  
## redisとは
=> データをメモリ上に保存するタイプのインメモリ型のKVS(Key Value Store)。  
KVSとは、主としてキーとバリューのシンプルなデータを保存するタイプのデータベースのことで、RDB(Relational Database)のような複雑なデータは扱えない反面、高速に動作するという特徴がある。  
KVSはデータの永続化を目的とする場合は向いていない。※Redisは永続化の機能があるので可能  
  
RailsでRedisを使うメリットデメリット  
**<メリット>**
- キャッシュ(Rails.cache)、Ruby on Rails のセッションデータなど一時的なデータ保存先としてRedisを利用するとRailsを高速化できる  
- セッションデータの場合は、CookieからRedisへ変更することで情報漏洩などのセキュリティリスクを軽減できる  
  
**<デメリット>**
- インメモリDBは高速だがCookieに比べると処理が遅い  
- Redisを導入するコストがかかる  
- Redisに障害が発生すると、セッション機能が使えなくなる  
  
## annotateとは
=> 各モデルのスキーマ情報やルーティング情報をファイルの先頭もしくは末尾にコメントとして書き出してくれるGem。いちいちschema.rbを見たりrails routesを打たなくてもよくなる。  
  
## i18nとは
=> Railsのアプリケーションを多言語化してくれるもの。  
まずデフォルトで使用する言語をconfig/application.rbに設定し、gem 'rails-i18n'をインストールする。(gem導入によRailsを日本語で使う場合のデフォルトのロケールファイル「svenfuchs/rails-i18n」をダウンロードしなくても使えるようになる)　  
config/locales以下の複数のロケールファイルが読み込まれるようpathを通し、config/locales以下にロケールファイルを配置し日本語を設定する。そして設定した翻訳をビュー、モデル、コントローラ等で表示させる。  
ロケールファイルの設定内容の量などによってファイルをビュー、モデル、コントローラに分けることもできる。  
  
## database.ymlとは
 =>Railsでデータベースに接続するための情報を記載したファイル。rails new <アプリ名>でrailsのプロジェクトを新規作成した時に、configディレクトリ配下に自動生成される。デフォルトのDBはsqlite。  
 DBを指定するときは以下のようにする。  
`rails new <アプリ名> -d mysql`  
  
default, development, test, productionに分かれて情報が記載されている。  
   
## migrationファイルとschema.rbとは？
=>マイグレーションファイルは、データベースを生成する際の設計図になる。  
どのようなカラム(カラム名、型)を持ったテーブルを作成するか等を記述する。  
`rails db:migrate`を打ち込むとマイグレーションファイルを実行し記述した内容に基づいたデータテーブルが生成されschema.rbにテーブルデータとして反映される。schema.rbには最新のDBの状態が反映されている。  
  
`rails db:migrate:status`でマイグレーションファイルの状態を確認。downは反映されていない、upは反映済みの状態  
  
`rails db:rollback`で最新のマイグレーションファイルのバージョンが upからdownになり、データベースがmigrateされる前の状態に戻る。修正が必要な場合はマイグレーションファイルを修正し、再度`rails db:migrate`コマンドを実行するという流れ。  
  
モデル側にuniquenessバリデーションを書いているカラムに対しては、DBのマイグレーションファイルにunique indexをつけた方がいい。  
=>確実にユニークにするためと、パフォーマンスの問題。  
(https://pocke.hatenablog.com/entry/2020/02/18/015421)  
  
## config/application.rbとは
=>アプリ全体の設定を記述するファイル。タイムゾーンの設定や、デフォルトで使用する言語、アプリ起動時に読み込むディレクトリ、rails generateで生成するファイルの設定など。  
  
## yarnとは
=>主にJavaScriptで開発されたプログラム部品（モジュール）を管理するためのパッケージ管理システムの一つ。npmと互換性があり、乗り換えたり併用することができる。  
Rubyにおけるgem(RubyGems)、MacにおけるHomebrewのようなもの  
  
**<開発時の流れ>**
**最初にプロジェクト作成する人**  
- yarn init でpackage.jsonを作成  
- yarn add {パッケージ名}で本番用パッケージをpackage.jsonに追加&yarn.lockにバージョン情報追加  
- yarn add --dev {パッケージ名}で開発用パッケージをpackage.jsonに追加&yarn.lockにバージョン情報追加  
- package.jsonとyarn.lockをプッシュして共有  
  
**プロジェクトに参加したメンバーたち**
- yarn intall でみんなと同じパッケージをインストール  
※この時、各パッケージはyarn.lockに記述されているバージョンがインストールされるため、チームメンバーと同一バージョンをインストール可能  
  
**新しいパッケージを入れるとき**
- yarn add {パッケージ名} でインストールしてローカルで動作確認  
- package.jsonとyarn.lockをプッシュして共有  
  
# 感想
最初環境構築や初期設定も何から手をつければいいか忘れてしまっていた状態だったので自力で書くことは難しかったですが、解答例を見つつ、1つ1つ復習しながら思い出しながら進められました。  
  
今まで使ったことのない便利なgem(annotateやbetter_errorsとbinding_of_caller)や、material design for bootstrap、redisなどがあり、1つ1つ調べて導入するのには時間がかかりましたが今後使えるようになるのでよかったです。  
