# データベースの環境設定概要

CircleCI 2.0ではデータベースの設定を明示的に宣言しなければならない。
複数のpre-buildまたはカスタムイメージが利用されるからである。


例えば、RailsでPostgreSQLを利用する場合はこのようなDB接続設定のと処理ステップが必要となる。

```
version: 2
jobs:
  build:
    working_directory: ~/appName

    # dockerイメージとしてrubyイメージとPostgresSQLのイメージを利用
    docker:
      - image: ruby:2.3.1
        environment:
        - PG_HOST=localhost
        - PG_USER=ubuntu
        - RAILS_ENV=test
        - RACK_ENV=test
      - image: postgres:9.5
        environment:
        - POSTGRES_USER=ubuntu
        - POSTGRES_DB=db_name

    # ????この辺りのホスト指定がlocalhostになっているのは単一のPod内でRailsとPostgresが動いているから...??
    steps:
      - checkout
      - run:
          name: Install System Dependencies
          command: apt-get update -qq && apt-get install -y build-essential postgresql libpq-dev nodejs rake
      - run:
          name: Install Ruby Dependencies
          command: bundle install
      - run: 
          name: Set up DB
          command: |
            bundle exec rake db:create db:schema:load --trace
            bundle exec rake db:migrate
        environment:
          DATABASE_URL: "postgres://ubuntu@localhost:5432/db_name"
```

# バイナリの利用
pg_dumpやpg_restoreを利用するには、PostgresSQLのユーティリティを呼び出せる必要がある。
以下のようにして指定する。

```
     steps:
    # Add the Postgres 9.6 binaries to the path.
       - run: echo ‘/usr/lib/postgresql/9.6/bin/:$PATH’ >> $BASH_ENV

```

# Passwordなしでのログイン設定

パスなしログインをでいるようにするには以下のようにしなければいけない。
((既存ユーザを新しいユーザとパスワードで上書き))

```
steps:
  - run:
# Add a password to the `ubuntu` user, since Postgres is configured to
# always ask for a password without sudo, and fails without one.
    command: sudo -u postgres psql -p 5433 -c "create user ubuntu with password ‘ubuntu’;”
    command: sudo -u postgres psql -p 5433 -c "alter user ubuntu with superuser;"

# Create a new test database.
    command: sudo -u postgres psql -p 5433 -c "create database test;"
```

# dockerizeの利用

複数コンテナを利用している場合、コンテナの起動順序が影響することがある。
例えばここまでのr例だと、railsコンテナが立ち上がる前にpostgresコンテナが起動していなければならない。

この場合、`dockerize`を利用する。

```
version: 2.0
jobs:
  build:
    working_directory: /your/workdir
    docker:
    # イメージとして複数指定
      - image: your/image_for_primary_container
      - image: postgres:9.6.2-alpine
        environment:
          POSTGRES_USER: your_postgres_user
          POSTGRES_DB: your_postgres_test
    steps:
      - checkout
      - run:
        # dockerizeをインストールする。
          name: install dockerize
          command: wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz
          environment:
            DOCKERIZE_VERSION: v0.3.0
      - run:
          name: Wait for db
          command: dockerize -wait tcp://localhost:5432 -timeout 1m
```

この例では、アプリケーションのコンテナ(your/image_~~)をpostgresSQLのコンテナ(postgres:9.6.2-1lpine)
と一緒に立ち上げてdockerizeをインストール後、DBの起動を待機する形になっている。

## (補足) dockerizeについて
docker-composeのdepends_onはコンテナの起動までで、その内部のサービスの起動までは待ってくれない。

```
dockerize -wait <<待つ対象サービスのURL>> <<コマンド>>
```

とすることで、コマンド実行後にに対象サービスのURLが有効になるまで待機してくれる。


例えばDBを待つことが多いと思うのでこのようになるはず。
```
# MySQLの場合
dockerize -wait tcp://localhost:3306 -timeout 1m
# Redisの場合
dockerize -wait tcp://localhost:6379 -timeout 1m
# なんかのwebサーバの場合
dockerize -wait http://localhost:80 -timeout 1m
```


### 参考(dockerize)

https://unicorn.limited/jp/item/777
https://qiita.com/ta_ta_ta_miya/items/c7acb37991f01b5641a7


# PostgreSQLのCircleCI設定の例

```
version: 2
jobs:
  build:
    working_directory: ~/myapp
    docker:
      - image: circleci/ruby:2.3-node
        environment:
          PGHOST: 127.0.0.1
          PGUSER: myapp-test
          RAILS_ENV: test
      - image: circleci/postgres:9.5-alpine
        environment:
          POSTGRES_USER: myapp
          POSTGRES_DB: myapp-test
          POSTGRES_PASSWORD: ""
    steps:
      - checkout

      # 環境の整備??
      - restore_cache:
          name: Restore bundle cache
          keys:
            - myapp-bundle-{{ checksum "Gemfile.lock" }}
            - myapp-bundle-

      - restore_cache:
          name: Restore yarn cache
          keys:
            - myapp-yarn-{{ checksum "yarn.lock" }}
            - myapp-yarn-
      # 必要なモジュール・」パッケージのインストール
      - run:
          name: Bundle Install
          command: bin/bundle check --path vendor/bundle ||  bin/bundle install --path vendor/bundle --jobs 4 --retry 3

      - run:
          name: Yarn Install
          command: yarn install

      - save_cache:
          name: Store bundle cache
          key: myapp-bundle-{{ checksum "Gemfile.lock" }}
          paths:
            - vendor/bundle

      - save_cache:
          name: Store yarn cache
          key: myapp-yarn-{{ checksum "yarn.lock" }}
          paths:
            - ~/.yarn-cache
      # Rubocopによる静的解析
      - run:
          name: Rubocop
          command: bin/rubocop --rails
      # DB起動の待機
      - run:
          name: Wait for DB
          command: dockerize -wait tcp://localhost:5432 -timeout 1m

      # DBのスキーマロードとテスト
      - run:
          name: Database setup
          command: bin/rails db:schema:load --trace

      - run:
          name: Run tests
          command: bin/rails test
```

# Railsアプリケーション向けのstructure.sqlに対するCircleCI設定例

コンテナイメージに不足しているものをインストールするときは素直にrunで必要なものを
apt-getなりyumなりしましょう。

# Postgreイメージの最適化

circleci/postgresのDockerイメージはディスク上の永続化領域を利用しているので、
テストの際はデータをtmpfsにおいてあげると早くなったりする。

pre-builtなイメージに以下の記述を追加することを考慮してくらはい

```
Dockerfile:
FROM circleci/postgres
ENV PGDATA /dev/shm/pgdata/data
```



# ざっくりまとめ


- 複数イメージを立ち上げると単一のPodの中に取り込まれる
- コンテナの間のサービス起動の待ち合わせにはdockerizeを使う
    - `dockerize -wait <<待つ対象サービスのURL>> <<コマンド>>`




# 全体の参考資料

- 後で読む系
    - https://dzone.com/articles/continuous-delivery-with-kubernetes-docker-and-cir
        - k8sとDockerとCircleCIを使ったCDについての話題
    - https://medium.com/@sergeychikin/circleci-2-0-ecr-kops-kubernetes-gradle-53a9ad1d63f5
        - CircleCI2.0とECR, Kops, k8sを使った事例
