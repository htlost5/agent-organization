---
agent: ORC
task_id: N/A
date: 2026-07-07
status: draft
category: shared
destination: shared/context/
related: []
tags:
  - ORC
  - index
  - context
---

# プロジェクト全体インデックス

## ドメイン一覧

| ドメイン | パス | インデックス | 説明 |
|---------|------|-------------|------|
| foundation | `foundation/` | — | エージェント基盤（ORC/AGM/AGI） |
| code | `code/` | [code/project-index.md](code/project-index.md) | 実装系エージェント・コード |
| research | `research/` | [research/project-index.md](research/project-index.md) | 研究系エージェント・実験 |
| search | `search/` | [search/project-index.md](search/project-index.md) | 検索系エージェント・調査 |

## foundation 構造

```
foundation/
├── .github/
│   ├── agents/          ← エージェント定義ファイル (.agent.md)
│   ├── config/           ← 運用設定 (ops_config.yml)
│   ├── instructions/     ← 全エージェント共通指示書
│   └── copilot-instructions.md  ← 共通ルール
└── docs/
    └── shared/context/   ← プロジェクト共通知識
```

## キーファイル

| パス | 用途 |
|------|------|
| `foundation/.github/copilot-instructions.md` | 全エージェント共通ルール |
| `foundation/.github/config/ops_config.yml` | 運用パラメータ・制限値 |
| `foundation/.github/instructions/localdocs_rules.instructions.md` | Local Docs 権限・禁止事項 |
| `foundation/.github/instructions/handoff_protocol.instructions.md` | ハンドオフフォーマット |
| `foundation/docs/shared/context/project-index.md` | このファイル |
