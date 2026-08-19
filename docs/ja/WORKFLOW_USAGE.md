# 再利用可能ワークフロー利用ガイド

このドキュメントでは、このリポジトリに格納されている共有の再利用可能ワークフローを、自プロジェクトのワークフローから呼び出す方法を説明します。

---

## `create-sync-branch-pr.yml` — 同期ブランチ PR の作成

**ソース:** [`.github/workflows/create-sync-branch-pr.yml`](../../.github/workflows/create-sync-branch-pr.yml)

### 概要

このワークフローは、長期管理ブランチ間（例: `main → develop`）の「同期」Pull Request 作成を自動化します。

呼び出されると、以下の処理を行います。

1. 呼び出し元ブランチから `sync/<ソースブランチ>-to-<ターゲットブランチ>` という名前の同期ブランチを作成または上書き更新します。
2. そのブランチから指定した `target_branch` へ PR を作成します。
3. 同期ブランチに対する PR が既に存在する場合は PR の新規作成をスキップします。
4. PR に `sync` および `automated` ラベルを付与します。

### 入力パラメータ

| 項目            | 型       | 必須 | 説明                                                                                                                               |
| --------------- | -------- | ---- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `target_branch` | `string` | ✅   | 同期 PR のマージ先ブランチ名。                                                                                                     |
| `app_id`        | `string` | 任意 | 同期ブランチの push と PR 作成用トークンを発行する GitHub App の App ID。`secrets.app_private_key` と併用する。指定時は `pr_token` より優先される。 |

### 必要なパーミッション

呼び出し元ジョブには以下のパーミッションが必要です。

| パーミッション  | レベル  |
| --------------- | ------- |
| `contents`      | `write` |
| `pull-requests` | `write` |

### Secrets

| 項目              | 必須 | 説明                                                                                                                                                                                                                                                                       |
| ----------------- | ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `app_private_key` | 任意 | `inputs.app_id` で指定した GitHub App の private key。`app_id` と併用し、`actions/create-github-app-token` で短命のインストールトークンを生成する。                                                                                                                     |
| `pr_token`        | 任意 | 同期ブランチの push と PR 作成に使用する静的トークン。`app_id`/`app_private_key` が指定されていない場合のみ使用し、いずれも未指定なら `secrets.GITHUB_TOKEN` にフォールバックする。`GITHUB_TOKEN` で作成した PR は作者が `github-actions[bot]` になり、リポジトリ/Organization の承認ポリシーによっては `pull_request` トリガーのワークフロー実行に毎回手動承認が必要になる。 |

> [!TIP]
> このワークフローが作成した PR で常に "Workflows will not run until approved by a user with write permissions" と表示される場合、原因はデフォルトの `GITHUB_TOKEN`（作者が `github-actions[bot]` になる）です。特に組織全体で使う場合は、共有の人間/bot アカウントに紐づく PAT よりも **GitHub App** を使うことを推奨します。
>
> ```yaml
> uses: lepusinc/.github/.github/workflows/create-sync-branch-pr.yml@main
> with:
>   target_branch: develop
>   app_id: ${{ vars.SYNC_PR_APP_ID }}
> secrets:
>   app_private_key: ${{ secrets.SYNC_PR_APP_PRIVATE_KEY }}
> ```
>
> Organization レベルで GitHub App を作成し、リポジトリ権限に `Contents: Read and write` と `Pull requests: Read and write` を付与して対象リポジトリにインストールしてください。App ID は機密情報ではないため Organization/リポジトリの **Variable**（`SYNC_PR_APP_ID`）に、private key は **Secret**（`SYNC_PR_APP_PRIVATE_KEY`）に保存します。どちらも Organization レベルで設定できるため、このワークフローを呼び出す全リポジトリが同じ App を共有でき、リポジトリごとの認証情報管理が不要になります。
>
> GitHub App がまだ用意できない場合は、`pr_token` によるフォールバックも利用できます（書き込み権限を持つコラボレーターアカウントの PAT を渡す）。
>
> ```yaml
> secrets:
>   pr_token: ${{ secrets.SYNC_PR_TOKEN }}
> ```

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
| 同期 PR の CI が毎回 "workflow awaiting approval" になる | PR がデフォルトの `GITHUB_TOKEN` で作成され、作者が `github-actions[bot]` になっているため、承認ポリシーの対象になっている。 | コラボレーターアカウントの PAT を `secrets.pr_token` として渡す（上記 Secrets の節を参照）。 |
