# サブエージェント契約

## 入力必須項目

- `task_id`
- `goal`
- `in_scope`
- `out_of_scope`
- `dependencies`
- `worktree_path`
- `expected_outputs`
- `definition_of_done`
- `required_docs_to_update`

## 出力必須項目

- 割り当て範囲に限定したコードまたは文書変更。
- `definition_of_done` に紐づくテスト証跡。
- `orchestration/tasks/<task-id>/subagent-output.md` への実施内容追記（実施レポート / PR説明文 / 残課題）。
- レビュー担当の場合は `orchestration/review-log.md` への記録。

## エスカレーション条件

- 要件不足で実装が停止する。
- 依存タスクが未完了で着手不可。
- 文書間で意思決定が矛盾する。
- 必須ファイルの担当が他タスクと衝突する。

## 待機・介入ポリシー

- メインエージェントはサブエージェント実行中に途中介入しない。
- メインエージェントは完了応答まで待機を継続する。
- 待機開始から1時間を超えて未完了の場合のみ介入または強制終了する。
- 介入または強制終了時は理由、時刻、次アクションを `orchestration/tasks/<task-id>/task.md` の `Coordinator Notes` に記録する。

## 引継ぎノート

- 何を変更したか、なぜ変更したか、何が未完了かを記載する。
- 正確なファイルパスを記載する。
- 検証に使ったコマンドを記載する。
