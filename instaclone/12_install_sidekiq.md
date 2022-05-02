# 12.メールのジョブの永続化を実装
### 内容
- メールのジョブを永続化できるように実装してください。  
- ActiveJobのアダプターにはsidekiqを利用してください。  
  
### 補足
- ダッシュボード用にsinatraもインストールする  

# 作業手順
- Sidekiqの導入(Gemfile)、設定(config/application.rb, config/initializers/sidekiq.rb, )
- sinatraの導入(Gemfile)、設定(routes.rb)  
  
[gem Sidekiqの導入](https://qiita.com/tanutanu/items/dabd550e3f969f72d64c)  
[Railsで非同期処理を行える「Sidekiq」](https://qiita.com/yumiyon/items/6835d90e621e73268021)  
  
  
# 学んだこと
そもそもメールジョブの永続化とは？？ActiveJob?？sidekiq？？という状態だったので言葉や概念を理解するところから始めた。  
  
## ジョブ(Job)とキュー(Queue)とは  
`ジョブ`: コンピュータが行う1つ1つのタスクのようなもの  
`キュー`: タスクを登録するための入れ物のようなもの。キューの特徴は 先に登録されたものから先に実行する(先入れ先出し)という点。  
  
  
## 非同期通信と同期通信について復習  
以前課題でajaxの時にやってある程度理解していたが、再度ざっと復習。  
  
[非同期処理とは？ 同期処理との違い、実装方法について解説](https://www.rworks.jp/system/system-column/sys-entry/21730/)  
  
  
## ActiveJobとは
- ジョブを宣言し、それによってバックエンドでさまざまな方法によるキュー操作を実行するためのフレームワーク。  
処理をアプリケーションの裏側(バックグラウンド)で実行させることができるので、ActiveJobは基本的にリアルタイム性を伴わない場合や重たい処理に使われることが多い。  
(例)  
・ メールの送信  
・ 画像の処理  
・ データを集計してCSVに落とす  
=>このような時間がかかる処理の場合には、先にレスポンスをクライアントへ返しておき、バックグラウンドで処理を別途実行する。  
  
- デフォルトのRailsは非同期キューを実装できるがActiveJobにはジョブをメモリに保持するインプロセスのキューイングシステムしかないので、プロセスがクラッシュしたり再起動したするとジョブは全て失われてしまう。  
開発環境や、小規模アプリケーションの場合、ミッションクリティカルでないジョブであればActiveJobのみでも問題ないが、多くの場合本番環境では永続的なバックエンドを選ぶ(ジョブの永続化)必要がある。  
その為、ActiveJobには非同期バックエンドなるサードパーティーのキューイングライブラリ(`Sidekiq`など)が必要。  
  
- ActiveJobの目的はRailsアプリにジョブ管理インフラを配置すること。  
ActiveJobを経由してジョブ実行機能のAPI(sidekiq, Delayed Job, Resqueなど)を利用することで、さまざまなの違いを気にせずにジョブフレームワーク機能やその他のgemを搭載することができるようになり、バックエンドでのキューイング作業では、操作方法以外のことを気にせずに済む。  
さらに、ジョブ管理フレームワークを切り替える際にジョブを書き直さずにコードの流用ができる。  
  
[Rails ガイド　- Active Job の基礎](https://railsguides.jp/active_job_basics.html)  
[Ruby on RailsのActiveJobとは?](https://qiita.com/petertakahashi/items/cb9ae73e5ba3020f4a89)  
  
  
## sidekiq(gem)
Rubyとは異なるプロセスをバックグラウンドで実行するためのキューイングバックエンド。上述の通りActiveJobだけではジョブの永続化ができないので導入する必要あり。  
キューを永続的に保存するため`Redis`を導入する必要がある。(よくRedisが使われている)。  
  
[Sidekiq - wiki](https://github.com/mperham/sidekiq/wiki)  
  
  
## Redisについて復習
以前セッションの管理の為にReidsを導入したが、　再度ざっと読んで復習。  
メモリ上で動作するデータベース(インメモリDB)では、メモリ上のデータは時間が経つと消えてしまうので、Redisを使うとジョブが消えてしまわないの？と思ったが、Redisではメモリ上のデータをストレージに格納してデータを永続的に保持する機能があるので永続化に使われている。  
  
[Redisとはどのようなデータベース？3つの特徴や使い方についても解説](https://www.fenet.jp/infla/column/database/redis%E3%81%A8%E3%81%AF%E3%81%A9%E3%81%AE%E3%82%88%E3%81%86%E3%81%AA%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%EF%BC%9F3%E3%81%A4%E3%81%AE%E7%89%B9%E5%BE%B4%E3%82%84%E4%BD%BF%E3%81%84/)  
[初心者による初心者のためのRedis解説](https://qiita.com/keinko/items/60c844bcf329bd3f4af8)  
  
  
## railsのsidekiqでredisが使われることの利点は？  
sidekiq等のジョブキューマネージャーは大量のジョブをさばくことが目的で、1秒間にさばけるジョブの数を重視していて、またジョブの管理の都合上、ジョブがどのような状態にあるかを頻繁に永続化する(=redisに書き込む)もの。  
そのような頻度の高い読込&更新オペレーションには、MySQLなどのRDBMSは適してなく、高速化のためにredisなどのKVSを用いている。  
  
[railsのsidekiqでredisが使われることの利点は？](https://ja.stackoverflow.com/questions/27035/rails%E3%81%AEsidekiq%E3%81%A7-redis-%E3%81%8C%E4%BD%BF%E3%82%8F%E3%82%8C%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AE%E5%88%A9%E7%82%B9%E3%81%AF)  
  
  
## sinatora(gem)
Railsのようなrubyのフレームワーク。軽量で、小規模なアプリケーションを作成するのに向いている。Railsのように「MVC」に基づいて作成されていない。  
Sidekiqには、ジョブの処理状況をGUIで確認できるインターフェース(ダッシュボード)が備わっている。そのインターフェースはsinatraで動いているため導入が必要となる。  
    
[Sinatraの特徴や仕組み・使い方を徹底解説！](https://agency-star.co.jp/column/sinatra)  
[Sidekiq(gem) - Monitoring](https://github.com/mperham/sidekiq/wiki/Monitoring)  
  
  
## ざっくりまとめると・・
今回でいうと、例えばコメントされた場合に通知が飛ぶ仕様だが、通知メール送信は時間のかかるタスクなのでそれを「後で処理するリスト」(キューのこと、Redisに保存される)に入れておき、先にコメント投稿の際の処理(コメント内容が表示される等)をクライアント側に返しておき、「後で処理するリスト」のメール送信のタスクも並列で行う。  
  
Rails、Redis、Sidekiqの関係のイメージ  
[![Image from Gyazo](https://i.gyazo.com/19c2e2235403a69f0f0932d4440e98d9.png)](https://gyazo.com/19c2e2235403a69f0f0932d4440e98d9)
  
Sidekiqを使ってバックグラウンドで非同期的にジョブを処理する、またジョブを永続化するためには、Railsサーバの他にRedisとSidekiqを起動しておく必要がある。  
Railsサーバの処理の中で「後で処理するリスト」に任せたいジョブがあれば、Redisに永続化しているキューにジョブを追加（エンキュー）し、Sidekiqがジョブを取り出して（デキュー）実行していく。  
これらのキュー操作を様々な方法で実行するためにインターフェースなどを定めてくれているフレームワークがActiveJob。  
Active JobはキューのアダプタとしてSidekiqをサポートしているのでActive Job + Sidekiq + Redis でバックグラウンドでの非同期処理を実現している企業も多い。　　
  
[Sidekiqってどんなキック？](https://dev.icare.jpn.com/dev_cat/sidekiq/)　　
  
  
## 感想
実装自体は導入し、設定するだけという感じだったので、概念的なことや言葉を調べることに時間を使いました。  
ただ、最初Railsガイドから読んだら、分からない言葉が多く(あるある)、そこから更に検索をして噛み砕いて説明しているサイトなどを見つけ、ざっくり理解できました。  
RailsガイドやGemの公式などをなるべく最初に読むようにした方がいいのだと思いますが・・まだそれだけでは心もとない感じです。  
