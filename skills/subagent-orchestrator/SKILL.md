---
name: subagent-orchestrator
description: 複数サブエージェントで大規模開発をオーケストレーションする。ユーザーがアプリ構想のみを提示し、実装を一貫して任せる場合に使用する。
---

# Subagent Orchestrator

## 目的

曖昧な構想から実装完了までを、再現可能なオーケストレーション手順で進める。
判断・進捗・引継ぎ情報を `orchestration/` 配下に一元化する。

## メインエージェント運用ルール

- メインエージェントは進捗管理と調整に専念する。
- メインエージェントは実装作業とレビュー作業を原則行わない。
- 実装とレビューは必ずサブエージェントへ委任する。
- サブエージェント待機中は途中介入せず、完了応答まで待機を継続する。
- 待機開始から1時間を超えて完了しない場合のみ介入または強制終了を行う。
- 介入または強制終了時は理由と次アクションを `orchestration/tasks/<task-id>/task.md` の `Coordinator Notes` に記録する。

## 実行手順

1. オーケストレーション文書を初期化する。
   - `bun run scripts/init_orchestration.ts [repo-root]`
   - `orchestration/README.md` を確認して各ファイルの役割を把握する。
2. 目的と制約を明文化する。
   - 目標・成功条件・制約・未解決事項を `orchestration/charter.md` に記載する。
3. 実装前に要件を深掘りする。
   - 不明点を解消する。
   - 目標を小さく検証可能なタスクへ分解し、`orchestration/task-breakdown.md` に記録する。
4. タスクをファイル単位で定義する。
   - `orchestration/tasks/<task-id>/` を作成する。
   - メインエージェント専用の `task.md` と、サブエージェント専用の `subagent-output.md` を分離する。
   - `task.md` の frontmatter に `id`, `summary`, `status`, `deps`, `branch` を記述する。
   - 推奨形式:

```markdown
---
id: T01
summary: API仕様を追加
status: todo
deps: [T00]
branch: feat/add-api-spec
---
```

5. 依存関係をDAGとして確定する。
   - `bun run scripts/integration_order.ts --tasks-dir orchestration/tasks --write orchestration/dependency-dag.md`
   - `done` タスクはDAG表示から除外し、今着手可能なタスクを `orchestration/ready-now.md` に出力する。
6. タスクごとに役割を分離する。
   - 実装担当とレビュー担当を分離する。
   - 割当は必要なら `orchestration/task-breakdown.md` に記録する。
   - 契約は `references/subagent-contract.md` に従う。
7. 並列タスク用に独立作業環境を作成する。
   - 1タスクの実装/レビューは同じブランチを使う。
   - 1タスクに対して1つのworktreeを作る。
   - ブランチ名は `<scope>/<summary-kebab>` 形式にする。
   - `scope` は `feat|fix|docs|style|refactor|test|chore|build|ci|perf|revert` から選ぶ。
   - `summary-kebab` はタスク内容を非常に簡潔なケバブケースで記述する。
   - 必要な場合のみ次を実行する。
     - 新規ブランチ作成込み: `git worktree add -b <branch> .worktrees/<task-id> HEAD`
     - 既存ブランチ利用: `git worktree add .worktrees/<task-id> <branch>`
8. サブエージェントを明確な責務で実行する。
   - 各サブエージェントへタスク、DoD、成果物パス、必要文書のみを渡す。
   - ステータス更新先を `orchestration/tasks/<task-id>/subagent-output.md` に固定する。
   - 実行中は途中介入せず、完了応答まで待機を継続する。
   - 待機開始から1時間超過時のみ介入または強制終了する。
9. コミット規約を強制する。
   - conventional commits のみ許可する。
   - 件名は `type: 日本語summary` 形式で、scopeは使わない。
   - 2行目は空行、3行目以降に変更詳細を記載する。
   - 必要に応じて `bun run scripts/cc_commit_check.ts` で検証する。
10. 独立レビューを実施する。
    - 実装担当と別の担当でレビューする。
    - 結果を `orchestration/review-log.md` に記録する。
