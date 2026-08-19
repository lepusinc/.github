# Reusable Workflow Usage Guide

This document explains how to call shared reusable workflows hosted in this repository from your own workflows.

---

## `create-sync-branch-pr.yml` — Create a Sync Branch PR

**Source:** [`.github/workflows/create-sync-branch-pr.yml`](../../.github/workflows/create-sync-branch-pr.yml)

### Overview

This workflow automates the creation of a "sync" Pull Request between two long-lived branches (e.g., `main → develop`).

When called, the workflow will:

1. Create or reset a sync branch named `sync/<source-branch>-to-<target-branch>` from the calling branch.
2. Open a PR from that sync branch into the specified `target_branch`.
3. Skip PR creation if a PR for the sync branch already exists.
4. Add the labels `sync` and `automated` to the PR.

### Inputs

| Name            | Type     | Required | Description                                                                                                                    |
| --------------- | -------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `target_branch` | `string` | ✅       | The branch to merge the sync PR into.                                                                                          |
| `app_id`        | `string` | No       | App ID of a GitHub App used to mint an installation token for the sync branch push and PR creation. Used with `secrets.app_private_key`. Takes priority over `pr_token` when set. |

### Required Permissions

The calling job must have the following permissions:

| Permission      | Level   |
| --------------- | ------- |
| `contents`      | `write` |
| `pull-requests` | `write` |

### Secrets

| Name              | Required | Description                                                                                                                                                                                                                                                                                                                                              |
| ----------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `app_private_key` | No       | Private key of the GitHub App identified by `inputs.app_id`. Used together with `app_id` to generate a short-lived installation token via `actions/create-github-app-token`.                                                                                                                                                                          |
| `pr_token`        | No       | Static token used to push the sync branch and create the PR, used only when `app_id`/`app_private_key` are not provided. Falls back to `secrets.GITHUB_TOKEN` when none of the above are set. A PR created with `GITHUB_TOKEN` is authored by `github-actions[bot]`, and depending on the repository/organization's approval policy, `pull_request`-triggered workflow runs on it may require manual approval every time. |

> [!TIP]
> If your PRs from this workflow keep showing "Workflows will not run until approved by a user with write permissions", the default `GITHUB_TOKEN` is the cause — its PRs are authored by `github-actions[bot]`. The recommended fix, especially for org-wide use, is a **GitHub App** (rather than a PAT tied to a shared human/bot account):
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
> Create a GitHub App at the organization level with `Contents: Read and write` and `Pull requests: Read and write` repository permissions, install it on the repositories that need it, and store its App ID as an org/repo **variable** (`SYNC_PR_APP_ID`, not sensitive) and its private key as an org/repo **secret** (`SYNC_PR_APP_PRIVATE_KEY`). Because both can be set at the organization level, every repository that calls this workflow can share the same App without per-repo credential management.
>
> If a GitHub App isn't available yet, `pr_token` still works as a fallback — pass a PAT from a collaborator account with write access:
>
> ```yaml
> secrets:
>   pr_token: ${{ secrets.SYNC_PR_TOKEN }}
> ```

### Sync Branch Naming Convention

The sync branch is created with the following name:

```text
sync/<source-branch>-to-<target-branch>
```

For example, calling from the `main` branch with `target_branch: develop` produces:

```text
sync/main-to-develop
```

---

### Usage Example

The following example shows a workflow that automatically creates a sync PR from `main` to `develop` whenever a push is made to `main`.

```yaml
# .github/workflows/sync-main-to-develop.yml
name: Sync main to develop

on:
  push:
    branches:
      - main

jobs:
  create-sync-pr:
    name: Create sync PR (main → develop)
    permissions:
      contents: write
      pull-requests: write
    uses: lepusinc/.github/.github/workflows/create-sync-branch-pr.yml@main
    with:
      target_branch: develop
```

> [!NOTE]
> The `permissions` block must be defined on the **job** that calls the reusable workflow, not at the workflow level, to grant the necessary write access.

---

### Calling from a `workflow_dispatch` Trigger

You can also expose `target_branch` as a manual input to allow on-demand sync PRs:

```yaml
# .github/workflows/manual-sync.yml
name: Manual sync PR

on:
  workflow_dispatch:
    inputs:
      target_branch:
        description: "Branch to sync into"
        type: string
        required: true

jobs:
  create-sync-pr:
    name: Create sync PR
    permissions:
      contents: write
      pull-requests: write
    uses: lepusinc/.github/.github/workflows/create-sync-branch-pr.yml@main
    with:
      target_branch: ${{ inputs.target_branch }}
```

---

### Pinning to a Specific Ref

To improve stability and auditability, pin the workflow reference to a specific commit SHA or tag instead of a branch name:

```yaml
uses: lepusinc/.github/.github/workflows/create-sync-branch-pr.yml@<commit-sha>
```

Refer to [GitHub's documentation on reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) for the full list of supported `uses` ref formats.

---

### Labels Applied to the PR

The workflow automatically applies the following labels to the created (or pre-existing) PR:

| Label       | Description                                  |
| ----------- | -------------------------------------------- |
| `sync`      | Marks the PR as a branch synchronization PR. |
| `automated` | Marks the PR as created by automation.       |

> [!WARNING]
> If the labels `sync` or `automated` do not exist in the target repository, the label-add step will report a warning in the job summary but will not fail the workflow.

---

### Troubleshooting

| Symptom                                                      | Cause                                                                 | Resolution                                                                              |
| ------------------------------------------------------------ | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Workflow fails with "Resource not accessible by integration" | Missing `contents: write` or `pull-requests: write` permission.       | Add the `permissions` block to the calling job (see example above).                     |
| No PR is created but no error is shown                       | A PR for the sync branch already exists.                              | Check for an open PR with the head branch `sync/<source>-to-<target>`.                  |
| Force-push to sync branch fails                              | Another actor pushed to the sync branch after the branch was fetched. | Re-run the workflow. If the issue persists, delete the sync branch manually and re-run. |
| CI on the sync PR always shows "workflow awaiting approval"  | The PR was created with the default `GITHUB_TOKEN`, so it is authored by `github-actions[bot]`, which some approval policies always gate. | Pass `secrets.pr_token` from a collaborator account's PAT (see the Secrets section above). |
