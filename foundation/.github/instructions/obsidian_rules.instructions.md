---
name: Obsidian Common Rules
description: Common rules for all agents
applyTo: "vault/**/*.md"
---

## 5. 権限まとめ

| エージェント | `_inbox/` | `shared/tasks/` | `shared/decisions/` | `shared/specs/` | `shared/context/` | `logs/`                |
| ------------ | --------- | --------------- | ------------------- | --------------- | ----------------- | ---------------------- |
| **ORC**      | ✅        | ✅ 書込         | 読取のみ            | 読取のみ        | ✅ 管理           | —                      |
| **RES**      | ✅        | 読取のみ        | 読取のみ            | 読取のみ        | 追記可            | `logs/research/`       |
| **DEV**      | ✅        | 読取のみ        | `design/`           | `requirements/` | 追記可            | `logs/planning/`       |
| **ARC**      | ✅        | 読取のみ        | `architecture/`     | `interfaces/`   | 追記可            | `logs/architecture/`   |
| **IMP**      | ✅        | 読取のみ        | 読取のみ            | 読取のみ        | —                 | `logs/implementation/` |
| **REV**      | ✅        | 読取のみ        | 読取のみ            | 読取のみ        | —                 | `logs/review/`         |
| **TST**      | ✅        | 読取のみ        | 読取のみ            | 読取のみ        | —                 | `logs/testing/`        |
| **REL**      | ✅        | 読取のみ        | 読取のみ            | 読取のみ        | —                 | `logs/releases/`       |
| **EXD**      | ✅        | 読取のみ        | `experiment/`       | —               | 追記可            | `logs/experiments/`    |
| **ANL**      | ✅        | 読取のみ        | 読取のみ            | 読取のみ        | 追記可            | `logs/analysis/`       |

---

## 7. 禁止事項・注意事項

1. `_inbox/` をスキップして直接 `shared/` や `logs/` に書き込まない

2. フロントマターなしのファイルを作成しない

3. 他エージェントの書き込み権限外フォルダへの書き込み禁止

4. `shared/` 内の承認済みファイルを無断で書き換えない（差分提案は `_inbox/` 経由）

5. `logs/` 内のファイルは参照専用として扱い、後工程への引き継ぎは `shared/` を使う

6. `shared/context/project-meta.md` と `glossary.md` の更新は必ず Orchestrator の承認を経ること
