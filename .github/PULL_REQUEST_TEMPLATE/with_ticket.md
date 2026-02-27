<!-- If AI generates this PR content, include both Japanese and English. -->
<!-- Use the reviewee's language for PR review comments. If unknown, write both Japanese and English. -->
<!-- PRでのレビューコメントはレビュイーの言語でコメントする。言語が分からなければ、日本語と英語の併記でコメントする。 -->
<!-- Write the PR body based on verified facts; do not fill unknowns with guesses. -->
<!-- PR本文は確認済みの事実ベースで記載し、未確認事項を推測で埋めない。 -->
<!-- Check only applicable checklist items; add a short reason for non-applicable items when needed. -->
<!-- チェックリストは該当項目のみチェックし、必要に応じて非該当理由を簡潔に記載する。 -->
<!-- For verification, prioritize reproducible evidence (commands, environment, links). -->
<!-- 動作確認は再現可能な証跡（実行コマンド、環境、リンク）を優先して記載する。 -->
<!-- Include both rationale and decision reason in 1-2 lines. -->
<!-- 変更理由と判断理由を1-2行で明記する。 -->
<!-- Explicitly state security/permission/data impact; write "none" when there is no impact. -->
<!-- セキュリティ/権限/データ影響は明示し、影響がない場合も「なし」と記載する。 -->
<!-- For review feedback responses, separate addressed items and intentionally unaddressed items with reasons. -->
<!-- レビュー指摘への対応は、対応済みと未対応（理由付き）を分けて記載する。 -->
<!-- Do not delete comment lines in this template. -->
<!-- このテンプレートのコメント行は削除しない。 -->
<!-- Do not modify or remove existing option texts. -->
<!-- テンプレート内の既存選択肢テキストの変更および選択肢の削除はしない。 -->
<!-- Toggle checklist states (on/off) as appropriate. -->
<!-- チェックの on / off は適切に行う。 -->
<!-- Adding options is allowed; append "（追加）" to added option text so additions are identifiable. -->
<!-- 選択肢の追加は許可するが、追加したことが分かるよう選択肢のテキストに「（追加）」サフィックスを付与する。 -->

# Pull Request (With Ticket)

## 📝 Ticket / 対象チケット

> Prefix the PR title with the ticket number/key (required)
>
> PRタイトルの先頭にチケット番号/キーを付けること（必須）

<https://lepus.atlassian.net/browse/PRJ-123>

---

## 🔄 Changes / 変更内容

> List the main changes in bullet points
>
> 主な変更点を箇条書きで

- 変更点1
- 変更点2
- ...

---

## 🖥️ Verification / 動作確認

> Evidence of developer-led verification in the development environment.
>
> 開発環境で開発者が行った動作確認。

- [ ] Include screenshots or videos in the ticket's "Implementation Results" section  
  チケットに実装結果のスクリーンショット or 動画を記載
- [ ] Unit test added → Execution results (screenshot/CI link)  
  単体テスト追加 → 実行結果 (スクショ/CIリンク)
- [ ] Integration/E2E test added → Execution results (screenshot/CI link)  
  統合/E2Eテスト追加 → 実行結果 (スクショ/CIリンク)
- [ ] Log file snippets  
  ログファイルの抜粋
- [ ] API request & response samples  
  APIリクエスト&レスポンスのサンプル
- [ ] Database records (before & after)  
  DBレコード(変更前後)
- [ ] Performance metrics  
  パフォーマンスメトリクス
- [ ] REPL/Debugger output  
  REPL/デバッガーの実行結果

---

## 📋 Nature of this PR / このPRの性質

> Select all applicable categories
>
> 該当するカテゴリをすべて選択

<!-- prettier-ignore -->
- [ ] Feature / Enhancement  
  機能追加 / 改善
- [ ] Bug Fix  
  バグ修正
- [ ] Refactoring (no behavior change)  
  リファクタリング（振る舞い変更なし）
- [ ] Performance / Reliability Improvement  
  パフォーマンス / 信頼性改善
- [ ] Test-only Change  
  テストのみの変更
- [ ] Build / Deployment / CI / Tooling / Dependency Update  
  ビルド / デプロイ / CI / ツール / 依存関係の更新

---

## 🔍 Checklist / チェックリスト

<!-- prettier-ignore -->
- [ ] CI is green  
  CIがグリーンである
- [ ] PR title is prefixed with the ticket number/key  
  PRタイトルの先頭にチケット番号/キーを付けた
- [ ] Verification by developer is documented  
  開発者による動作確認を記載した
- [ ] Implementation results are documented in the ticket  
  実装結果をチケットに記載した
- [ ] Verification steps are documented in the ticket  
  動作確認手順をチケットに記載した
- [ ] Security/permissions/performance impact considered  
  セキュリティ/権限/パフォーマンス影響を考慮した
- [ ] System impact is documented in the ticket  
  システム影響範囲をチケットで選択した
- [ ] Known constraints/TODOs that should be communicated to reviewers are documented  
  レビュアーに伝えるべき既知の制約・TODOを記載した

---

> 🛈 For detailed review criteria, please refer to [REVIEW_POLICY.md](https://github.com/lepusinc/development-guidelines/blob/main/docs/en/REVIEW_POLICY.md)  
> 🛈 詳細なレビュー基準については [REVIEW_POLICY.md](https://github.com/lepusinc/development-guidelines/blob/main/docs/ja/REVIEW_POLICY.md) を参照してください
