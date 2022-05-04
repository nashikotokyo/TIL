# 13. 通知の設定を実装
### 内容
- 通知をするかしないかをユーザーが設定できる機能を実装してください。  
  
### 補足
- マイページに通知設定というメニューを追加してください。  
- コメント時の通知メール, いいね時の通知メール, フォロー時の通知メールのオンオフを切り替えられるようにしてください。  
  
# 作業手順
- usersテーブルに、通知設定に必要な3カラムを追加(% rails g migration AddNotificationFlagsToUsersでマイグレーションファイル作成、実行)
- ルーティングの設定(resource :notification_setting, only: %i[edit update])
- サイドバーのパーシャルに通知設定部分を追加(app/views/mypage/shared/_sidebar.html.slim)
- 通知設定のビューの作成(app/views/mypage/notification_settings/edit.html.slim)
- mypage/notification_settingsコントローラの作成(edit, updateアクション)
- カラムの翻訳の追加(config/locales/ja.yml)
- 通知可否の判定ロジック追加(likes, raltionships、commentsのコントローラのcreateアクション)

# 学んだこと
## ○○○○ if A && B について
- 論理演算子`&&` ... A && B	AかつBが真のとき真、それ以外は偽を返す
- 論理演算子のポイント
1. <左辺> [論理演算子] <右辺>の式は、左から順番に評価される  
2. 左辺を評価した時点で式全体の結果が決まらない場合のみ右辺が評価される  
3. 最後に評価された式が式全体の結果(計算の答え)になる  
  
今回、通知可否の判定のロジックの部分で論理演算子を使って以下のように定義していたので詳しく見てみた。  
※コメントの場合に絞って記載  
```rb
UserMailer.with(user_from: current_user, user_to: @comment.post.user, comment: @comment).comment_post.deliver_later if @comment.save && @comment.post.user.notification_on_comment?
```
↓ これを読み換えると以下のようになる。  
  
`メールの通知処理をする` `if` `コメントを保存(登録)する` `&&` `コメントされたユーザがコメントされた際の通知を許可している`  
  
左辺`コメントを保存(登録)する`がまず評価(実行)され、コメントが登録され(true)、評価が右辺に移る。  
右辺`コメントされたユーザがコメントされた際の通知を許可している`がtrueなら、式全体の結果がtrueになり`メールの通知処理をする`が実行される。  
※右辺`コメントされたユーザがコメントされた際の通知を許可している`がfalseで式全体がfalseの場合でも、左辺 `コメントを保存(登録)する`は評価されるのでコメントは登録されている。  
  
実際にどのような処理になっているか確認してみた。  
- コメントされたユーザがコメントされた際の通知をオフにしている場合の処理
```
    4: def create
    5:   @comment = current_user.comments.build(comment_params)
    6:   binding.pry
 => 7:   UserMailer.with(user_from: current_user, user_to: @comment.post.user, comment: @comment).comment_post.deliver_later if @comment.save && @comment.post.user.notification_on_comment?
    8: end

[1] pry(#<CommentsController>)> UserMailer.with(user_from: current_user, user_to: @comment.post.user, comment: @comment).comment_post.deliver_later if @comment.save && @comment.post.user.notification_on_comment?
   (2.1ms)  BEGIN
  ↳ (pry):3
  Post Load (3.6ms)  SELECT  `posts`.* FROM `posts` WHERE `posts`.`id` = 13 LIMIT 1
  ↳ (pry):3
  Comment Create (4.0ms)  INSERT INTO `comments` (`user_id`, `post_id`, `body`, `created_at`, `updated_at`) VALUES (12, 13, 'テスト', '2022-05-03 23:35:12', '2022-05-03 23:35:12')
  ↳ (pry):3
   (18.4ms)  COMMIT
  ↳ (pry):3
  User Load (2.1ms)  SELECT  `users`.* FROM `users` WHERE `users`.`id` = 11 LIMIT 1
  ↳ app/models/comment.rb:34
   (2.3ms)  BEGIN
  ↳ app/models/comment.rb:34
  Activity Create (3.4ms)  INSERT INTO `activities` (`subject_type`, `subject_id`, `user_id`, `action_type`, `created_at`, `updated_at`) VALUES ('Comment', 40, 11, 0, '2022-05-03 23:35:12', '2022-05-03 23:35:12')
  ↳ app/models/comment.rb:34
   (1.8ms)  COMMIT
  ↳ app/models/comment.rb:34
=> nil
```
=>コメントが登録され、通知レコードが作成されている。
  
