# 16. システムスペックを実装
### 内容
- モデルスペックを実装してください

### 補足
解答例では以下のスペックを書いています。

- ログイン成功/失敗
- ログアウトできる
- ユーザー登録成功/失敗
- フォローできること
- フォローをはずせること
- 投稿一覧が閲覧できる
- 新規投稿できる
- 自分の投稿に編集・削除ボタンが表示される
- 他人の投稿には編集・削除ボタンが表示されない
- 投稿を更新できる
- 投稿を削除できる
- 投稿の詳細画面が閲覧できる
- 投稿に対していいねできる
- 投稿に対していいねを外せる
  
# 作業手順
- システムスペックに必要なGem(capybara, webdrivers)の導入(Gemfile, Gemfile.lock)
- capybaraやシステムスペックについての初期設定(spec/rails_helper.rb)
- ログイン・ログアウトのシステムスペックを作成(spec/support/system_helper.rb, spec/system/user_sessions/user_session_spec.rb)
- ユーザー登録フローのシステムスペック実装(spec/system/users/users_spec.rb)
- ポストのCRUDのシステムスペック実装(app/views/posts/_form.html.slim, app/views/posts/_post.html.slim, spec/system/posts/posts_spec.rb)
- いいね機能のシステムスペックの実装(app/views/posts/_like.html.slim, app/views/posts/_unlike.html.slim, spec/system/posts/posts_spec.rb)
- フォロー機能のシステムスペックの実装(spec/system/users/users_spec.rb)

# 学んだこと
## capybaraとは
UIテストのためのrubyフレームワークで、ページの表示、フォームの入力、ボタン(リンク)のクリック等のUI操作をテストコード上で実行できる。  
  
