# CircleCIの導入
下記などを参考に`rubocop`をgit pushされるごとに自動に回すようにした。  
rspecの設定はまだ。  

[CI初心者がCircleCIでRubocopとRSpecの設定をするために急いで学んだこと](https://qiita.com/tanutanu/items/9da75ca35bf71b04e39f)  
[【Rails】GithubとCircleCIを連携してcommit時にrspecとrubocopを動かす](https://qiita.com/junara/items/a40bb231c405be7983f7)  
[怖くない CircleCI の config.yml の書き方](https://qiita.com/kasaharu/items/bfeb2a41b9d636388531)  
[【Rails × CircleCI】初心者がCI/CD設定で躓いたエラー集と解決法](https://qiita.com/Yuya-hs/items/c1e1b40ee47f0ea136bf#nomethoderror-cannot-load-database-configuration)  
  
## 1. CircleCIの登録とgithubとの連携をする
[【CircleCI】CircleCI 2.0からはじめる個人での簡単なCI導入方法 - githubとの連携まで](https://www.tweeeety.blog/entry/2018/02/09/195345)などを参考にCircleCIに登録。  
[CircleCI Sign UP](https://circleci.com/signup)　　
　　
## 2. `.circleci/config.yml`を作成しgithubにpushし、CircleCI側で連携する  
下記のように設定  
※注意点、ハマりポイントは下に記載
```
.circleci/config.yml
# 最新のCircleCI 2.1 を使用します
version: 2.1
jobs: # 一連のステップ
  build: # ワークフローを使用しない実行では、エントリポイントとして `build` ジョブが必要です
    parallelism: 1 # 何個並列でCIを走らせるか。無料版だと1しか無理。
    docker: # Docker でステップを実行します
      - image: cimg/ruby:3.1-node # このイメージをすべての `steps` が実行されるプライマリ コンテナとして使用します
        environment: # プライマリ コンテナの環境変数
          BUNDLE_JOBS: 3
          BUNDLE_RETRY: 3
          BUNDLE_PATH: vendor/bundle
          RAILS_ENV: test
          DB_HOST: 127.0.0.1
      - image: redis:latest
      - image: cimg/mysql:8.0 # データベース イメージ
        environment: # データベースの環境変数
          MYSQL_DB: circle_test
          MYSQL_ALLOW_EMPTY_PASSWORD: yes
    steps: # 実行可能コマンドの集合
      - checkout # ソース コードを作業ディレクトリにチェックアウトする特別なステップ

      # Bundler のバージョンを指定します
      - run:
          name: setup bundler
          command: |
            gem install bundler:2.3.7
            bundle -v
          # バンドル キャッシュを復元します
          # 依存関係キャッシュについては https://circleci.com/ja/docs/2.0/caching/ をお読みください

      - restore_cache:
          keys:
            - rails-demo-bundle-v2-{{ checksum "Gemfile.lock" }}
            - rails-demo-bundle-v2-

      # Ruby の依存関係をインストールします
      - run: 
          name: bundle install
          command: bundle check --path vendor/bundle || bundle install --deployment

      # Ruby の依存関係のバンドル キャッシュを保存します
      - save_cache:
          key: rails-demo-bundle-v2-{{ checksum "Gemfile.lock" }}
          paths:
            - vendor/bundle

      # アプリケーションで Webpacker または Yarn を他の何らかの方法で使用する場合にのみ必要です
      - restore_cache:
          keys:
            - rails-demo-yarn-{{ checksum "yarn.lock" }}
            - rails-demo-yarn-

      - run:
          name: yarn install
          command: yarn install --cache-folder ~/.cache/yarn

      # Yarn または Webpacker のキャッシュを保存します
      - save_cache:
          key: rails-demo-yarn-{{ checksum "yarn.lock" }}
          paths:
            - ~/.cache/yarn

      - run:
          name: wait database
          command: dockerize -wait tcp://localhost:3306 -timeout 1m

      - run:
          name: setup database.yml
          command: cp config/database.ci.yml config/database.yml

      - run:
          name: setup database
          command: bin/rails db:schema:load --trace

      - run:
          name: Rubocop
          command: bundle exec rubocop
```

## 3. ハマりポイント&注意点列挙

- `.circleci/config.yml`ファイルのスペーシングが間違っていた。
  [CircleCi expected type: String, found: Mapping エラー workflow でハマったときの対処方法](https://qiita.com/GENYA/items/f29a00fc05f08a5c4d5e
)  
  
- `MYSQL_USER=“root”`をremoveしろとのエラー
  ```
  <エラー内容>
  2022-08-11 08:15:17+00:00 [ERROR] [Entrypoint]: MYSQL_USER="root", MYSQL_USER and MYSQL_PASSWORD are for configuring a regular user and cannot be used for the root user
      Remove MYSQL_USER="root" and use one of the following to control the root user password:
      - MYSQL_ROOT_PASSWORD
      - MYSQL_ALLOW_EMPTY_PASSWORD
      - MYSQL_RANDOM_ROOT_PASSWORD
  ```
  => [こちら](https://teratail.com/questions/g82650lt0blom7)にあるように削除したら解消。  
  
- Rubyのバージョンの齟齬があるというエラー
  ```
  <エラー内容>
  #!/bin/bash -eo pipefail
  bundle check --path vendor/bundle || bundle install --deployment
  Your Ruby version is 2.6.10, but your Gemfile specified 3.1.2
  ```
  => Docker imageの選択を、自分のRubyのバージョンに合わせて変えてなかったため。  
  [こちらの記事](https://www.moncefbelyamani.com/how-to-run-ruby-2-7-6-in-circleci/)で気づいた。  
  [イメージタグ](https://circleci.com/developer/images/image/cimg/ruby)の中から選んで`image: cimg/ruby:3.1-node`に変えたら解消。  
  
  ※[ruby 系で yarn ないし nodejs 系のコマンドを利用したい場合には、 -node がついているイメージを利用する必要がある](https://ja.stackoverflow.com/questions/77777/circleci実行時-command-not-found-yarnがインストールできません)とあるのでnodeのものを選ぶようにする。  
  最初、 `image: cimg/ruby:3.1.2`でやったらyarnコマンドがエラーになったはず(多分・・)  
  
- `image: cimg/mysql:8.0`の部分も自分のMySQLバージョンに変更する
  [イメージタグ](https://circleci.com/developer/ja/images/image/cimg/mysql)から選択  

- bundlerのバージョンも自分のと合わせておく必要がある。
  ```
   - run:
     name: setup bundler
     command: |
       gem install bundler:2.3.7
       ...
  ```
  => 最初、これやらなかったからか[こちら](https://qiita.com/kobayashiryou/items/0d50e25cb29c475db229)や[こちら](https://marketing-web.hatenablog.com/entry/--add-platform_x86_64-linux
)のようなエラーが出た。(多分・・)  
  
- `setup database`の部分で`undefined method 'Twitter.key' for nil:NilClass`のエラー
  => `config/initializers/sorcery.rb`に設定している
  ```
    config.twitter.key = Settings.twitter.access_key
    config.twitter.secret = Settings.twitter.secret_key
    config.twitter.callback_url = Settings.twitter.callback_url
    config.twitter.user_info_mapping = {
      username: 'screen_name'
    }
  ```
  の部分でエラーが出ていた。  
  CircleCIはtest環境で`settings/test.yml`を見ているのに記載がなかったため`settings/development.yml`と同じ内容を設定し、解消。  
  
- `undefined method `[]' for nil:NilClass (NoMethodError`のエラー
  ```
  <エラー内容>
  #!/bin/bash -eo pipefail
  bin/rails db:schema:load --trace
  (erb):2:in `<main>': undefined method `[]' for nil:NilClass (NoMethodError)
  	from /usr/local/lib/ruby/3.1.0/erb.rb:905:in `eval'
	  from /usr/local/lib/ruby/3.1.0/erb.rb:905:in `result'
  ```
  => `master.key`の設定をCircleCI側にする必要があった
  [こちら](https://qiita.com/Yuya-hs/items/c1e1b40ee47f0ea136bf#nomethoderror-cannot-load-database-configuration)や[こちら](https://qiita.com/waniwaninowani/items/125d5bdbfc7a764f1c32#circleci%E3%81%B8%E3%81%AE%E7%99%BB%E9%8C%B2)を参考に設定して解消。  
  