- コメントされたユーザがコメントされた際の通知をオンにしている場合の処理
```
    4: def create
    5:   @comment = current_user.comments.build(comment_params)
    6:   binding.pry
 => 7:   UserMailer.with(user_from: current_user, user_to: @comment.post.user, comment: @comment).comment_post.deliver_later if @comment.save && @comment.post.user.notification_on_comment?
    8: end

[1] pry(#<CommentsController>)> UserMailer.with(user_from: current_user, user_to: @comment.post.user, comment: @comment).comment_post.deliver_later if @comment.save && @comment.post.user.notification_on_comment?
   (3.1ms)  BEGIN
  ↳ (pry):4
  Post Load (3.3ms)  SELECT  `posts`.* FROM `posts` WHERE `posts`.`id` = 13 LIMIT 1
  ↳ (pry):4
  Comment Create (2.3ms)  INSERT INTO `comments` (`user_id`, `post_id`, `body`, `created_at`, `updated_at`) VALUES (12, 13, 'テスト2', '2022-05-03 23:55:18', '2022-05-03 23:55:18')
  ↳ (pry):4
   (1.8ms)  COMMIT
  ↳ (pry):4
  User Load (3.6ms)  SELECT  `users`.* FROM `users` WHERE `users`.`id` = 11 LIMIT 1
  ↳ app/models/comment.rb:34
   (16.9ms)  BEGIN
  ↳ app/models/comment.rb:34
  Activity Create (2.5ms)  INSERT INTO `activities` (`subject_type`, `subject_id`, `user_id`, `action_type`, `created_at`, `updated_at`) VALUES ('Comment', 41, 11, 0, '2022-05-03 23:55:19', '2022-05-03 23:55:19')
  ↳ app/models/comment.rb:34
   (2.0ms)  COMMIT
  ↳ app/models/comment.rb:34
[ActiveJob] Enqueued ActionMailer::Parameterized::DeliveryJob (Job ID: 79ef1847-1ebe-4f14-b3e3-6ffc01e376b0) to Sidekiq(mailers) with arguments: "UserMailer", "comment_post", "deliver_now", {:user_from=>#<GlobalID:0x00007f8f1e815448 @uri=#<URI::GID gid://insta-clone-by-me/User/12>>, :user_to=>#<GlobalID:0x00007f8f1e814e08 @uri=#<URI::GID gid://insta-clone-by-me/User/11>>, :comment=>#<GlobalID:0x00007f8f1e8147a0 @uri=#<URI::GID gid://insta-clone-by-me/Comment/41>>}
=> #<ActionMailer::Parameterized::DeliveryJob:0x00007f8f3cc57c18
 @arguments=
  ["UserMailer",
   "comment_post",
   "deliver_now",
   {:user_from=>
     #<User:0x00007f8f1ea6da88
      id: 12,
      email: "user2@ex.com",
      crypted_password:
       "$2a$10$7hu59Pzf5XLrGqNYGBNDhefiEMGDlpU3mZy8p4pIxawS1ryVb2nU.",
      salt: "ApGyPwiaJKzhcgZypgyr",
      username: "user2",
      created_at: Fri, 25 Mar 2022 17:18:45 JST +09:00,
      updated_at: Fri, 25 Mar 2022 17:18:45 JST +09:00,
      avatar: nil,
      notification_on_comment: true,
      notification_on_like: true,
      notification_on_follow: true>,
    :user_to=>
     #<User:0x00007f8f3cfd53b8
      id: 11,
      email: "user@ex.com",
      crypted_password:
       "$2a$10$d/W0jhT6g7F72zVOPDpd6OZFM8a6A2t.edjPSD57BD2j2mBhToUOm"
,
      salt: "tm_ZFSXcs8GLwtSM6xGP",
      username: "user",
      created_at: Mon, 21 Feb 2022 16:49:13 JST +09:00,
      updated_at: Tue, 03 May 2022 23:54:33 JST +09:00,
      avatar: "20210414-112146.jpg",
      notification_on_comment: true,
      notification_on_like: true,
      notification_on_follow: true>,
    :comment=>
     #<Comment:0x00007f8f3ce13bb0
      id: 41,
      user_id: 12,
      post_id: 13,
      body: "テスト2",
      created_at: Tue, 03 May 2022 23:55:18 JST +09:00,
      updated_at: Tue, 03 May 2022 23:55:18 JST +09:00>}],
 @executions=0,
 @job_id="79ef1847-1ebe-4f14-b3e3-6ffc01e376b0",
 @priority=nil,
 @provider_job_id="5622cab8d61840d8a0cfc806",
 @queue_name="mailers">
(END)
```
=>コメントが登録され、通知レコードが作成された後、メール通知処理が行われている。
  
[【Ruby】 論理演算子(!, &&, ||, not, and, or)をまるごと学ぼう！](https://pikawaka.com/ruby/logical-operators)

# 感想
今課題の実装は今までの実践編の知識の組み合わせで十分にできる内容で、ちょうど復習のような感じでよかったです。
論理演算子&&についてまだ使い慣れておらず、理解を深めるいい機会でした。左辺から評価されるということがわかってなかったので、実際の動きも確認することでイメージも掴めて理解でき、よかったです。
