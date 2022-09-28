# 14.SEO対策 メタタグの設定
### 内容
- SEO対策として必須であるメタタグの設定をしましょう
- gem 'meta-tags'で実装しましょう。

### 補足
- title, description, keywordが適切に設定されるようにしましょう。
[![Image from Gyazo](https://i.gyazo.com/399f3fec50a34cfb4f99cb0683a29b88.png)](https://gyazo.com/399f3fec50a34cfb4f99cb0683a29b88)

- ngrokを使うと一時的にインターネット上にサイトを公開できるのでSlackに投稿した時にメタタグが本当に反映されているか確認できます。

# 作業手順
- meta-tags(gem)の導入(Gemfile, Gemfile.lock)
- % bundle exec rails g meta_tags:installで設定ファイルの生成。
- 共通部分の設定
  - application_helper.rbにdefault_meta_tagsメソッドを定義し、application.html.slimで呼び出す(config/settings.ymlに定数も定義、ogp.pngの画像を保存等)
- 各ページごとの設定
  - set_meta_tagsを使って各ページのtitleやdescriptionを設定
- ngrokを使って実際に確認

[meta-tags(gem)](https://github.com/kpumuk/meta-tags)  
[【Rails】『meta-tags』gemを使ってSEO対策をおこなう方法](http://vdeep.net/rubyonrails-meta-tags-seo)  

# 学んだこと
## SEO対策とは
SEOとは「Search Engine Optimization（検索エンジン最適化）」の略語。  
特定の検索ワード、すなわち検索クエリの回答として「このページこそが最適だ」と検索エンジンから評価してもらい、検索結果で上位表示を目指す施策。  
国内で使われる検索エンジンのベースは、約97％がGoogle、つまり検索エンジン＝Googleなので、Googleが公式に出している下記のスターターガイドを参考にすべし。  
  
[【2022年版】SEOとは？初心者が上位表示を目指す14の対策](https://blog.hubspot.jp/what-is-seo)  
[検索エンジン最適化（SEO）スターター ガイド](https://developers.google.com/search/docs/beginner/seo-starter-guide?hl=ja)  
  
## メタタグとは
SEO対策の内の1つで、ユーザーではなく、検索エンジンやブラウザなどのシステムに対して情報（メタデータ）を伝えるHTMLタグです。htmlファイルのhead内に設定する。　　
今課題においてはtitle, description, keywordを設定。また、meta-tags(gem)を使うことでより簡単に設定できるようになる。  
metaタグではgoogleやその他の検索エンジンに情報を渡したり、Google検索に載せたくないページを決めたり、OGP(Open Graph Protocol)設定をしたり、内部のファイルに向けてはどの文字コードを使うべきか、あるいは、スマートフォン向けに表示方法を指定したりする。  
  
[![Image from Gyazo](https://i.gyazo.com/a60e963a009de4092d411f978d74a435.png)](https://gyazo.com/a60e963a009de4092d411f978d74a435)  
[![Image from Gyazo](https://i.gyazo.com/e77cd4bf34fcc0e137a657aec382fcc2.png)](https://gyazo.com/e77cd4bf34fcc0e137a657aec382fcc2)  
  
[【HTML】メタタグとは？基本的なmetaタグの使い方と書き方を解説【SEO効果についても】](https://creive.me/archives/14308/#i)  
  
## config/settings.yml
すべての環境で利用する定数を定義する。configのgemにて使用可能の機能。※以前課題でやった  
今回はapplication_helper.rbに定義するdefault_meta_tagsメソッド内で使う定数を定義している。  
  
[gem configを理解する ~ configを使った定数管理の方法](https://qiita.com/tanutanu/items/8d3b06d0d42af114a383)  
  
## 共通の設定(default_meta_tagsメソッドとconfig/settings.ymlの内容)
```rb
# app/helpers/application_helper.rb

module ApplicationHelper
  def default_meta_tags
    {
      # サイト名
      site: Settings.meta.site,
      # trueを設定することで「タイトル | サイト名」の並びで出力してくれる
      reverse: true,
      # ページのタイトル名
      title: Settings.meta.title,
      # ページの説明
      description: Settings.meta.description,
      # 検索ワード
      keywords: Settings.meta.keywords,
      # canonicalタグとは、サイト内で複数ある重複コンテンツを1つのURLにまとめたURL（正規化URL）をGoogleの検索エンジンに認識させるタグのこと
      # request.original_urlで現在のリクエストURLのString("http://localhost:3000/")を取得する
      canonical: request.original_url,
      # OGP(Open Graph Protocol)とはWebサイトのコンテンツの内容をSNSで伝える際に使用する仕組み
　　　　　　　　　　　　#　　SNSでWebの記事をシェアしたときにその記事のURLとタイトル、簡単な内容やサムネイル画像がボックスにまとめられて表示されるが、以下はその設定に使う
      og: {
        # :full_title とすると、サイトに表示される <title> と全く同じものを表示できる
        title: :full_title,
        # [‘website’, ‘article’, ‘blog’, …]などからひとつ設定
        type: Settings.meta.og.type,
        # URLを設定(サイトの方の設定と同じ)
        url: request.original_url,
        # シェア用の画像を設定
        # image_urlでapp/assets/imagesディレクトリの下に置かれている画像アセットへのURL("http://localhost:3000/assets/ogp-2f2030aa1b072169461a89a85796957bc9a3f004b8ccbf89c106439978c05e37.png")を算出する
        image: image_url(Settings.meta.og.image_path),
        # サイト名を設定(サイトの方の設定と同じ)
        site_name: :site,
        # ページの説明の設定(サイトの方の設定と同じ)
        description: :description,
        # リソースの言語を設定
        locale: 'ja_JP'
      },
      # Twitterでリンクを表示させる場合のカードの種類を設定
      # 'summary'の方が画像が小さい
      twitter: {
        card: 'summary_large_image'
      },
    }
  end
end
```
```yaml
# config/settings.yml

meta:
  site: InstaClone
  title: InstaClone - Railsの実践的アプリケーション
  description: Ruby on Railsの実践的な課題です。Sidekiqを使った非同期処理やトランザクションを利用した課金処理など実践的な内容が学べます。
  # yamlファイルで-(ハイフン)は配列を作る
  keywords:
    - Rails
    - InstaClone
    - Rails特訓コース
  og:
    type: website
    image_path: ogp.png
```
  
## 各ページごとの設定について
`set_meta_tags`を使うと各ページ(ビュー)で上書きしたい設定を書くことができる。  
以下、投稿詳細ページの例。  
```rb
# app/views/posts/show.html.slim(投稿詳細ページ)
# titleとdescriptionを上書きしている
# "#{request.base_url}#{@post.images.first.url}"では"http://localhost:3000/uploads/post/images/13/test.png"のようなURLを取得できる  
- set_meta_tags title: '投稿詳細ページ', description: @post.body,
        og: { image: "#{request.base_url}#{@post.images.first.url}"} # 本来carrierwaveのasset_pathで設定すべき
```
  
## ngrokについて
ローカルPC上で稼働しているネットワーク（TCP）サービスを外部公開できるサービス。ローカルPCのWebサーバを外部公開することができる。  
Slackやtwitterに投稿した時に今回のメタタグの設定が本当に反映できているかどうかを確認できる。  
手順は以下を参考にした。  
1. ngrokの公式サイトでユーザ登録、ログインする  
2. 公式サイトからngrokをインストールしzipファイルを解凍  
3. 公式サイトの「Connect your account」に記載されているngrokのコマンドを実行し、認証トークンで自分のアカウントを連携する  
4. `rails s`でアプリを起動した状態で`ngrok http 3000`を実行し、Forwardingに書いてあるurlにアクセスすると公開されたものが確認できる  
  
[ngrokが便利すぎる](https://qiita.com/mininobu/items/b45dbc70faedf30f484e)  
[ngrok(公式サイト)](https://dashboard.ngrok.com/get-started/setup)
  
SlackとTwitterでメタタグの設定が反映できていることを確認↓
[![Image from Gyazo](https://i.gyazo.com/a7d95e04da713be8230ff3830d11bdf7.png)](https://gyazo.com/a7d95e04da713be8230ff3830d11bdf7)  
[![Image from Gyazo](https://i.gyazo.com/dd6466febcdb6a0de666fd0f195e8326.png)](https://gyazo.com/dd6466febcdb6a0de666fd0f195e8326)  
  
# 感想
SEO対策に関するメタタグの基本的な設定は今回の課題でイメージが出来ました。設定内容は解答例に書いてあった項目のみ確認して概要を掴んだレベルで、載っていない部分は深堀は特にしませんでした。  
また、参考サイトによるとkeywordは現在SEO対策に一切寄与しておらず、Googleもそのことを公式に発表していることなどから、今回のメタタグの設定は最低限の設定であって、検索結果で上位表示を目指すための本気のSEO対策をするのであれば、高品質なコンテンツを作る（コンテンツSEO）
、サイトの内部を整備する（内部対策）、他のメディアなど外部から評価してもらう（外部対策）などの対策やGoogleが公式に出しているガイドの参考などが不可欠なんだと知りました。  
SEOをぼんやり知っていたけれど、奥深いなあと思い、色々調べるきっかけになりました。  
