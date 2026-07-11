# タスク: 実装計画に従ったコード生成

> 入力: `docs/plan/実装計画.md`、CLAUDE.md、`docs/development/`
> 出力: タスクIssue・実装ブランチ・PR（最大 `$MAX_TASKS` 件）

## 目的

`docs/plan/実装計画.md` に従い、未着手タスクを実装してPRを作成する。

## 前提（存在するものは必ず読む）

- CLAUDE.md
- docs/development/ 配下の開発ルール・ガイド
- docs/plan/実装計画.md

## 作業内容

1. `docs/plan/実装計画.md` を読み、ステータスが「未着手」のタスクを依存順に並べる。
2. まだ対応するIssueが存在しないタスクについて、タスクごとにGitHub Issueを作成する。
   - `gh label create claude-task --force` を先に実行してよい
   - タイトル: `[TASK-XXX] <タスク名>`
   - 本文: 実装計画の該当タスクの内容をそのまま転記し、末尾に「生成元: docs/plan/実装計画.md」と書く
   - ラベル: `claude-task`
   - 既存Issueとの重複を `gh issue list --search` で確認してから作成する
3. 依存順の先頭から **最大 $MAX_TASKS 件** のタスクを実装する。タスクごとに:
   - デフォルトブランチから `feature/issue-<Issue番号>-<slug>` ブランチを作成する
   - 実装計画・設計ドキュメントの範囲のみ実装する（範囲外のリファクタ禁止）
   - package.json 等からこのリポジトリで利用可能な lint / typecheck / test コマンドを
     検出して実行し、エラーがあれば修正する。
     ただし、変更がドキュメント（.md ファイル）のみの場合は省略してよい
   - Conventional Commits形式でコミットし、ブランチをpushする
   - PRを作成する（`gh pr create`）。.github/pull_request_template.md が存在する場合は
     その構成に沿い、変更内容・目的・確認内容・影響範囲を記載する
     - 「対応Issue」として `Closes #<Issue番号>` を記載する
   - `docs/plan/実装計画.md` の該当タスクのステータス更新は行わない
     （ステータスはPRマージ後に人間が更新する）
4. 実装しなかった残りのタスクについて、最後に一覧を出力する。
   - 残タスクのIssueは、人間がIssueコメントで `@claude` とメンションすることで
     claude-issue-to-pr ワークフローから個別に実行できる

## 禁止事項

- docs/ 配下の要件・設計ドキュメントの変更
- 実装計画にないタスクの実装
- タスク内容が曖昧・設計と矛盾する場合は、そのタスクをスキップして
  対応するIssueへ質問コメントを投稿する（推測で実装しない）
- デフォルトブランチへの直接push
