
# 手順概要

1. .circleciディレクトリをgitリポジトリのルートに配置する。
1. config.ymlファイルを.circleciディレクトリ内に作成する。
1. confi.gymlファイルをcommit & pushし、2.0を作成する。
1. Projectsページに行ってプロジェクトに上記のgitリポジトリを追加する。
1. ビルドして結果をみる(多分Greenなはず)

```
version: 2
jobs:
   build:
     docker:
       - image: circleci/golang:1.8
     steps:
       - checkout
       - run: echo "hello world"
```

## config.ymlの取り扱い

pre-commith hookを用いてconfig.ymlをテストすると良い。
gitにpushするときに `circleci config validate` コマンドを実行する。

参考) https://circleci.com/docs/2.0/local-jobs/