[Capybaraチートシート](https://qiita.com/morrr/items/0e24251c049180218db4)  
[使えるRSpec入門・その4「どんなブラウザ操作も自由自在！逆引きCapybara大辞典」](https://qiita.com/jnchito/items/607f956263c38a5fec24)
[capybara (gem)](https://github.com/teamcapybara/capybara)  
  
## capybara導入に関して
rails new時にデフォルトで以下が既にインストールされていた。
```rb
# Gemfile

group :test do
  # Adds support for Capybara system testing and selenium driver
  gem 'capybara', '>= 2.15'
  gem 'selenium-webdriver'
  # Easy installation and use of chromedriver to run system tests with Chrome
  gem 'chromedriver-helper'
end
```
### ・selenium-webdriver(gem)とは
capybaraはシンプルなブラウザシミュレータ(つまりドライバ)を使って、 テストに書かれたタスクを実行していく。このドライバは`Rack::Test`というドライバで、速くて信頼性が高いが、JavaScript の実行はサポートしていない。  
javascriptをテストするためにselenium-webdriverというgemが必要。(CapybaraではSeleniumがデフォルトのJavaScriptドライバーになっている。)  
デフォルトではFirefoxを使ってテストするよう設定されているが、最近のFirefoxだと互換性問題が報告されているのでChromeを使うように設定しておいた方が良い。  
※設定は下記`chromeの設定`参照

[capybaraとselenium-webdriverとは](https://qiita.com/seri1234/items/7be4053fff94c7a120cc)  
[【Rspec】Capybaraについて](http://www.code-magagine.com/?p=9739)  
  
### ・chromedriver-helper(gem)とwebdrivers(gem)とは  
`webdrivers`とは:  
RailsのSystem TestやFeature Spec、System SpecとChromeを組み合わせて動かしたい場合は、ChromeDriverのインストールが別途必要になる。  
このインストールを簡便化するために、`chromedriver-helper`というChromeDriverをかんたんに導入してくれるgemが提供されていたが、サポートが終了し今は`webdrivers`というgemを使う必要がある。  
なお、Webdriversはライブラリの依存関係上`selenium-webdriver`も一緒にインストールしてくれるため明示的に書かなくても済む。(つまり`gem 'selenium-webdriver'`は書かなくてもいいらしい)  
アプリケーション全体が一つのシステムとして期待通りに動くか否かを検証する場合、実際にブラウザでテストがそうのように動いているのか目視できた方が分かりやすいので、Capybaraとwebdriversはセットで入れることが多い。  

[【Capybara,webdrivers】統合テスト(System Spec)とは](https://qiita.com/mmaumtjgj/items/83fe862df5d6e0593569)  
  
※`chromedriver-helper`から`webdrivers`に変更した。  
[サポートが終了したchromedriver-helperからwebdrivers gemに移行する手順](https://qiita.com/jnchito/items/f9c3be449fd164176efa)  
  
※また、`webdrivers`があれば'selenium-webdriver'を明示的に書かなくてもいい為、結果的に解答例(下記)のようになった。
```
group :test do
  gem 'capybara'
  gem 'webdrivers'
end
```
  
## `spec_helper.rb`と`rails_helper.rb`の違い
`spec/rails_helper.rb`にはRails 特有の設定を書き、`spec/spec_helper.rb`にはRSpecの全体的な設定を書く。これによって、Railsを必要としないテストを書きやすくなる。  
下記の記事では、普通のRailsアプリケーションで、Railsを使わずにテストをする、というケースは考えにくいので`rails_helper.rb`にだけ設定をまとめたほうが分かりやすくていいんじゃないか。とあるので今回その通りにしてみた。  
  
[【備忘録】rails_helperとspec_helperの違いについて](https://qiita.com/geshi/items/d2158802bdf8540e9200)  
[Rspecの設定はrails_helper.rbにだけ書けばよい](https://qiita.com/kazutosato/items/8255a685206b327ad151)  
  
## 設定関連
### ・capybaraの初期設定
capybaraをRSpecで利用する場合、`spec/rails_helper.rb`に記述を追加する。  
```
# spec/rails_helper.rb

require 'capybara/rspec'
```
  
### ・システムスペックを書くにあたっての設定  
RSpecのテストでコードの重複をなくし共通化したい処理は、モジュールにして`spec/support配下`に置くのが作法。  
特にフィーチャースペックやシステムスペックでよく使える手法（ログイン処理の共通化等）。  
  
下記の2つの設定を`rails_helper.rb`に追加  
1. spec/support/配下のファイルを読み込む設定をする  
下記のように、コメントアウトされていた1ラインを有効にする  
```rb
Dir[Rails.root.join('spec', 'support', '**', '*.rb')].sort.each { |f| require f }
```
  
2. spec/support/配下で定義したモジュールをspecファイル内で読み込まれるように設定する  
今回は`spec/support/system_helper.rb`を作成し、共通処理としてログイン処理のモジュールを定義している。  
以下のように設定して,`system_helper.rb(SystemHelper)`を指定したテストタイプ`system`で読み込み、systemのspecファイル内で使用できるようにしている。  
```rb
RSpec.configure do |config|
  # config.include ◯◯(モジュール名), type: :☓☓(使用するテストのタイプ)
  config.include SystemHelper, type: :system
end
```
※各specファイルで以下のように設定することもできる  
```rb
RSpec.describe '◯◯', type: :system do
  include ◯◯(モジュール名)
end
```
  
### ・chromeのヘッドレスドライバの設定
jsを使用するsystem specの場合はchromeのヘッドレスドライバ(headless chrome)を使用するよう設定する。  
  
※`headless chrome` => プログラミングのコードを書いてそれを実行させることで(GUIなしで)chromeを高速に動作させる事ができることからE2Eテストで使われている  
※jsを使用しないテストの場合は、高速なRack::Testドライバを使用  
  
下記のように設定  
```rb
RSpec.configure do |config|
  # before(:each) => type: :system（つまりシステムスペック）で書かれたテストの各exampleの前に毎回実行される処理
  config.before(:each, type: :system) do
    # driven_by(ドライバの種類, using: 使うブラウザ, screen_size: [XXXX, XXXX], options: オプション={})
    driven_by :selenium, using: :headless_chrome, screen_size: [1920, 1080]
  end
end
```
[Railsドキュメント - driven_by](https://railsdoc.com/page/driven_by)  
[before(:suite)/before(:all)/before(:each)それぞれの違いについてまとめてみた](https://techtechmedia.com/before-suite-all-each-rspec/)  
[「Headless Chrome」を使えるようにする方法を解説！](https://appli-world.jp/posts/9026)  
[RSpecコトハジメ 初期設定マニュアル](https://qiita.com/naoki_mochizuki/items/1d3026a32786642fc762)  
  
## ログインログアウトのシステムスペックテスト
- ログイン処理をモジュールとして`spec/support/system_helper.rb`に定義
```rb
# spec/support/system_helper.rb

module SystemHelper
  # ログインの処理
  def login
    # userをDBに作成する
    user = create(:user)
    # visitでログイン画面に遷移する
    visit login_path
    # fill_in ~ with でメールアドレスとパスワードを入力
    fill_in 'メールアドレス', with: user.email
    fill_in 'パスワード', with: '12345678'
    # ログインボタンをクリック
    click_button 'ログイン'
  end
  
  # userを引数に渡すバージョン
  def login_as(user)
    visit login_path
    fill_in 'メールアドレス', with: user.email
    fill_in 'パスワード', with: '12345678'
    click_button 'ログイン'
  end
end
```
- ログイン、ログアウトのシステムスペックを定義
```rb
# spec/system/user_sessions/user_session_spec.rb

require 'rails_helper'

RSpec.describe 'ログイン・ログアウト', type: :system do

  # userの定義。letなので実際に使われる時に遅延評価される。
  let(:user) { create(:user) }
  
  # ログインについてのテストをグルーピング
  describe 'ログイン' do
    # contextで条件によってグルーピング
    context '認証情報が正しい場合' do
      # これがexample
      it 'ログインできること' do
        # 下記のログインの処理はモジュールSystemHelperで定義したloginで省略できるが、
        # ログインできない場合と比較するためにloginは使っていない(読みやすさ)
        visit login_path
        fill_in 'メールアドレス', with: user.email
        fill_in 'パスワード', with: '12345678'
        click_button 'ログイン'
        # ログイン成功で投稿一覧画面に遷移し、"ログインしました"というフラッシュメッセージがあるか確認する
        expect(current_path).to eq posts_path
        expect(page).to have_content 'ログインしました'
      end
    end

    context '認証情報に誤りがある場合' do
      it 'ログインできないこと' do
        visit login_path
        fill_in 'メールアドレス', with: user.email
        # あえて間違ったパスワードを入れる
        fill_in 'パスワード', with: '1234'
        click_button 'ログイン'
        # ログイン失敗でログイン画面のままなこと、"ログインに失敗しました"というフラッシュメッセージがあるか確認する
        expect(current_path).to eq login_path
        expect(page).to have_content 'ログインに失敗しました'
      end
    end
  end

  # ログアウトについてのテストをグルーピング
  describe 'ログアウト' do
    # beforeでexampleの前にログイン処理を実行
    before do
      login
    end
    it 'ログアウトできること' do
      # ログアウトをクリック
      click_on('ログアウト')
      # ログアウトが成功し、ログイン画面に遷移し、"ログアウトしました"というフラッシュメッセージがあるか確認する
      expect(current_path).to eq login_path
      expect(page).to have_content 'ログアウトしました'
    end
  end
end
```
## ユーザー登録フローのシステムスペックを定義
```rb
# spec/system/users/users_spec.rb

require 'rails_helper'

RSpec.describe 'ユーザー登録', type: :system do
  describe 'ユーザー登録' do
    context '入力情報が正しい場合' do
      it 'ユーザ登録できること' do
        # ユーザ登録画面に遷移
        visit new_user_path
        # 登録に必要な正しい情報を入力
        # fill_in 'ユーザー名'だと検索ボックスのユーザー名とAmbiguous matchになってしまうのでname属性で指定
        fill_in 'user[username]', with: 'Rails太郎'
        fill_in 'メールアドレス', with: 'rails@example.com'
        fill_in 'パスワード', with: '12345678'
        fill_in 'パスワード確認', with: '12345678'
        # 登録ボタンをクリック
        click_button '登録'
        # ユーザ登録が成功し、投稿一覧画面に遷移し、"ユーザーを作成しました"というフラッシュメッセージがあるか確認する
        expect(current_path).to eq posts_path
        expect(page).to have_content 'ユーザーを作成しました'
      end
    end

    context '入力情報に誤りがある場合' do
      it 'ユーザ登録できないこと' do
        visit new_user_path
        # 各入力情報をあえてブランクにする
        fill_in 'user[username]', with: ''
        fill_in 'メールアドレス', with: ''
        fill_in 'パスワード', with: ''
        fill_in 'パスワード確認', with: ''
        click_button '登録'
        # ユーザ登録に失敗し各種エラーメッセージやフラッシュメッセージがあるか確認する
        expect(page).to have_content 'ユーザー名を入力してください'
        expect(page).to have_content 'メールアドレスを入力してください'
        expect(page).to have_content 'パスワードは3文字以上で入力してください'
        expect(page).to have_content 'パスワード確認を入力してください'
        expect(page).to have_content 'ユーザーの作成に失敗しました'
      end
    end
  end
end
```
#### ・Capybaraの`fill_in`には文字列以外も`name属性`や`id`が指定できる
`fill_in 'ユーザー名', with ~~ `にしてしまうとヘッダにある検索ボックスの'ユーザー名'とかぶってしまっていて、テストがAmbiguous matchになってしまうのでname属性で指定することにした。  
※idでも指定可能   例) fill_in 'user_username', with: 'Rails太郎'  
  
[Capybara の fill_in 対象はラベル以外でも指定できることを思い出そう](https://zenn.dev/tatsurom/articles/1976321aae77f4b63a14)  
  
## ポストのCRUDのシステムスペックを実装
```rb
# spec/system/posts/posts_spec.rb

require 'rails_helper'

RSpec.describe 'ポスト', type: :system do
  # ポスト一覧についてのテストをグルーピング
  describe 'ポスト一覧' do
    # let!なので事前に実行される
    # userをdbに作成しそのuserのpostと、その他2つの別のuserのpostを作成しておく
    let!(:user) { create(:user) }
    let!(:post_1_by_others) { create(:post) }
    let!(:post_2_by_others) { create(:post) }
    let!(:post_by_user) { create(:post, user: user) }

    # contextで条件によってグルーピング
    context 'ログインしている場合' do
      # beforeで事前にuserとしてログインし、post_1に紐づくuserをフォローするよう定義
      before do
        login_as user
        user.follow(post_1_by_others.user)
      end
      it 'フォロワーと自分の投稿だけが表示されること' do
        # 投稿一覧画面に遷移
        visit posts_path
        # pageの中に自分と、post_1の投稿のbodyが含まれ、post_2の投稿のbodyは含まれないことをテスト
        expect(page).to have_content post_1_by_others.body
        expect(page).to have_content post_by_user.body
        expect(page).not_to have_content post_2_by_others.body
      end
    end

    context 'ログインしていない場合' do
      it '全てのポストが表示されること' do
        # 投稿一覧画面に遷移
        visit posts_path
        # userの投稿、post_1の投稿、post_2の投稿のbodyが含まれることをテスト
        expect(page).to have_content post_1_by_others.body
        expect(page).to have_content post_2_by_others.body
        expect(page).to have_content post_by_user.body
      end
    end
  end

  # ポスト投稿についてのテストをグルーピング
  describe 'ポスト投稿' do
    it '画像を投稿できること' do
      # まずログインする
      login
      # 投稿作成画面に遷移
      visit new_post_path
      # id: 'posts_form' を投稿フォーム(_form.html.slim)のidとしてform_withの部分に定義しておき、
      # システムテスト側でwithin '#posts_form' do ~ end とすることでこのid内にdo ~ end内の検索/操作の範囲を制限することができる  
      # これをしないと、fill_in '' が検索ボックスの'本文'とAmbiguous matchになってしまう
      within '#posts_form' do
        # attach_fileでfixtureに格納してある画像を設定
        attach_file '画像', Rails.root.join('spec', 'fixtures', 'fixture.png')
        # ｆill_in ~ with で投稿の本文を入力し登録するボタンをクリック
        fill_in '本文', with: 'This is an example post'
        click_button '登録する'
      end
      # 投稿が成功し、正しいフラッシュメッセージと本文の内容がpageに含まれていることをテスト
      expect(page).to have_content '投稿しました'
      expect(page).to have_content 'This is an example post'
    end
  end

  # ポスト更新についてのテストをグルーピング
  describe 'ポスト更新' do
    # userをdbに作成しそのuserのpostと、もう1つ別のuserのpostを作成しておく
    let!(:user) { create(:user) }
    let!(:post_1_by_others) { create(:post) }
    let!(:post_by_user) { create(:post, user: user) }
    # beforeで事前にuserとしてログインする。下記の3つのexampleの前に毎回呼ばれる
    before do
      login_as user
    end
    it '自分の投稿に編集ボタンが表示されること' do
      # 投稿一覧画面に遷移
      visit posts_path
      # id="post-#{post.id}" を各投稿のパーシャル(_post.html.slim)のidとして定義しておき、
      # システムスペック側でwithin "#post-#{post_by_user.id}" do ~ end とすることで対象の投稿(自分の投稿)のid内に検索/操作の範囲を絞っている  
      # idを設定しないと、投稿一覧画面内の複数の投稿の中でテスト対象の投稿を見つけられない
      within "#post-#{post_by_user.id}" do
        # 削除・編集ボタンに操作の目印がなかったので、投稿のパーシャル(_post.html.slim)内の
        # 削除・編集ボタンのlink_to部分にclass: 'edit-button'と'delete-button'を追記
        # have_cssでpage内の自分の投稿に、設定した削除ボタンと編集ボタンのCSS要素(class属性)が含まれていることをテスト  
        expect(page).to have_css '.delete-button'
        expect(page).to have_css '.edit-button'
      end
    end

    #  '自分の投稿に編集ボタンが表示されること'と流れはほぼ同じ
    it '他人の投稿には編集ボタンが表示されないこと' do
      # userでログイン後、post_1に紐づくuserをフォローする
      user.follow(post_1_by_others.user)
      # 投稿一覧画面に遷移
      visit posts_path
      # フォローしたことで表示されたpost_1に編集ボタンが表示されていないことをテスト
      within "#post-#{post_1_by_others.id}" do
        expect(page).not_to have_css '.edit-button'
      end
    end

    it '投稿が更新できること' do
      # userでログイン後、userの投稿の編集画面に遷移
      visit edit_post_path(post_by_user)
      # 本文を変更して更新をする
      within '#posts_form' do
        attach_file '画像', Rails.root.join('spec', 'fixtures', 'fixture.png')
        fill_in '本文', with: 'This is an example updated post'
        click_button '更新する'
      end
      # 更新が成功して、pageに正しいフラッシュメッセージと本文があるかどうかテスト
      expect(page).to have_content '投稿を更新しました'
      expect(page).to have_content 'This is an example updated post'
    end
  end
  
  # ポスト削除についてのテストをグルーピング
  # ポスト更新の時の流れとほぼ同じ
  describe 'ポスト削除' do
    let!(:user) { create(:user) }
    let!(:post_1_by_others) { create(:post) }
    let!(:post_by_user) { create(:post, user: user) }
    before do
      login_as user
    end
    # '自分の投稿に編集ボタンが表示されること'の流れとやり方はほぼ同じ
    it '自分の投稿に削除ボタンが表示されること' do
      visit posts_path
      within "#post-#{post_by_user.id}" do
        expect(page).to have_css '.delete-button'
      end
    end

    # '他人の投稿には編集ボタンが表示されないこと'の流れとほぼ同じ
    it '他人の投稿には削除ボタンが表示されないこと' do
      user.follow(post_1_by_others.user)
      visit posts_path
      within "#post-#{post_1_by_others.id}" do
        expect(page).not_to have_css '.delete-button'
      end
    end

    # '投稿が更新できること'の流れとほぼ同じ
    it '投稿が削除できること' do
      # ログイン後、投稿一覧画面に遷移
      visit posts_path
      # 自分の投稿の削除ボタンをクリックし、確認ダイアログをOKする
      within "#post-#{post_by_user.id}" do
        page.accept_confirm { find('.delete-button').click }
      end
      # 削除が成功し、正しいフラッシュメッセージが出ていること、削除した投稿が存在しないことをテスト
      expect(page).to have_content '投稿を削除しました'
      expect(page).not_to have_content post_by_user.body
    end
  end

  # ポスト詳細についてのテストをグルーピング
  describe 'ポスト詳細' do
    # userをdbに作成しそのuserのpostを作成しておく
    let(:user) { create(:user) }
    let(:post_by_user) { create(:post, user: user) }
    # 事前にuserとしてログイン
    before do
      login_as user
    end

    it '投稿の詳細画面が閲覧できること' do
      # 投稿の詳細画面に遷移
      visit post_path(post_by_user)
      # 現在のパスが投稿の詳細画面のパスであることをテスト
      expect(current_path).to eq post_path(post_by_user)
    end
  end
end
```
#### ・`within`(capybara)
withinを使うと、検索/操作の影響範囲を制限することができる。  
`within('selector')` =>	指定セレクタに該当する要素以下をスコープとする  
  
[スコープを切る(within)](https://qiita.com/morrr/items/0e24251c049180218db4#%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%97%E3%82%92%E5%88%87%E3%82%8Bwithin)  
  
#### ・`have_css`(capybara)
ページ内に特定のCSS要素(class属性)が表示されていることを検証できる  
例) expect(page).to have_css '.delete-button'  
  
[ページ内に特定のCSS要素が表示されていることを検証する](https://qiita.com/jnchito/items/607f956263c38a5fec24#%E3%83%9A%E3%83%BC%E3%82%B8%E5%86%85%E3%81%AB%E7%89%B9%E5%AE%9A%E3%81%AEcss%E8%A6%81%E7%B4%A0%E3%81%8C%E8%A1%A8%E7%A4%BA%E3%81%95%E3%82%8C%E3%81%A6%E3%81%84%E3%82%8B%E3%81%93%E3%81%A8%E3%82%92%E6%A4%9C%E8%A8%BC%E3%81%99%E3%82%8B)  
  
#### ・`accept_confirm`
Capybaraで確認ダイアログを操作する場合はpage.accept_confirmを使う  
{}(ブロック)内の、モーダルを表示させる処理を実行し、Okを押してくれる  
例) page.accept_confirm { find('.delete-button').click } => 削除ボタンをクリックした後確認ダイアログをOKしている  
  
[Capybara でデータ削除時の confirm ボタンを押したい](https://k-koh.hatenablog.com/entry/2020/08/21/225715)  
[Method: Capybara::Session#accept_confirm](https://www.rubydoc.info/gems/capybara/Capybara%2FSession:accept_confirm)  
  
## いいね機能のシステムスペックを実装
`spec/system/posts/posts_spec.rb`に追記
```rb
# spec/system/posts/posts_spec.rb

# いいねについてのテストをグルーピング
describe 'いいね' do
  # let!で事前にuserをdbに作成し、もう一人別のuserのpostを作成しておく
  let!(:user) { create(:user) }
  let!(:post) { create(:post) }
  # 事前にuserとしてログインし、もう一人のuserをフォローしておく
  before do
    login_as user
    user.follow(post.user)
  end
  it 'いいねができること' do
    # 投稿一覧画面に遷移
    visit posts_path
    # いいねボタンに操作の目印がなかったので、like, unlikeパーシャル(app/views/posts/_like.html.slim)内の
    # like, unlikeボタンのlink_to部分にclass: 'like-button'と'unlike-button'を追記しておく
    # expect{ X }.to change{ Y }.by( A ) ＝> 「XするとYが（元の個数はともかく）B個増える(or減る)ことを期待する」
    # つまり、フォローした他userの投稿のlikeボタンをクリックしたら成功しunlikeボタンに変わる(= X)ことをテストし
    # その結果、userが持ついいねした投稿の数(= Y)が1増える(= A)ことをテストしている
    expect {
      within "#post-#{post.id}" do
        find('.like-button').click
        expect(page).to have_css '.unlike-button'
      end
    }.to change{user.like_posts.count}.by(1)
  end
  
  
  it 'いいねを取り消せること' do
    # 先にフォローした他userの投稿をいいねしておく
    user.like(post)
    # あとは'いいねができること'とほぼ同じ流れ
    visit posts_path
    expect {
      within "#post-#{post.id}" do
        find('.unlike-button').click
        expect(page).to have_css '.like-button'
      end
    }.to change(user.like_posts, :count).by(-1)
  end
end
```
## フォロー機能のシステムスペックを実装
`spec/system/users/users_spec.rb`に追記
```rb
# spec/system/users/users_spec.rb

# フォローについてのテストをグルーピング
describe 'フォロー' do
  # let!で事前にログインuserともう一人のuserをdbに作成しておく
  let!(:login_user) { create(:user) }
  let!(:other_user) { create(:user) }

  # 事前にlogin_userでログインしておく
  before do
    login_as login_user
  end
  
  it 'フォローができること' do
    # 投稿一覧画面に遷移  
    visit posts_path
    # いいね機能のテストの書き方とほぼ同じ流れ
    expect {
      # フォロー、アンフォローボタンのパーシャル(_follow_area.html.slim)にはid="follow-area-#{user.id}"と定義済
      # login_userがother_userのフォローボタンをクリックしたらフォロー成功し、アンフォローボタンに変わることをテストし
      # その結果、login_userがフォローしているuser数が1増えることをテストしている
      # いいね、編集、削除ボタン(アイコン)と違って、'フォロー''アンフォロー'という文字が表示されているためそれを使って検索・操作している
      within "#follow-area-#{other_user.id}" do
        click_link 'フォロー'
        expect(page).to have_content 'アンフォロー'
      end
    }.to change{login_user.following.count}.by(1)
  end


  it 'フォローを外せること' do
    # 先にlogin_userがother_userをフォローしておく
    login_user.follow(other_user)
    # あとは'フォローができること'とほぼ同じ流れ
    visit posts_path
    expect {
      within "#follow-area-#{other_user.id}" do
        click_link 'アンフォロー'
        expect(page).to have_content 'フォロー'
      end
    }.to change{login_user.following.count}.by(-1)
  end
end
```
# 感想
各システムスペックを細かくみていくことで(その分長くなりましたが..)、テストするポイントや、よく使われる構文、書き方や流れが一緒の部分などがわかり、少しRSpec+FactoyBot+Capybaraの書き方に慣れてきた感じがします。  
このあとポートフォリオ作成に入ると初めて自分の力で0から書く事になるので、この課題とモデルスペックの課題でやったこと、アウトプットを参考に実装していければと思います。  
また、ビューのファイル内でボタンや各投稿などにidやclassを設定しておくことが、テストを書く際にも重要になることを学べました。  
将来的にはTDDをするべきなのでしょうが、今の時点だとすごく難しそうな印象です・・自分でいくつかアプリを作ってみて慣れてきたらいつかやってみようかなと思っています。
