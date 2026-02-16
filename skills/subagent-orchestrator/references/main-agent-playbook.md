# メインエージェント実行プレイブック

## 運用ルール

- 進捗管理と調整に専念する。
- 実装作業とレビュー作業は原則行わない。
- 実装とレビューは必ずサブエージェントへ委任する。
- サブエージェント待機中は途中介入しない。
- 完了応答まで待機を継続する。
- 待機開始から1時間を超えて完了しない場合のみ介入または強制終了する。
- 介入または強制終了時は理由と次アクションを `orchestration/tasks/<task-id>/task.md` の `Coordinator Notes` に記録する。

## 実行手順

1. オーケストレーション文書を初期化する。
   - `bun run scripts/init_orchestration.ts [repo-root]`
   - `orchestration/README.md` を確認し、更新責務を把握する。
2. 目的と制約を明文化する。
   - 目標、成功条件、制約、未解決事項を `orchestration/charter.md` に記載する。
3. 実装前に要件を深掘りする。
   - 不明点を解消する。
   - 目標を小さく検証可能なタスクへ分解し、`orchestration/task-breakdown.md` に記録する。
4. タスクを定義する。
   - `orchestration/tasks/<task-id>/` を作成する。
   - `task.md` と `subagent-output.md` を分離する。
   - `task.md` の frontmatter に `id`, `summary`, `status`, `deps`, `branch` を記述する。
5. 着手可能タスクを確定する。
   - `bun run scripts/integration_order.ts --tasks-dir orchestration/tasks --ready-write orchestration/ready-now.md`
   - `done` タスクを前提として依存を評価し、今着手可能な `todo` を `orchestration/ready-now.md` に出力する。
   - `dependency-dag` の存在は前提にせず、`tasks/` 配下の `task.md` から依存を評価する。
6. 実装中の再計画を許容する。
   - 追加タスク、実装順の変更、不要タスクの削除を許容する。
   - 変更理由を `orchestration/task-breakdown.md` または各 `task.md` に記録する。
7. 役割を分離する。
   - タスクごとに実装担当とレビュー担当を分離する。
   - 契約は `references/subagent-contract.md` に従う。
8. 必要時のみ独立作業環境を作成する。
   - 1タスクの実装/レビューは同じブランチを使う。
   - 1タスクに対して1つのworktreeを作成する。
   - 新規ブランチ込み: `git worktree add -b <branch> .worktrees/<task-id> HEAD`
   - 既存ブランチ利用: `git worktree add .worktrees/<task-id> <branch>`
9. サブエージェントへ委任する。
   - `TASK_ID` と `worktree_path` を渡す。
   - `orchestration/tasks/<task-id>/` 配下を確認して作業するよう指示する。
   - 委任時は `subagent-orchestrator` スキルを必ず利用し、実装担当またはレビュー担当の手順に従うよう指示する。
10. コミット規約を強制する。
    - conventional commits のみ許可する。
    - 必要に応じて `bun run scripts/cc_commit_check.ts` で検証する。
11. 独立レビューを実施する。
    - 実装担当と別担当でレビューする。
    - 結果を `orchestration/tasks/<task-id>/review.md` に記録する。
12. 依存順で統合する。
    - `deps` を満たす順で統合する。
    - 競合解消と検証結果を `orchestration/integration-log.md` に記録する。
13. 引継ぎを完了する。
    - 完了範囲、残課題、次アクションを `orchestration/handover.md` に確定する。

## サブエージェント委任テンプレート（簡潔版）

実装担当:

```text
あなたはサブエージェントです。TASK_ID=<id> の実装担当です。
`subagent-orchestrator` スキルを必ず利用し、実装担当の手順に従ってください。
`orchestration/tasks/<id>/` 配下（特に `task.md` と `subagent-output.md`）および `orchestration/task-breakdown.md` を確認して作業してください。
作業場所:
- worktree_path: <assigned worktree path>
```

レビュー担当:

```text
あなたはサブエージェントです。TASK_ID=<id> のレビュー担当です。
`subagent-orchestrator` スキルを必ず利用し、レビュー担当の手順に従ってください。
`orchestration/tasks/<id>/` 配下（特に `task.md`、`subagent-output.md`、`review.md`）および `orchestration/task-breakdown.md` を確認して作業してください。
作業場所:
- worktree_path: <assigned worktree path>
```
