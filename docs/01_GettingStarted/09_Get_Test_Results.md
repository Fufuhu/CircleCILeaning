# テストのメタデータの収集

CircleCIではXMLファイルからテストのメタデータを収集する。
テストのメタデータ収集には、`store_test_restuls`ステップを利用する。

# フォーマッタの有効化

JUnitのxmlフォーマットでなければならない。
自動では収集されない。



# テストステップのメタデータ収集

```
- store_test_results:
    path: JUnitのxml形式のファイルパス
```

# テスト記述のサンプル

## ESLint

```
    steps:
      - run: 
          command: |
            mkdir -p ~/reports
            eslint ./src/ --format junit --output-file ~/reports/eslint.xml
          when: always
      - store_test_results:
          path: ~/reports
      - store_artifacts:
          path: ~/reports  
```

## go

テスト結果をJUnit形式でなんとかして出力させて
`store_test_results`すればOK



