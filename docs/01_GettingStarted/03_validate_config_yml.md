

# config.ymlのバリデーション

```
circleci config validate -c .circleci/config.yml
```

# pre-commitフックを追加する

## ファイルの作成

`.git/hooks/pre-push`ファイルが存在するとpush前に内容が実行される。

こんな感じの内容を書いておく。

```
CIRCLE_CI_COMMAND=`which circleci`↲
${CIRCLE_CI_COMMAND} config validate↲
``` 
