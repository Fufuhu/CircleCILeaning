

# CircleCI CLIのインストール

1. Dockerをインストールする
1. 下記のコマンドを実行する

```
curl -o /usr/local/bin/circleci https://circle-downloads.s3.amazonaws.com/releases/build_agent_wrapper/circleci && chmod +x /usr/local/bin/circleci
```

```
$ circleci
Receiving latest version of circleci...
The CLI tool to be used in CircleCI.

Usage:
  circleci [flags]
  circleci [command]

Available Commands:
  build       run a full build locally
  config      validate and update configuration files
  help        Help about any command
  step        execute steps
  tests       collect and split files with tests
  version     output version info

Flags:
  -c, --config string   config file (default is .circleci/config.yml)
  -h, --help            help for circleci
      --taskId string   TaskID
      --verbose         emit verbose logging output
```

ここまできたら一旦完了。