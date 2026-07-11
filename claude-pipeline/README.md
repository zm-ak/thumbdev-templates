# Claude Pipeline テンプレート

ThumbDevアプリが**ユーザーのリポジトリへプッシュして導入する**、AI駆動開発パイプラインのテンプレート一式。
このフォルダの内容はthumbdevリポジトリ自身では動作しない（アプリの資産として保持している）。

## 提供する体験

ユーザーがアプリで登録した1次ドキュメント（要件・設計・メモ）を起点に、
ユーザーのリポジトリ内で以下が自動で進む。

```text
アプリで1次ドキュメントを登録（__DOCS_PATH__/ へ保存）
↓
ユーザー: 作業ブランチ → デフォルトブランチへのPRをマージ  ←【起点】
↓
[自動] claude-docs-pipeline.yml
  1. ドキュメントフォルダ構成整理      (claude-sonnet-5)
  2. CLAUDE.mdの整理（無ければ生成）  (claude-sonnet-5)
  3. プロジェクトルールの整理         (claude-sonnet-5)
  4. リポジトリ説明の生成            (claude-haiku-4-5)
  5. 実装計画の生成                  (claude-opus-4-8)
↓
ドキュメント整理・実装計画のPRが作成される
↓
ユーザー: レビュー・マージ  ←【承認】
↓
[自動] claude-plan-to-code.yml
  - 実装計画のタスクをIssue化
  - 依存順に実装し、タスクごとにPRを作成 (claude-opus-4-8)
↓
残タスクはIssueへ「@claude」コメントで個別実行 (claude-issue-to-pr.yml)
```

## 配置規約（ミラー方式）

アプリは `claude-pipeline/` プレフィックスを除いた相対パスへそのままコミットする。
テンプレート内のパス = ユーザーリポジトリでの配置先。

| テンプレート内のパス | ユーザーリポジトリでの配置先 |
| --- | --- |
| `.github/workflows/*.yml` | `.github/workflows/*.yml` |
| `.github/claude/prompts/*.md` | `.github/claude/prompts/*.md` |
| `README.md` | （配布対象外・導入時に除外） |

例: `claude-pipeline/.github/workflows/claude-docs-pipeline.yml`
→ ユーザーリポジトリの `.github/workflows/claude-docs-pipeline.yml`

パス変換テーブルはアプリ側に持たない。テンプレート改善時にアプリの再ビルドは不要。

## プレースホルダ

アプリは配置時に、**全ファイル**に対して以下の置換を行う。

| プレースホルダ | 置換内容 | 例（デフォルト） |
| --- | --- | --- |
| `__DOCS_PATH__` | アプリの保存フォルダ設定（末尾スラッシュなし） | `docs/thumbdev` |

## ユーザーリポジトリ側で必要な設定

テンプレート配置だけでは動作しない。ユーザーに以下を案内する必要がある。

| 設定 | 必須 | 内容 |
| --- | --- | --- |
| `ANTHROPIC_API_KEY`（Actions Secret） | 必須 | Claude実行用のAPIキー |
| `claude-task` ラベル | 必須 | Issue自動実装（issue-to-pr / plan-to-code）用。未作成なら `gh label create claude-task` で作成可 |
| `REPO_ADMIN_TOKEN`（Actions Secret） | 任意 | リポジトリ説明（About欄）の自動反映用。repo権限のPAT。未設定時は手動反映の警告を出す |
| GitHub Actionsの有効化 | 必須 | プライベートリポジトリ等で無効化されていないこと |

## タスクとモデルの対応

| タスク | モデル | 理由 |
| --- | --- | --- |
| ドキュメント整理 / CLAUDE.md整理 / ルール整理 | claude-sonnet-5 | 分類・整合性確認。バランス重視 |
| リポジトリ説明の生成 | claude-haiku-4-5 | 要約中心の軽量タスク。速度・コスト優先 |
| 実装計画の生成 | claude-opus-4-8 | 設計理解と計画立案。最も推論力が必要 |
| コード生成（plan-to-code / issue-to-pr） | claude-opus-4-8 | 品質最優先 |

## 安全設計

- 各ステップの成果物は必ずPRになり、**マージ（人間の承認）なしに次の段階へ進まない**
- コード生成は1回の実行で最大3タスク（`MAX_TASKS`）に制限
- `claude/` で始まるブランチのマージでは docs pipeline は再起動しない（ループ防止）
- 自動生成Issueは GITHUB_TOKEN 起点のため issue-to-pr を自動起動しない（GitHub Actionsの仕様）。
  残タスクの実行はユーザーの `@claude` コメントによる明示的な指示が必要
- `@claude test` / `@claude security` / `@claude verify` は Pipeline Pro 強化パック用のコマンドであり、
  本ワークフロー（実装）では起動しない（Pro 同梱の `claude-pipeline-pro.yml` が処理する）

## テンプレートの意図的な簡素化

本テンプレートは任意のユーザーリポジトリ向けの汎用版である。
特定プロジェクト固有の高度な運用（例: `docs-task` ラベル連鎖、サブIssue分割、固定の npm コマンド）は含めない。
スタック固有のルールは、docs pipeline が1次ドキュメントから生成・整理する CLAUDE.md / `docs/development/` 経由で適用される。

## テンプレート更新時の注意

- このフォルダはユーザーリポジトリへ配置される成果物である。
  thumbdevリポジトリ固有のパス・ルール・ツール（docs/development/ 配下のガイド、npm前提など）を書かないこと
- 配置規約（ミラー方式）を崩さないこと。配布ファイルは必ず配置先と同じ相対パスで保持する
- プレースホルダを増やす場合は、この README とアプリ側の置換処理の両方を更新すること
