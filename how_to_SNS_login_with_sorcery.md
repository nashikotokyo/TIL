# SorceryでのSNS認証のやり方
基本的には以下のサイトを参考に進めたが、記事が数年前のもので多少違うところもあるので改めてまとめた。  
[公式Wiki](https://github.com/Sorcery/sorcery/wiki/External)  
[【Rails】SorceryでTwitter認証](https://blog.aiandrox.com/posts/tech/2020/03/29/)  
[【Rails】Sorcery で Twitter 認証 [2]](https://note.com/artefactnote/n/ne0089a217489)  
  
## 前提:  
今開発中のアプリはメールアドレス、パスワードの登録なしに使えるように、SNSログインのみで実装しようとしている。  
開発環境の考慮のみしている。本番環境はゆくゆく。  
Twitterログインを実装
  
## 大まかな流れ
sorcery によるTwitter認証は、アプリからsorceryを経由してTwitter APIを叩いてTwitter へアクセス（ログイン）し、コールバックでアプリ側へ戻って、アプリのユーザー登録と紐付ける流れとなる。
この際、指定しておいたTwitterアカウント情報が取得出来るので必要があれば指定して取得、利用する。

## Oauthについて  
[一番分かりやすい OAuth の説明](https://qiita.com/TakahikoKawasaOAuthについて１から勉強したki/items/e37caf50776e00e733be)  
[OAuthについて１から勉強した](https://qiita.com/takecho123/items/42cdf39a3aff2a441374)  
  
## sorceryでのSNS認証の内部の動きについて  
[sorceryのSNS認証の動き](https://sakitadaiki.hatenablog.com/entry/2021/10/05/075917)
  
## 手順
- sorceryのgem導入
- sorceryでusersテーブルを作成(`bundle exec rails g sorcery:install`した後、username(1意)だけ持ったusersテーブルを作成)  
  メールやパスワードは保存の必要なしと判断したため下記のように編集 => `rails db:migrate`
  ```rb
  class SorceryCore < ActiveRecord::Migration[6.1]
    def change
      create_table :users do |t|
        t.string :username, null: false

        t.timestamps                null: false
      end

      add_index :users, :username, unique: true
    end
  end
  ```
- authenticationsテーブルとモデルを作成  
  - [Wiki](https://github.com/Sorcery/sorcery/wiki/External)通りに`bundle exec rails g sorcery:install external —only-submodules`する  
    下記のマイグレーションファイルを`db:migrate`
    ```rb
    class SorceryExternal < ActiveRecord::Migration[6.1]
      def change
        create_table :authentications do |t|
          t.integer :user_id, null: false
          t.string :provider, :uid, null: false

          t.timestamps              null: false
        end

        add_index :authentications, [:provider, :uid]
        add_index :authentications, :user_id # この部分は追記している
      end
    end
    ```
  - `rails g model Authentication —migration=false)`でAuthenticationモデルを作成する。
- UserモデルとAuthenticationモデルの関連づけ
  Authenticationモデル  
  ```rb
  class Authentication < ApplicationRecord
    belongs_to :user # 追記
  end
  ```
  Userモデル  
  ```rb
  class User < ApplicationRecord
    authenticates_with_sorcery!

    validates :username, uniqueness: true, presence: true

    has_many :authentications, dependent: :destroy # 追記
    accepts_nested_attributes_for :authentications # 追記
  end
  ```
- Twitter Developer(Twitter側)の設定(Twitter APIキーとパスワードの取得、Callback URLの設定)
  - sorcery.rb(sorceryの設定ファイル)にTwitter APIに接続する為のキーとパスワードを設定する必要があり、Twitter Developerで取得したものを設定する。  
    取得方法は[こちら](https://blog.suzukaji.com/programming/application-twitter-api/)を参照  
    ※use caseはそれぞれ目的によって変わる  
    ※keyやsecretは漏洩しないよう保管する
  - `callback_url`(ユーザーが認証画面で「許可する」を押した後のcallbackメソッドを実行する為のURL)をTwitter Developer側に設定しておく    
    `http://127.0.0.1:3000/oauth/twitter/callback`と設定(callbackメソッドやルーティングの設定は後ほど記載)  
    設定方法は[こちらの手順2](https://di-acc2.com/system/rpa/9688/)を参照  
    ※[公式の説明](https://developer.twitter.com/en/docs/apps/callback-urls)に以下のようにあるため127.0.0.1とした。
    ```
    Don’t use localhost as a callback URL
    Instead of using localhost, please use a custom host locally or http(s)://127.0.0.1
    ```
    ※`ローカルでアプリにアクセスする際はlocalhost:3000ではなく127.0.0.1:3000にアクセスする点に注意。`  
    [Twitterログイン 401 Authorization Required](https://qiita.com/niwa1903/items/2d6a3e9c103652d88538)  
    ※今回はOauth2.0をonにした  
    ※App Typeはアプリにより異なるが、今回は`Web App`を選択した  
    ※Website URLはまだないので仮で`https://twitter.com/`と設定しておいた。決まり次第変更が必要。  
- configのgem導入  
  sorcery.rbに`callback_url`を設定する必要があるが、環境毎に変える必要があるため[config](https://qiita.com/tanutanu/items/8d3b06d0d42af114a383)を入れ`rails g config:install`しておく  
- `config/initializers/sorcery.rb`に上記で取得したkeyやsecret、設定したCallback URLなど設定  
  wikiには以下のように例が書いてあるが、keyやsecretは機密情報のため、直接書きたくないのでcredentialsを使用、またCallback URLは環境で異なるためconfigを使用する。
  ```rb
  config.twitter.key = "<your key here>"
  config.twitter.secret = "<your key here>"
  config.twitter.callback_url = "http://0.0.0.0:3000/oauth/callback?provider=twitter"
  config.twitter.user_info_mapping = {:email => "screen_name"}     
  ...
  ```
  - [credentials](https://autovice.jp/articles/127)の設定  
    [こちら](https://qiita.com/NaokiIshimura/items/2a179f2ab910992c4d39)や[こちら](https://thr3a.hatenablog.com/entry/20190420/1555714812)を参考に`EDITOR='code --wait' bundle exec rails credentials:edit`を打ってcredentialsを開き、先ほど取得したkeyとsecretを  
    ```yml
    twitter:
      access_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
      secret_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    ```
    のように記載し保存する。  
    取得は以下のようにできる。
    ```rb
    Rails.application.credentials.twitter[:access_key]
    Rails.application.credentials.twitter[:secret_key]
    ```
  - `config/settings/development.yml`に開発環境でtwitter APIを使用する際の定数(access keyとsecret key、Callback URL)をまとめて記載する。  
    ※本番環境はまた別途`config/settings/production.yml`に設定の必要あり  
    ```yml
    twitter:
      access_key: <%= Rails.application.credentials.twitter[:access_key] %> # 上記でcredentialsに登録したものを取得し設定
      secret_key: <%= Rails.application.credentials.twitter[:secret_key] %> # 上記でcredentialsに登録したものを取得し設定
      callback_url: http://127.0.0.1:3000/oauth/twitter/callback # 事前にTwitter Developersに登録しておいたものと同じURLを設定
    ```
    取得は以下のようにできる。  
    ```rb
    Settings.twitter.access_key
    Settings.twitter.secret_key
    Settings.twitter.callback_url
    ```
  -  sorcery.rbに設定する`config.twitter.user_info_mapping`について  
     wikiに`The user_info_mapping takes care of converting the user info from the provider (Twitter/Facebook) into the attributes of the User. For example, the "screen_name" that we receive from Twitter will be mapped to the User's username.`とあるように、provider(Twitterなど)から取得したパラメータ(ユーザー情報)をUserモデルにマッピングする為の設定なので今回はusernameが必要なので以下のように設定する。  
     ```
     config.twitter.user_info_mapping = {
       username: 'screen_name', # Userモデルの属性名: 'twitterのパラメータ'
     }
     ```
  -  以上を踏まえて最終的に`sorcery.rb`の設定を以下のようにする  
     ```rb
     # config/initializers/sorcery.rb
     Rails.application.config.sorcery.submodules = [:external, blabla, blablu, ...]
  
     Rails.application.config.sorcery.configure do |config|
       ...
       config.external_providers = [:twitter]
      
       config.twitter.key = Settings.twitter.access_key
       config.twitter.secret = Settings.twitter.secret_key
       config.twitter.callback_url = Settings.twitter.callback_url
       config.twitter.user_info_mapping = {
         nickname: 'screen_name'
       }
       ...

       # --- user config ---
       config.user_config do |user|
       ...

         # -- external --
         user.authentications_class = Authentication
         ...

       end
       ...
    
     end
     ```  
- Oauth処理を行うOauthコントローラを作成(`rails g controller Oauths oauth callback`)  
  ※`app/views/oauths`が作成されてしまうが、使わないので削除する  
  wikiの通り、以下のように設定。  
  ```rb
  class OauthsController < ApplicationController
    skip_before_action :require_login, raise: false
      
    def oauth
      login_at(params[:provider])
    end
      
    def callback
      provider = params[:provider]
      if @user = login_from(provider)
        redirect_to root_path, :notice => "#{provider.titleize}でログインしました"
      else
        begin
          @user = create_from(provider)

          reset_session # protect from session fixation attack
          auto_login(@user)
          redirect_to root_path, :notice => "#{provider.titleize}でログインしました"
        rescue
          redirect_to root_path, :alert => "#{provider.titleize}ログインに失敗しました"
        end
      end
    end
  end
  ```
- oauthsコントローラのためのルーティングを設定  
  ```rb
  Rails.application.routes.draw do
    ...
    post 'oauth/:provider/callback', to: 'oauths#callback'
    get 'oauth/:provider/callback', to: 'oauths#callback' # for use with Github, Facebook
    get 'oauth/:provider', to: 'oauths#oauth', as: :auth_at_provider
  end
  ```
- ログイン画面を作成し、ログインボタンを表示する  
  - まずUserSessionsコントローラを作成(bundle exec rails g controller UserSessions new)
  - `routes.rb`に`get 'login', to: 'user_sessions#new'`を追加
  - ログイン画面のビュー(`user_sessions/new.html.erb`)を作成し、以下のようにログインボタンを設定
    ```rb
    <%= link_to auth_at_provider_path(provider: :twitter), class: 'btn btn-info d-flex align-items-center p-3 text-capitalize' do %>
      <i class="fa-brands fa-twitter"></i>
      <div class="mr-auto">
        Twitterでログイン
      </div>
    <% end %>
    ```
  
## ハマりポイント
上記の設定後、Twitterでログインできるかどうか確認したところ以下の記事と全く同じエラーが出ていた。  
[【Rails】Sorcery & Twitter認証で詰まった話](https://qiita.com/takemuu/items/f994a4fe20c7a236ee8c)  
  
<エラー内容>  
```
[1] pry(#<OauthsController>)> @user_hash
=> {:token=>"1385106890997178368-B06VLQWc5jLFWs2TyM2tOJznsNqOxN",
 :user_info=>
  {"errors"=>
    [{"message"=>
       "You currently have Essential access which includes access to Twitter API v2 endpoints only. If you need access to this endpoint, you’ll need to apply for Elevated access via the Developer Portal. You can learn more here: https://developer.twitter.com/en/docs/twitter-api/getting-started/about-twitter-api#v2-access-leve",
      "code"=>453}]},
 :uid=>""}
```
理由は、2022年4月から厳しくなったようで、Twitter Developerでアクセス権限をEssentialからElevatedにしないといけなくなった。  
変更方法は以下の記事を参照した。  
[Twitter API Elevated(高度なアクセス)の利用申請をしてみる！](https://tensei-shinai.com/2022/04/27/twitter-api-elevated/#toc1)  

## その他
- ユーザー情報からメールアドレス取得したい場合は別途追加設定が必要。  
設定方法は冒頭の参考にしたサイトに書いてある
- 今回は開発環境しか考慮してないので本番用の設定(config/settings/production.ymlなど)が必要
  
  
# LINE認証の追加
Twitterログインの実装後LINEログインを追加した  
  
## 手順
- [LINE Messaging APIのプロバイダーとチャネルを作成する方法](https://www.yukibnb.com/entry/linemessagingapi_account) を参考にLINE Developersに登録、`チャネルID`と`チャネルシークレット`を取得する  
- LINE DevelopersにCallback URLを設定する  
  [コールバックURLを設定する](https://developers.line.biz/ja/docs/line-login/integrate-line-login/#setting-callback-url)  
  <ハマりポイント>  
  localhostだと弾かれて使えないのでTwitter認証同様に127.0.0.1にする必要があった。  
  [![Image from Gyazo](https://i.gyazo.com/a7503ec6768930a4ae65aff3b4f4928c.png)](https://gyazo.com/a7503ec6768930a4ae65aff3b4f4928c)  
  [![Image from Gyazo](https://i.gyazo.com/02b389d3fd2b1b62e704820c8c4519ef.png)](https://gyazo.com/02b389d3fd2b1b62e704820c8c4519ef)  
  ※Twitter認証同様`ローカルでアプリにアクセスする際はlocalhost:3000ではなく127.0.0.1:3000にアクセスする点に注意。`  
- https通信できるように設定する  
　 [mkcertでローカルホストをSSL化して動作確認](https://qiita.com/tarakish/items/a4c73104375878ad61c5#8mkcert%E3%81%A7%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E3%83%9B%E3%82%B9%E3%83%88%E3%82%92ssl%E5%8C%96%E3%81%97%E3%81%A6%E5%8B%95%E4%BD%9C%E7%A2%BA%E8%AA%8D)にあるようにあるようにLINE認証の場合は、https通信でないと動作してくれないため動作確認のためにmkcertとpumaを使う方法でSSL化する必要がある。  
  [アプリケーションにhttpsでアクセスできるように設定する](https://qiita.com/t12u/items/52721d3ea2ce1e605ae9#%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%ABhttps%E3%81%A7%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%A7%E3%81%8D%E3%82%8B%E3%82%88%E3%81%86%E3%81%AB%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)を参考に、`localhost`が使えないのでその部分を`127.0.0.1`に読み替えて設定。  
  ※`mkcert localhost`でなく`mkcert 127.0.0.1`を実行  
  ※puma.rbには以下のように設定  
  ```rb
  ssl_bind "0.0.0.0", "3000", {
    cert: "config/certs/127.0.0.1.pem",
    key:  "config/certs/127.0.0.1-key.pem"
  }
  ```
  ※アプリへのアクセスは`https://localhost:3000`ではなく`https://127.0.0.1:3000`にすることに注意  
- [【Rails】Sorceryで外部認証 〜LINE編〜](https://qiita.com/tarakish/items/a4c73104375878ad61c5#6-sorceryrb%E3%81%AE%E8%A8%AD%E5%AE%9A)を参考に`sorcery.rb`、`config/settings/development.yml`、`config/credentials.yml.enc`をTwitterの時と同様に以下のように設定していく  
  ```yml
  # config/credentials.yml.enc
  line:
    access_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    secret_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  ```
  ```yml
  # config/settings/development.yml
  line:
    access_key: <%= Rails.application.credentials.line[:access_key] %>
    secret_key: <%= Rails.application.credentials.line[:secret_key] %>
    callback_url: https://127.0.0.1:3000/oauth/line/callback
  ```
  ```rb
  # config/initializers/sorcery.rb
  Rails.application.config.sorcery.submodules = [:external]
  
  Rails.application.config.sorcery.configure do |config|
  ...
    config.external_providers = [:twitter, :line]
    
    ... 
    config.line.key = Settings.line.access_key
    config.line.secret = Settings.line.secret_key
    config.line.callback_url = Settings.line.callback_url
    config.line.scope = "profile"
    # config.line.bot_prompt = "normal"
    config.line.user_info_mapping = {
      username: 'displayName'
    }
    ...
  end
  ```
- ボタンの設置
  ```rb
  <%= link_to auth_at_provider_path(provider: :line), class: 'btn btn-success d-flex align-items-center justify-content-center p-3 m-2' do %>
    <i class="fa-brands fa-line fa-2x"></i>
    <div class="flex-grow-1">
      LINEでログイン
    </div>
  <% end %>
  ```
- LINE認証では`http`から`https`に変更したのでTwitter認証の方も合わせてCallback URLの設定を以下のように変更しておく
  ```yml
  # config/settings/development.yml
  twitter:
    access_key: <%= Rails.application.credentials.twitter[:access_key] %>
    secret_key: <%= Rails.application.credentials.twitter[:secret_key] %>
    callback_url: https://127.0.0.1:3000/oauth/twitter/callback
  ```
  Twitter Developer側の設定  
  [![Image from Gyazo](https://i.gyazo.com/82e00d02e8b58b97dd1fa4c71b692d7e.png)](https://gyazo.com/82e00d02e8b58b97dd1fa4c71b692d7e)  
  
# Google認証の追加
Twitter、LINE認証と同様の実装方法。  
[【Rails】Sorceryで外部認証 〜Google編〜](https://qiita.com/tarakish/items/8e61106fa2473a9d97e8)参照  
※`config.google.scope`については[Google API の OAuth 2.0 スコープ](https://developers.google.com/identity/protocols/oauth2/scopes#oauth2)参照。  
以下のように実装。  
```yml
# config/credentials.yml.enc
google:
  access_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  secret_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```
```yml
# config/settings/development.yml
google:
  access_key: <%= Rails.application.credentials.google[:access_key] %>
  secret_key: <%= Rails.application.credentials.google[:secret_key] %
  callback_url: https://127.0.0.1:3000/oauth/google/callback
```
```rb
# config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:external]
  
Rails.application.config.sorcery.configure do |config|
...
  config.external_providers = [:twitter, :line]
    
  ... 
  config.google.key = Settings.google.access_key
  config.google.secret = Settings.google.secret_key
  config.google.callback_url = Settings.google.callback_url
  config.google.user_info_mapping = {
    username: "name"
  }
  config.google.scope = "https://www.googleapis.com/auth/userinfo.profile"
  ...
end
```
```rb
# app/views/user_sessions/new.html.erb(ボタンの設置)
<%= link_to auth_at_provider_path(provider: :google), class: 'btn btn-danger d-flex align-items-center justify-content-center p-3 m-2' do %>
  <i class="fa-brands fa-google fa-2x"></i>
  <div class="flex-grow-1">
    Googleでログイン
  </div>
<% end %>
```
# その他追加対応
- Twitter認証のキャンセル後の401エラーの解消  
  Twitter認証画面に遷移後キャンセルを押すと401エラーが出てしまうのでoauthsコントローラーに以下を追加した  
  ```rb
  # oauthsコントローラ
  def callback
    return redirect_to root_path, warning: 'ログインをキャンセルしました' if params[:error] || params[:denied]
  ```
  [キャンセル後のリダイレクト処理](https://blog.aiandrox.com/posts/tech/2020/03/29/#キャンセル後のリダイレクト処理)  
  
- ログアウト機能とボタンの実装  
  user_sessionsコントローラにdestroyアクションを実装、ヘッダにログアウトボタンを設置  
 
