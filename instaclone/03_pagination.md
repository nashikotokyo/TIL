# 03.ページネーションの実装
### 内容
投稿のページネーションを実装してください  
  
### 補足
- ページネーションにはkaminariを使用する
- 1ページあたり15件とする
- ページネーションにもbootstrapを適用する
  
# 作業内容
- Gemfileにkaminariを追加しbundle install  
  
- `Rails g kaminari:config`**で設定ファイル（config/initializers/kaminari_config.rb）を生成、各ページ15件表示に設定  
※デフォルトは25件  
```
# kaminari_config.rb

Kaminari.configure do |config|
  config.default_per_page = 15
end
```
  
- posts_controllerのindexアクションを修正（pageメソッドを使う）  
```
 index
    @posts = Post.all.includes(:user).order(created_at: :desc).page(params[:page])
  end
```
=>例えば2ページ目を押した場合、`page(params[:page]`の部分で、urlに含まれる`http://localhost:3000/?page=2`の2という情報を取得しているので、2ページ目に表示する投稿だけに絞られて表示される。  
  
- index.htmlにページネーションを表示
```
.container
  .row
    .col-md-8.col-12
      = render @posts
      = paginate @posts
 ~略~
```
  
- application.scssにページネーション部分のcssを追記
```
.pagination {
  justify-content: center;
}
```
=> 要素を真ん中に寄せている  
  
- `rails g kaminari:views bootstrap4`でbootstrapのデザインを適用させる
  
# 全体を通して
ページネーションは以前使用したことがあり、また実装手順もわかりやすいので特に問題なく実装できた。  
  
kaminariの日本語化は今回は対応しなかったが、localeファイルに記載することでできるので、自分のポートフォリオを作るときなどに試してみたい。  
