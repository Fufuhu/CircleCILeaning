
ジョブは複数のステップから構成される。


```
steps:
  run: 
    run配下に記述可能なkey: 値
  run:
    run配下に記述可能なkey: 値
```

# runステップ配下に記述可能なkey

|key|必須?|型|内容|
|---|---|---|---|
|command|Y|String|シェル経由で実行するコマンド|
|name|N|String|CircleCIのUI上で表示するステップのタイトル|
|shell|N|String|コマンドを実行するためのシェルを指定する|
|environment|N|String|環境変数を指定する|
|background|N|Boolean|このステップがバックグラウンドで実行されるべきか|
|working_directory|N|String|ステップを実行するディレクトリを指定する。(default: jobのworking_directory|
|no_output_timeout|N|String|出力なしに実行可能なコマンドの経過時間(default: 10分)|
|when|N|String|ステップの有効または無効化の条件(always, on_success, on_fail, default:on_success)|

runステップごとに新しいシェルが起動するので注意してね。

## shellの指定例

```
- run:
    shell: /bin/sh
    command: |
      echo Running test
      mkdir -p /tmp/test-results
      make test
``` 

## environmentの指定例

```
      - run:
          name: Run unit tests
          environment:
            CONTACTS_DB_URL: "postgres://root@localhost:5432/circle_test?sslmode=disable"
            CONTACTS_DB_MIGRATIONS: /go/src/github.com/CircleCI-Public/circleci-demo-go/db/migrations
```

## backgroundについて

バックグラウンド指定された場合は即座に次のステップに処理が移行する。

```
- run:
    name: Running X virtual framebuffer
    command: Xvfb :99 -screen 0 1280x1024x24
    background: true

- run: make test
```

この場合はXを立ち上げてテストを実行している。

## whenについて

- on_success
    - exit codeが0
- on_fail
    - exit codeが0以外

# checkoutステップ

gitリポジトリからコードをチェックアウトする

CircleCIはサブモジュールをチェックアウトしないので、こんな感じにしないといけない

```
- checkout
- run: git submodule sync
- run: git submodule update --init
```

# save_cache/restore_cacheステップ

```
- save_cache:
    key: v1-myapp-{{ arch }}-{{ checksum "project.clj" }} # キャッシュのキー
    paths: # キャッシュに含められるべきディレクトリ
      - /home/ubuntu/.m2
```

```
- restore_cache:
    keys:
      - v1-myapp-{{ arch }}-{{ checksum "project.clj" }}
      # if cache for exact version of `project.clj` is not present then load any most recent one
      - v1-myapp-
``` 
