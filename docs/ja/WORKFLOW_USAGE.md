# 再利用可能ワークフロー利用ガイド

このドキュメントでは、このリポジトリに格納されている共有の再利用可能ワークフローを、自プロジェクトのワークフローから呼び出す方法を説明します。

---

## `create-sync-branch-pr.yml` — 同期ブランチ PR の作成

**ソース:** [`.github/workflows/create-sync-branch-pr.yml`](../.github/workflows/create-sync-branch-pr.yml)

### 概要

このワークフローは、長期管理ブランチ間（例: `main → develop`）の「同期」Pull Request 作成を自動化します。

呼び出されると、以下の処理を行います。

1. 呼び出し元ブランチから `sync/<ソースブランチ>-to-<ターゲットブランチ>` という名前の同期ブランチを作成または上書き更新します。
2. そのブランチから指定した `target_branch` へ PR を作成します。
3. 同期ブランチに対する PR が既に存在する場合は PR の新規作成をスキップします。
4. PR に `sync` および `automated` ラベルを付与します。

### 入力パラメータ

| 項目            | 型       | 必須 | 説明                           |
| --------------- | -------- | ---- | ------------------------------ |
| `target_branch` | `string` | ✅   | 同期 PR のマージ先ブランチ名。 |

### 必要なパーミッション

呼び出し元ジョブには以下のパーミッションが必要です。

| パーミッション  | レベル  |
| --------------- | ------- |
| `contents`      | `write` |
| `pull-requests` | `write` |

### 同期ブランチの命名規則

同期ブランチは以下の命名規則で作成されます。

```text
sync/<ソースブランチ>-to-<ターゲットブランチ>
```

例: `main` ブランチから `target_branch: develop` を指定して呼び出した場合:

```text
sync/main-to-develop
```

---

### 使用例

`main` へのプッシュを契機に、`main → develop` の同期 PR を自動作成するワークフローの例です。

```yaml
# .github/workflows/sync-main-to-develop.yml
name: main を develop に同期

on:
  push:
    branches:
      - main

jobs:
  create-sync-pr:
    name: 同期 PR 作成 (main → develop)
    permissions:
      contents: write
      pull-requests: write
    uses: lepusinc/.github/.github/workflows/create-sync-branch-pr.yml@main
    with:
      target_branch: develop
```

> [!NOTE]
> `permissions` ブロックは、ワークフロー全体ではなく再利用可能ワークフローを呼び出す**ジョブ**に定義してください。

---

### `workflow_dispatch` トリガーからの呼び出し

手動実行で任意のターゲットブランチに同期 PR を作成したい場合は、`workflow_dispatch` の入力と組み合わせる方法が便利です。

```yaml
# .github/workflows/manual-sync.yml
name: 手動同期 PR

on:
  workflow_dispatch:
    inputs:
      target_branch:
        description: "同期先ブランチ名"
        type: string
        required: true

jobs:
  create-sync-pr:
    name: 同期 PR 作成
    permissions:
      contents: write
      pull-requests: write
    uses: lepusinc/.github/.github/workflows/create-sync-branch-pr.yml@main
    with:
      target_branch: ${{ inputs.target_branch }}
```

---

### 特定のリビジョンへのピン留め

安定性と監査可能性を高めるため、ブランチ名の代わりにコミット SHA またはタグを指定してワークフローをピン留めすることを推奨します。

```yaml
uses: lepusinc/.github/.github/workflows/create-sync-branch-pr.yml@<コミットSHA>
```

`uses` に指定できるフォーマットの詳細は [GitHub の再利用可能ワークフロードキュメント](https://docs.github.com/ja/actions/using-workflows/reusing-workflows) を参照してください。

---

### PR に付与されるラベル

ワークフローは作成済みまたは既存の PR に対して、以下のラベルを自動付与します。

| ラベル      | 説明                                           |
| ----------- | ---------------------------------------------- |
| `sync`      | ブランチ同期目的の PR であることを示す。       |
| `automated` | 自動化によって作成された PR であることを示す。 |

> [!WARNING]
> ターゲットリポジトリに `sync` または `automated` ラベルが存在しない場合、ラベル付与ステップはジョブサマリーに警告を出力しますが、ワークフロー全体は失敗しません。

---

### トラブルシューティング

| 症状                                                | 原因                                                             | 対処                                                                                 |
| --------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| "Resource not accessible by integration" で失敗する | `contents: write` または `pull-requests: write` が不足している。 | 呼び出し元ジョブに `permissions` ブロックを追加する（上記例を参照）。                |
| エラーなし・PR も作成されない                       | 同期ブランチに対する PR が既に存在する。                         | ヘッドブランチ `sync/<source>-to-<target>` のオープン PR を確認する。                |
| 同期ブランチへの force-push が失敗する              | ブランチ取得後に別の主体がプッシュした競合。                     | ワークフローを再実行する。解消しない場合は同期ブランチを手動削除してから再実行する。 |
