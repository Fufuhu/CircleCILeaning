

# config.ymlのバリデーション

```
circleci config validate -c .circleci/config.yml
```

# pre-commitフックを追加する

## ファイルの作成

`.git/hooks/pre-commit`ファイルが存在するとpush前に内容が実行される。

こんな感じの内容を書いておく。

```
#!/usr/bin/env bash

# The following line is needed by the CircleCI Local Build Tool (due to Docker interactivity)
exec < /dev/tty

# If validation fails, tell Git to stop and provide error message. Otherwise, continue.
if ! eMSG=$(circleci config validate -c .circleci/config.yml); then
    echo "CircleCI Configuration Failed Validation."
    echo $eMSG
    exit 1
fi
```

参考) https://circleci.com/blog/circleci-hacks-validate-circleci-config-on-every-commit-with-a-git-hook/
