# 15. モデルスペックを実装
### 内容
モデルスペックを実装してください  
  
### 補足
- ユーザーモデルとポストモデルのスペックは最低限書いてください。その他のモデルは任意とします。
- バリデーション, スコープ, インスタンスメソッドのスペックを書いてください。
- 人によっては本実装がサンプルアプリと異なるため、この解答例が絶対的な正解というわけではありません。

# 作業手順
- "rspec-rails"と"factory_bot_rails"のgemをbundle install
- rails generate rspec:installでRSpecのファイルもろもろを生成
- Factory_Botの設定(spec/rails_helper.rbにconfig.include FactoryBot::Syntax::Methodsを追記することでrspecのテストコード中でFactory_botのメソッドを使用する際に、クラス名の指定を省略できるようになる)
- userとpostのテストデータを作成(spec/factories/users.rbとspec/factories/posts.rbを作成、spec/factories/fixtures/fixture.pngを格納)
- RSpecのモデルスペックのテストを作成(spec/models/post_spec.rb, spec/models/user_spec.rb)
- テストが通ることを確認
[![Image from Gyazo](https://i.gyazo.com/8cd178bcbb56ca974369cb62645368c7.png)](https://gyazo.com/8cd178bcbb56ca974369cb62645368c7)

[RailsアプリへのRspecとFactory_botの導入手順](https://qiita.com/Ushinji/items/522ed01c9c14b680222c)  
[【Ruby on Rails】FactoryBotとFakerについてまとめ](https://qiita.com/mumucochimu/items/10432fd5173e63ebd418)


[rspec-rails(gem)](https://github.com/rspec/rspec-rails)  
[factory_bot_rails(gem)](https://github.com/thoughtbot/factory_bot_rails)
  
# 学んだこと
## RSpecとFactoryBotについて
Railsには、簡潔にテストを書ける`Minitest`という標準テストフレームワークがある。また、テスト用のデータ作成には、`fixture`というものがある。  
`Rspec`はMinitestに代わるテストフレームワークで、`FactoryBot`はfixtureに代わるデータ作成ライブラリ。どちらもgemで導入可能。  
MinitestでRSpec並みに凝ったことをやろうとするとプラグインをたくさん入れたり、継承、モジュール等を駆使することになるため、業務レベルのテストコードではminitestはあまり使われていないらしい。
  
[MinitestとRSpec、FixturesとFactoryGirlの良いところ悪いところをコードを書いて比較してみた](https://blog.jnito.com/entry/2015/05/06/074510)  
[RSpecとminitestのおおまかな違い](https://qiita.com/kenkenkengo-y/items/5fb3f3602dcf3826ce19)  
  
## テスト(RSpec)の書き方について
基本的な書き方は下記のサイト等を一読してイメージを掴んだ。  
全部網羅しようとすると大変な量なので、今回解答例で出てきた部分を最低限チェックした。  
  
[【初心者向け】テストコードの方針を考える（何をテストすべきか？どんなテストを書くべきか？）](https://qiita.com/jnchito/items/2a5d3e15761fd413657a)  
[使えるRSpec入門・その1「RSpecの基本的な構文や便利な機能を理解する」](https://qiita.com/jnchito/items/42193d066bd61c740612)  
[使えるRSpec入門・その2「使用頻度の高いマッチャを使いこなす」](https://qiita.com/jnchito/items/2e79a1abe7cd8214caa5)  
  
< ポイント >  
- 原則として`「1つのexampleにつき1つのエクスペクテーション」`で書いた方がテストの保守性が良くなる。
- `describe`
  - いくつでも書けるしネストさせることもできる。基本的な役割は「テストのグループ化」。適切にグループ化すると、「このdescribeブロックはこの機能をテストしてるんだな」と読み手がテストコードを理解しやすくなる。
  - 一番外側のdescribe以外はRSpec.を省略できる。
- FactoryBotのテストデータ作成のメソッド`build`と`create`の違い
  - `build` => メモリ上にインスタンスを確保する。DBにアクセスする必要がないので処理が比較的軽くなる。
  - `create`=> DB上にデータを作成する。DBにアクセスする処理のときは必須。（何かの処理の後、DBの値が変更されたのを確認する際は必要）
- `let`と`let!`の違い(letはRSpecではインスタンス変数の定義に使われる)
  - `let`  => 実際に必要になる瞬間まで呼び出されない(遅延評価される)
  - `let!` => 事前に実行される
- `subject`
  - テスト対象のオブジェクト（またはメソッドの実行結果）が明確に一つに決まっている場合はsubjectでテスト対象のオブジェクトを1箇所にまとめexampleの外(上)に引き上げることができる
  - `expect(Post.body_contain(‘hello’)).to include(post)`の`Post.body_contain(‘hello’)`部分を`subject { Post.body_contain('hello') }`として宣言した場合、`is_expected.to include post`のように`is_expected.to`を使った書き方に変わる
- `it`に渡す文字列は省略できる。`subject`と`is_expected.to`を組み合わせることで`it { is_expected.to include XXXXXX }`のように自然な英文のような表現ができる
- `context` => 何パターンかを条件別にグループ化する時に使う  
例) context 'フォローしている場合' do ... end　や、 context 'フォローしていない場合' do ... end など    
- `change + by` => 元の個数はともかく1個減る(または増える)ことを検証できる。  
expect { X }.to change { Y }.by(1) => XすることでYが1増えることを期待  
※ブロック{}を渡すことに注意  
例) expect { user_a.like(post_by_user_b) }.to change { Like.count }.by(1)  
- `before` => before do ... end で囲まれた部分はexampleの実行前に毎回呼ばれる。  
beforeブロックの中では、テストを実行する前の共通処理やデータのセットアップ等を行うことが多い。  
- `to`と`not_to` 
  - to => 「～であること」を期待する場合は`to`を使う。 例) expect(1 + 2).to eq 3
  - 反対に「～ではないこと」を期待する場合は`not_to`を使う。 例) expect(1 + 2).not_to eq 1
  
<参考>  
[FactoryBot(FactoryGirl)チートシート](https://qiita.com/morrr/items/f1d3ac46b029ccddd017)  
[FactoryBotにおけるcreateとbuildの違い](https://qiita.com/takutotacos/items/4a8bd25ccb42141bf67c)  
[Railsガイド - errors[]](https://railsguides.jp/active_record_validations.html#errors)  
[subject を使ってテスト対象のオブジェクトを1箇所にまとめる](https://qiita.com/jnchito/items/42193d066bd61c740612#subject-%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%83%86%E3%82%B9%E3%83%88%E5%AF%BE%E8%B1%A1%E3%81%AE%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%82%921%E7%AE%87%E6%89%80%E3%81%AB%E3%81%BE%E3%81%A8%E3%82%81%E3%82%8B)  
[change + from / to / by](https://qiita.com/jnchito/items/2e79a1abe7cd8214caa5#change--from--to--by)  
[context で条件別にグループ化する](https://qiita.com/jnchito/items/42193d066bd61c740612#context-%E3%81%A7%E6%9D%A1%E4%BB%B6%E5%88%A5%E3%81%AB%E3%82%B0%E3%83%AB%E3%83%BC%E3%83%97%E5%8C%96%E3%81%99%E3%82%8B)  
[before で共通の前準備をする](https://qiita.com/jnchito/items/42193d066bd61c740612#before-%E3%81%A7%E5%85%B1%E9%80%9A%E3%81%AE%E5%89%8D%E6%BA%96%E5%82%99%E3%82%92%E3%81%99%E3%82%8B)  
[to / not_to / to_not](https://qiita.com/jnchito/items/2e79a1abe7cd8214caa5#to--not_to--to_not)
  
下記は今回のRSpecの定義内容↓
```rb
# spec/models/post_spec.rb

# rspecを定義するファイルには必ず必要
require 'rails_helper'

# describe PostでPostクラス(モデル)をテストすることを明示。モデルのテストなのでtype: :modelと書いておく
RSpec.describe Post, type: :model do
  # バリデーションのテストをグループ化している
  describe 'バリデーション' do 
    # itはテストをexampleという単位にまとめる役割。結果が期待通りになればテストにパスしたことになる。
    it '画像は必須であること' do
      # FactoryBotの定義内容にオーバーライド(上書き)して、imegesをnilにして生成している
      # buildはDBにデータは作成せず、メモリ上にインスタンスを確保する
      post = build(:post, images: nil)
      # バリデーションをチェック
      post.valid?
      # errors[:images]で特定の属性imegesについてエラーメッセージをチェックし
　　　エラーメッセージに'を入力してください'が含まれているが確認している
      expect(post.errors[:images]).to include('を入力してください')
    end

    # 画像(上記)と同じ 
    it '本文は必須であること' do
      post = build(:post, body: nil)
      post.valid?
      expect(post.errors[:body]).to include('を入力してください')
    end

    it '本文は最大1000文字であること' do
      # aを1001文字分入れている
      post = build(:post, body: "a" * 1001)
      # バリデーションをチェック
      post.valid?
      # :bodyに対するエラーメッセージに'は1000文字以内で入力してください'が含まれていることを確認
      expect(post.errors[:body]).to include('は1000文字以内で入力してください')
    end
  end
  
  # スコープのテストをグループ化している
  describe 'スコープ' do
    # scope :body_containをテストすることを明示
    describe 'body_contain' do
      # let!で先にdbにデータを作成しておく。bodyは指定する。
      let!(:post) { create(:post, body: 'hello world') }
      # subjectを使ってテスト対象をまとめて(引き上げて)いる
      subject { Post.body_contain('hello') }
      # itに渡す文字列を省略し、subjectとの組み合わせで、is_expected.toで自然な英語になるようにしている
      it { is_expected.to include post }
    end
  end
end
```
```rb
# spec/models/user_spec.rb

# rspecを定義するファイルには必ず必要
require 'rails_helper'

RSpec.describe User, type: :model do
  # バリデーションのテストをグループ化している
  describe "バリデーション" do
    # post_specと同様
    it 'ユーザー名は必須であること' do
      user = build(:user, username: nil)
      user.valid?
      expect(user.errors[:username]).to include('を入力してください')
    end

    it 'ユーザー名は一意であること' do
      # createでuserをdbに作成しておく
      user = create(:user)
      # ユーザ名がかぶっているユーザをあえてbuildしvalid?でチェック
      same_name_user = build(:user, username: user.username)
      same_name_user.valid?
      # 'はすでに存在します'というエラーメッセージが出ることを確認
      expect(same_name_user.errors[:username]).to include('はすでに存在します')
    end

    # post_specと同様
    it 'メールアドレスは必須であること' do
      user = build(:user, email: nil)
      user.valid?
      expect(user.errors[:email]).to include('を入力してください')
    end

    # ユーザー名は一意であることと同様
    it 'メールアドレスは一意であること' do
      user = create(:user)
      same_email_user = build(:user, email: user.email)
      same_email_user.valid?
      expect(same_email_user.errors[:email]).to include('はすでに存在します')
    end
  end
  
  # インスタンスメソッドのテストをグループ化している
  describe 'インスタンスメソッド' do
    # まずuser_a,b,cをDBに作成する
    let(:user_a) { create(:user) }
    let(:user_b) { create(:user) }
    let(:user_c) { create(:user) }
    # user_a,b,cに紐づくpostをDBに作成する
    let(:post_by_user_a) { create(:post, user: user_a) }
    let(:post_by_user_b) { create(:post, user: user_b) }
    let(:post_by_user_c) { create(:post, user: user_c) }
    
    # own?のテストをグループ化
    describe 'own?' do
      # contextで条件ごとにグループ化
      context '自分のオブジェクトの場合' do
        it 'trueを返す' do
          # to be trueでtrueになることを確認
          expect(user_a.own?(post_by_user_a)).to be true
        end
      end

      context '自分以外のオブジェクトの場合' do
        it 'falseを返す' do
          # to be falseでfalseになることを確認
          expect(user_a.own?(post_by_user_b)).to be false
        end
      end
    end

    # likeのテスト
    describe 'like' do
      it 'いいねできること' do
        # chenge + by を使っている
        # expect { X }.to change { Y }.by(1) => XすることでYが1増えることを期待している
        expect { user_a.like(post_by_user_b) }.to change { Like.count }.by(1)
      end
    end

    # unlikeのテスト。likeの書き方とほぼ同じ。
    describe 'unlike' do
      it 'いいねを解除できること' do
        # まずuser_aがuser_bのpostをlikeする
        user_a.like(post_by_user_b)
        # unlikeでは.by(-1)となる。
        expect { user_a.unlike(post_by_user_b) }.to change { Like.count }.by(-1)
      end
    end

    # followのテスト。likeの書き方と同様
    describe follow' do
      it 'フォローできること' do
        expect { user_a.follow(user_b) }.to change { Relationship.count }.by(1)
      end
    end

    # unfollowのテスト。unlikeの書き方と同様
    describe 'unfollow' do
      it 'フォローを外せること' do
        user_a.follow(user_b)
        expect { user_a.unfollow(user_b) }.to change { Relationship.count }.by(-1)
      end
    end

    # following?のテスト。own?の書き方とほぼ同じ
    describe 'following?' do
      context 'フォローしている場合' do
        it 'trueを返す' do
          # まずuser_aがuser_bをフォローする、結果はtrueになる
          user_a.follow(user_b)
          expect(user_a.following?(user_b)).to be true
        end
      end

      context 'フォローしていない場合' do
        it 'falseを返す' do
          # まずuser_aはuser_bをフォローしていないので結果はfalseになる
          expect(user_a.following?(user_b)).to be false
        end
      end
    end

    # feedのテスト
    describe 'feed' do
      # 3つのexample(it ~ の部分)に共通する事前処理(user_aがuser_bをフォローする)をbeforeで定義
         各exampleの前に毎回実行される
      before do
        user_a.follow(user_b)
      end
      # subjectを使って共通のテスト対象(user_aのフィードを取得)をexample上にまとめて(引き上げて)いる
      subject { user_a.feed }
      # 以下は3つのexample
      # itに渡す文字列を省略し、subjectとの組み合わせで、is_expected.toで自然な英語になるようにしている
      it { is_expected.to include post_by_user_a }      # user_a(自身)のpostがfeedに含まれているかをチェック
      it { is_expected.to include post_by_user_b }      # user_b(フォローした相手)のpostがfeedに含まれているかをチェック
      it { is_expected.not_to include post_by_user_c }  # user_c(フォローしていないuser)のpostがfeedに含まれていないことをチェック
    end
  end
end
```

## アソシエーションの場合のFactoryBotの書き方
PostモデルがUserモデルに対してbelongs_toの関係にあり、factoryで定義している名称(user)がモデル名(user)と一致しているので、`spec/factories/posts.rb`で`user`と書くだけで`association :user, factory: :user`と同じになる。  
  
[Factorybotのアソシエーション(belongs_to)](https://qiita.com/jonson29/items/3798b79d5ca77fd711db)  
  
下記は今回のFactoryBotのテストデータ定義内容↓  
```rb
# spec/factories/users.rb

FactoryBot.define do
    factory :user do
      # Fakerを使い設定している
      email { Faker::Internet.unique.email }
      password { '12345678' }
      password_confirmation { '12345678' }
      username { Faker::Name.name }
    end
  end
```
```rb
# spec/factories/posts.rb

FactoryBot.define do
    factory :post do
      # Fakerを使い設定している
      body { Faker::Hacker.say_something_smart }
      # fixturesというディレクトリを掘ってそこに画像を置いておき、画像ファイルのパスをopenしてそれを渡す
      # 複数画像を扱っているのでなので[]で囲っている 
      images { [File.open("#{Rails.root}/spec/fixtures/fixture.png")] }
      # userと書くだけでassociation :user, factory: :userと同じ
      user
    end
  end
```
  
## その他
`transient`(FactoryBot)という実際に作成するデータと直接関係無い新しいattributeを定義する機能もある。  
そこで定義されたものは実際のmodelにはセットされないしattributes_forでも出力されない。何のために使うかというと作成時に挙動を変更するためのフラグや追加データとして利用するのが一般的。  
```
factory :user do
  transient do
    rockstar true
    upcased  false
  end

  name  { "John Doe#{" - Rockstar" if rockstar}" }
  email { "#{name.downcase}@example.com" }

  after(:create) do |user, evaluator|
    user.name.upcase! if evaluator.upcased
  end
end


# 以下により、userのfactoryが生成されるだけでなく、name属性が大文字になる
# => "JOHN DOE - ROCKSTAR"
create(:user, upcased: true).name
```
[FactoryGirlのinheritanceやtransientの確認をする](https://woshidan.hatenablog.com/entry/2014/09/15/002017)
  

  
# 感想
以前1回RSpecで書いたことはありましたが、今回ほど細かくは見なかったので、復習するいい機会になりました。  
特に、初心者がテストコードを書くときに気をつけることを説明した参考サイトを先に読んだことである程度の書き方のイメージがつきました。応用で使うような技術を盛り込見過ぎて読みにくくしてもダメで、そこに時間を使いすぎるのもダメ。とにかくわかりやすく書くのが大事なんだなと感じました。  
細かいRSpecの構文や機能などはこの先ポートフォリオ等で自分でテストコードを書く際に、必要に応じて調べつつやろうと思いました。今回はひとまず基本の基本を学べたのでよかったです。  
