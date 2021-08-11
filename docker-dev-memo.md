# Docker開発練習メモ
source 'https://rubygems.org'

gem 'rails', '~> 6.1.0'

docker-compose run web rails new . --force --database=mysql
Gemfileが更新されたら、ビルド
  docker-compose build 
docker-compose run web rails db:create
docker-compose up 



## 事前準備 
  githubに登録
    git config を設定
  herokuに登録
    クレカも登録する。herokuでは、mySQLをアドオンとして追加する。登録しないとmySQLがアドオンとして追加できない
  Hroku CLIをインストール

## Herokuにログイン
  heroku login 
  heroku container:login
## Herokuアプリを作成
  heroku create rails-docker-dik 

## DBを追加・設定
  heroku addons:create cleardb:ignite -a rails-docker-dik
  heroku config -a rails-docker-dik
    CLEARDB_DATABASE_URL: mysql://bd1bbf73b5e132:c899dd1b@us-cdbr-east-04.cleardb.com/heroku_6aee44e8b6a8ef4?reconnect=true
    `mysql://<user-name>:<password>@<host-name>/<database-name>`
  heroku configに設定
    heroku config:add APP_DATABASE='heroku_6aee44e8b6a8ef4' -a rails-docker-dik
    heroku config:add APP_DATABASE_USERNAME='bd1bbf73b5e132' -a rails-docker-dik
    heroku config:add APP_DATABASE_PASSWORD='c899dd1b' -a rails-docker-dik
    heroku config:add APP_DATABASE_HOST='us-cdbr-east-04.cleardb.com' -a rails-docker-dik
    heroku config -a rails-docker-dik
  
## Dockerfileを本番環境用に修正
  start.shに本番環境特有の処理をさせる
  assets:precompileが本番で起動するようになる
    heroku config:add RAILS_SERVE_STATIC_FILES='true' -a rails-docker-dik 
  Change boot timeout
    https://tools.heroku.support/limits/boot_timeout
    => 120 secに変更 
  server.pidが残っていると、herokuでエラーになる
    rm src/tmp/pids/server.pid

## Dockerイメージをビルド・リリース
  Dockerイメージをビルドして、コンテナレジストリにプッシュ
    heroku container:push web -a rails-docker-dik
  Dockerコンテナをheroku上にリリース
    heroku container:release web -a rails-docker-dik
  データベースのテーブルを更新したい時
    heroku run bundle exec rake db:migrate RAILS_ENV=production -a rails-docker-dik
      Error: Exec format error
      => $ heroku ps -a rails-docker-dik 
      => web.1: crashed 2021/08/08 13:02:40 +0900 (~ 52s ago)
  ## エラー対応 ##
    マルチプラットフォームのイメージビルド
    $ docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-dik:latest
    [+] Building 750.0s (12/12) FINISHED １２分かかった
    イメージにタグを付ける (イメージのコピー)
      docker tag d.shinmachi/rails-docker-dik registry.heroku.com/rails-docker-dik/web
    プッシュする
      docker push registry.heroku.com/rails-docker-dik/web
    リリースする
      heroku container:release web -a rails-docker-dik
    データベースのテーブルを更新
      heroku run bundle exec rake db:migrate RAILS_ENV=production -a rails-docker-dik
    通った！！
  herokuのアプリを開く
    $ heroku open -a rails-docker-dik 
  ログをコンフィグに設定
    $ heroku config:add RAILS_LOG_STDOUT='true' -a rails-docker-dik
  ログを表示
    heroku logs -t -a rails-docker-dik

## 機能追加
  コントローラーを作る
    docker-compose exec web bundle exec rails g controller users
  ルーティングの設定 routes.rb 

  heroku container:push web -a rails-docker-dik
  heroku container:release web -a rails-docker-dik
  heroku open -a rails-docker-dik 

  docker-compose down
  rm src/tmp/pids/server.pid
  docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-dik:latest
  docker tag d.shinmachi/rails-docker-dik registry.heroku.com/rails-docker-dik/web
  docker push registry.heroku.com/rails-docker-dik/web
  heroku container:release web -a rails-docker-dik
  heroku open -a rails-docker-dik 

# CI/CD を構築する (circleciを使用)
## GitHub 
  - git init 
  - mv src/.gitingore .
  - mv src/.gitattributes .
  - ls -a src/
  - rm -rf src/.git/
## CI 
  ## テストコードを記載
  $ docker-compose exec web bundle exec rake test
  ## CircleCIに登録
    github アカウントで登録
  ## プロジェクト を登録
    branch ci を選択してセットアップ
  ## configを設定
    circleci-cliをインストール=> config.ymlのバリデーションのため
      curl -fLSs https://raw.githubusercontent.com/CircleCI-Public/circleci-cli/master/install.sh | bash
    config.ymlのバリデーション
      circleci config validate
  ## 環境変数を設定
  ## GitHubにプッシュ
  ## テストを修正
## CD
 - configを修正
 - 環境変数を設定
 - Viewファイルを修正
 - GitHubにプッシュ
 - マージ、デプロイ

Tips
dockerイメージのコピー（タグ名変更）
  docker tag [対象イメージ名:タグ] [変更後イメージ名:タグ]

M1の場合、Docker イメージをビルドする時は、プラットフォームの指定に気をつけないといけない。
  $ docker buildx build . --platform linux/amd64 -t d.shinmachi/rails-docker-dik:latest

git switch -c <branch-name>
  checkoutの代わりに、switchが使えるようになった

git push origin <branch-name>
  GitHubのブランチにpushできる
  たとえ、mainでコミットしても、リモートのブランチにプッシュできる



