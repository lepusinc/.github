# Markdown Alert Problem Matcher

GitHub Actions ワークフローで Markdown alert形式のメッセージを GitHub Annotations に自動変換するための Problem Matcher です。

## 対応フォーマット

```markdown
> [!NOTE]
> メッセージ内容...

> [!WARNING]
> 警告メッセージ...

> [!CAUTION]
> または [!CRITICAL]
> エラーメッセージ...
```

これらは自動的に以下のアノテーションに変換されます：

```text
::notice::メッセージ内容...
::warning::警告メッセージ...
::error::エラーメッセージ...
```

## ワークフローでの使用方法

### 1. Problem Matcher を登録

ワークフロー内で以下を実行してマッチャーを登録します：

```yaml
- name: Add problem matcher for markdown alerts
  run: echo "::add-matcher::.github/matchers/markdown-alerts.json"
```

### 2. Markdown alert形式で出力

その後、ログに Markdown alert形式でメッセージを出力します：

```yaml
- name: Check something
  run: |
    if [ "$some_condition" ]; then
      cat <<EOF
    > [!WARNING]
    > This is a warning message
    > that spans multiple lines
    EOF
    fi
```

## 使用例

### 複数行メッセージ

```bash
echo "::add-matcher::.github/matchers/markdown-alerts.json"

# 以下のメッセージは自動的に JSON 問題マッチャーに拾われます
cat <<'EOF'
> [!WARNING]
> Failed to deploy to production
> Please check the deployment logs
EOF
```

### シェルスクリプト内での使用

```bash
#!/bin/bash

echo "::add-matcher::.github/matchers/markdown-alerts.json"

check_dependency() {
  if ! command -v "$1" &> /dev/null; then
    cat <<EOF
> [!CAUTION]
> Missing required dependency: $1
> Please install it before proceeding
EOF
    return 1
  fi
}
```

## ワークフロー例

```yaml
name: CI

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6

      - name: Add problem matcher
        run: echo "::add-matcher::.github/matchers/markdown-alerts.json"

      - name: Run tests
        run: |
          if [ "$CONDITION" ]; then
            cat <<EOF
          > [!WARNING]
          > Some tests were skipped
          EOF
          fi
```

## トラブルシューティング

### メッセージが拾われない場合

1. マッチャーが登録されているか確認：`::add-matcher::` コマンドを実行しましたか？
2. フォーマットが正確か確認：`> [!WARNING]` の形式を確認してください
3. ログが標準出力に出力されているか確認：`echo` または `cat` で出力してください

### 複数行メッセージが正しく処理されない場合

- 各行が `>` と半角スペースで始まっていることを確認してください
- 空行を含む場合は `>` の後ろに半角スペースだけを置いた行を挿入してください

## 参考

- [GitHub Actions: Problem Matchers](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/logging-for-workflows#adding-a-problem-matcher)
- [Markdown Alert Syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#alerts)
