## observility

"foundation" is the base agents which orchestrate and manage models.

1. orc: orchestrator
2. agm: agent manager (architect)
3. agi: agent manager (implementer)

## ドキュメント保存ルール（重要）

本パッケージ（`foundation/`）は**エージェント定義・共通指示の専用領域**である。
**ローカルドキュメント（ログ・タスク・共有・設計書）は保存しない**。

- AI 用ファイル（ログ・タスク・共有・インデックス）は、各プロジェクト内の `docs/_agent/` 配下に保存する
- 人間が読む設計・仕様ドキュメントは、ルート `docs/`（`architect/`・`research/`・`knowledge/`）または各プロジェクトの `docs/` に保存する
- `foundation/`・`code/` には実コンテンツを一切保存しない
