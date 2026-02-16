# Conventional Commits ガイド

## 必須フォーマット

```text
<type>: <summary>

<blank line>
<line 3+ details>
```

- `type`: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`, `revert`
- `scope`: 使わない。
- `summary`: 日本語で簡潔に書く（40文字以内を目安）。
- 2行目: 空行にする。
- 3行目以降: 変更の詳細、背景、検証内容を書く。

## 例

```text
feat: 依存バッチ生成を追加

- orchestration/tasks/T01/task.md などからDAGを生成
- 並列バッチの算出結果を出力
- bun run scripts/integration_order.ts で動作確認
```

## 破壊的変更

- `:` の前に `!` を付ける。
- 例:

```text
feat!: API契約をv2へ変更

- 旧レスポンス形式を廃止
- クライアント更新手順を docs に追記
```

## 運用方針

- 形式違反のコミットは統合しない。
- 1コミット1論理変更を優先する。
- `bun run scripts/cc_commit_check.ts` で事前検証する。