11. 依存順で統合する。
    - DAG順でマージする。
    - 競合解消と検証結果を `orchestration/integration-log.md` に記録する。
12. 引継ぎを完了する。
    - 完了範囲、残課題、次アクションを `orchestration/handover.md` に確定する。

## 管理ファイル

- `orchestration/README.md`: orchestration配下ファイルの役割と更新責務。
- `orchestration/charter.md`: 目的、成功条件、制約、意思決定履歴。
- `orchestration/task-breakdown.md`: タスク一覧、依存、担当、DoD、状態。
- `orchestration/tasks/<task-id>/task.md`: メインエージェント管理のタスク定義（frontmatter、要件、受け入れ条件、調整メモ）。
- `orchestration/tasks/<task-id>/subagent-output.md`: サブエージェント成果（実施レポート、PR説明文ドラフト、残課題）。
- `orchestration/dependency-dag.md`: 依存グラフ、並列バッチ、統合順。
- `orchestration/ready-now.md`: 今すぐ着手可能な `todo` タスク一覧。
- `orchestration/review-log.md`: レビュー指摘、判定、対応状況。
- `orchestration/integration-log.md`: 統合記録、競合対応、検証結果。
- `orchestration/handover.md`: 最終引継ぎ情報。

## サブエージェント指示テンプレート

実装担当テンプレート:

```text
あなたはサブエージェントです。TASK_ID=<id> だけを担当してください。
参照: orchestration/charter.md, orchestration/task-breakdown.md, orchestration/tasks/<id>/task.md, orchestration/tasks/<id>/subagent-output.md, orchestration/dependency-dag.md
作業場所: <assigned worktree path> のみ
入力:
- task_id: <id>
- goal: <このタスクで達成すること>
- in_scope: <実施対象>
- out_of_scope: <今回やらないこと>
- dependencies: <依存タスクID。なければ []>
- worktree_path: <assigned worktree path>
- expected_outputs: <期待成果物の一覧>
- definition_of_done: <完了条件>
- required_docs_to_update: orchestration/tasks/<id>/subagent-output.md
成果物:
- TASK_IDに関するコード変更
- TASK_IDに関するテスト
- orchestration/tasks/<id>/subagent-output.md の更新
コミットは conventional commits を使ってください。
件名は `type: 日本語summary` 形式にし、scopeは使わないでください。
2行目は空行、3行目以降に変更詳細を書いてください。
無関係なファイルは変更しないでください。
```

レビュー担当テンプレート:

```text
あなたはサブエージェントです。TASK_ID=<id> のレビューのみ担当してください。
参照: orchestration/charter.md, orchestration/task-breakdown.md, orchestration/tasks/<id>/task.md, orchestration/tasks/<id>/subagent-output.md, orchestration/review-log.md
実装担当と同じブランチ/worktreeを確認してください。
入力:
- task_id: <id>
- goal: <レビューで検証する目的>
- in_scope: <レビュー対象>
- out_of_scope: <レビュー対象外>
- dependencies: <依存タスクID。なければ []>
- worktree_path: <assigned worktree path>
- expected_outputs: review-log更新と判定
- definition_of_done: <判定と指摘が記録されている状態>
- required_docs_to_update: orchestration/review-log.md, orchestration/tasks/<id>/subagent-output.md
挙動の回帰、テスト不足、契約不一致を重点的に確認してください。
指摘と判定を orchestration/review-log.md に記録してください。
```

## 追加参照の読み分け

- タスク分解基準が必要な場合: `references/task-decomposition.md`
- DAG設計と並列化ルールが必要な場合: `references/dependency-model.md`
- worktree運用手順が必要な場合: `references/worktree-playbook.md`
- サブエージェント契約が必要な場合: `references/subagent-contract.md`
- 役割分離の責務整理が必要な場合: `references/role-separation.md`
- conventional commits の詳細が必要な場合: `references/conventional-commits.md`
- 統合と競合解消方針が必要な場合: `references/merge-integration.md`
- ドキュメント更新規律が必要な場合: `references/doc-operations.md`
