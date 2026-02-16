# ドキュメント運用ガイド

## 目的

どのエージェントでも即時に作業再開できる状態を、リポジトリ内文書で維持する。

## 更新タイミング

- スコープや制約が変わった時に `orchestration/charter.md` を更新する。
- タスク、担当、依存が変わった時に `orchestration/task-breakdown.md` を更新する。
- タスクの要件・受け入れ条件・調整事項が変わった時に `orchestration/tasks/<task-id>/task.md` を更新する。
- `orchestration/tasks/<task-id>/task.md` の frontmatter を変更した後に `bun run scripts/integration_order.ts --tasks-dir orchestration/tasks --write orchestration/dependency-dag.md` を再実行し、`orchestration/dependency-dag.md` と `orchestration/ready-now.md` を更新する。
- サブエージェントの実施内容・PR説明・残課題が変わった時に `orchestration/tasks/<task-id>/subagent-output.md` を更新する。
- サブエージェント待機開始時刻を `orchestration/tasks/<task-id>/task.md` の `Coordinator Notes` に記録する。
- 1時間超過で介入または強制終了した場合、判断理由と結果を `orchestration/tasks/<task-id>/task.md` の `Coordinator Notes` に記録する。
- レビューごとに `orchestration/review-log.md` を更新する。
- 統合作業ごとに `orchestration/integration-log.md` を更新する。
- 完了時または担当移管時に `orchestration/handover.md` を更新する。

## 最小記録フォーマット

- UTCタイムスタンプ
- 実行者（agent id または名前）
- 実施内容
- 結果
- 次アクション

## 整合性ルール

- すべての文書で同一のタスクIDを使う。
- すべてのログで同一のブランチ/worktree命名を使う。
- ブロッカー記録には必ず担当者と次アクションを含める。
