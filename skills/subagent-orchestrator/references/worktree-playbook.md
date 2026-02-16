# Worktree 運用ガイド

## 命名規約

- ルートディレクトリ: `.worktrees/`
- パス形式: `.worktrees/<task_id>`
- ブランチ形式: `<scope>/<summary-kebab>`
- `scope`: `feat|fix|docs|style|refactor|test|chore|build|ci|perf|revert`
  - コミットメッセージ規約の `type` と同じ語彙を使う。
- `summary-kebab`: タスク内容を非常に簡潔なケバブケースで記述する（40文字以内目安）。

## セットアップ

1. ブランチ作成可能な状態か確認する。
2. `orchestration/tasks/<task-id>/task.md` の frontmatter に `id/status/branch` を記載する。
3. 次を必要に応じて実行する。
   - 新規ブランチ作成込み: `git worktree add -b <branch> .worktrees/<task-id> HEAD`
   - 既存ブランチ利用: `git worktree add .worktrees/<task-id> <branch>`

## タスクファイル前提

```markdown
---
id: T01
summary: 認証画面を追加
status: todo
deps: []
branch: feat/add-auth-screen
---
```

- 実装担当とレビュー担当は同じ `branch` と `worktree` を共有する。
- 1件だけ作成したい場合は該当タスクだけ `git worktree add` を実行する。

## 同期ルール

- 合意した節目で機能ブランチをベースブランチへ追従させる。
- 1つのworktreeを複数タスクIDで使い回さない。
- ブランチ名とworktreeパスを `orchestration/tasks/<task-id>/task.md` の `Coordinator Notes` に記録する。

## クリーンアップ

- マージ後にworktreeを削除する。
  - `git worktree remove <path>`
- 不要ブランチを削除する。
  - `git branch -d <branch>`
