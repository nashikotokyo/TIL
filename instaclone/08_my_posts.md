# 08.ユーザーの詳細ページに投稿一覧を表示する  
### 内容
- ユーザーの詳細ページに同ユーザーの投稿を一覧表示させてください。
### 補足
- タイル表示させる
- ヘッダーのユーザーアイコンに自分のユーザー詳細ページへのリンクを設定してさせる
  
# 作業手順
- app/views/users/show.html.slim（ユーザ詳細画面）修正
- app/views/posts/_thumbnail_post.html.slim（ユーザ詳細画面のサムネイル1つ1つの部分のパーシャル）作成
- application.scss追記
- app/views/shared/_header.html.slim(ログイン後のヘッダ)修正
  
# 学んだこと
## レンダリングでcollectionオプションを使用した場合の注意点
最初はユーザの投稿一覧を表示する部分を以下のように定義していた。
```
# app/views/users/show.html.slim(ユーザ詳細画面)

.container
  .row
    .col-md-10.offset-md-1
      .card
        .card-body
          .text-center.mb-3
            = image_tag 'profile-placeholder.png', size: '100x100', class: 'rounded-circle mr-1'
          .profile.text-center.mb-3
            = @user.username
          .text-center
            = render 'follow_area', user: @user
          hr
          .row
            - @user.posts.each do |post|
              = render 'posts/thumbnail_post', post: post
```

```
# app/views/posts/_thumbnail_post.html.slim（ユーザ詳細画面のサムネイルのパーシャル）

.col-md-4.mb-3
  = link_to post_path(post), class: 'thumbs' do
    = image_tag post.images.first.url
```
  
show.html.slim(ユーザ詳細画面)で  
``` 
 - @user.posts.each do |post|
   = render 'posts/thumbnail_post', post: post
```
とeachで回すやり方で書いてしまうと、変数の要素分繰り返しrenderされるのでパフォーマンスが悪いという問題がある。  
  
これを解決するために`collectioinオプション`を利用する必要がある。  
  
show.html.slim(ユーザ詳細画面)の変更後  
```
   = render partial: 'posts/thumbnail_post', collection: @user.posts
```
=>collection:には 繰り返し表示したい要素が入っているインスタンス(配列)を設定するので今回は`@user.posts`  
そしてrenderしたいパーシャル名が`_thumbnail_post`  
  
また、collectionオプションを使用した場合、部分テンプレート内で使用する変数はpartialで指定した名前になる仕様。つまりパーシャル名が部分テンプレート内の変数となる。  
なので、`_thumbnail_post.html.slim`内の変数も以下のように`thumbnail_post`に変更する必要がある。  
```
.col-md-4.mb-3
  = link_to post_path(thumbnail_post), class: 'thumbs' do
    = image_tag thumbnail_post.images.first.url
```
  
いつもは`= render partial: 'posts/_post', collection: @posts`などの場合呼び出し部分で`render @posts`などと省略できるが、今回はパーシャル名やコレクション名が特有なので書き方に注意する。(省略ももちろんできない)  
  
※collectionオプションを使用する際は`render partial:`とpartialを明記しないとエラーになる。  
  
https://tech-essentials.work/questions/143  
https://railsguides.jp/layouts_and_rendering.html#コレクションをレンダリングする  
https://pikawaka.com/rails/partial_template  
  
# 感想
この課題で、サムネイルの実装でapplication.scssを修正する部分がありましたが、本質的な部分ではないので優先度低めと前教えて頂いたので、ざっくり動きみるだけで詳しく調べたりしなかったです。  
html, css, bootstrapについてはインスタクローン終わってポートフォリオに入ったら、以前教えて頂いた参考サイトを確認して復習してみようと思います。  
