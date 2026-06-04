---
name: Obsidian Structure Rules
description: Define the Obsidian Structure
---

## 1. エージェント一覧

`ORC`: Orchestrator

`RES`: Researcher

`DEV`: DevPlanner

`ARC`: Architecter

`IMP`: Implementer

`REV`: Reviewer

`TST`: Tester

`REL`: Release Manager

`EXD`: Experiment Designer

`ANL`: Analyst

---

## 2. ディレクトリ構造

```
vault/
│
├── _inbox/                          # ★全エージェント必須の一時書き込み場所
│   └── YYYY-MM-DD_HHMM_{ID}_{slug}.md
│
├── shared/                          # エージェント間共有情報（参照・引き継ぎ用）
│   ├── tasks/                       # タスク定義・進捗管理
│   │   ├── active/                  # 進行中タスク
│   │   │   └── TASK-{ID}_{title}.md
│   │   └── archive/                 # 完了・中断タスク
│   │       └── TASK-{ID}_{title}.md
│   │
│   ├── decisions/                   # 意思決定記録（ADR / Design Decision）
│   │   ├── architecture/            # ARC が書く：構成・技術スタック選定
│   │   │   └── ADR-{ID}_{title}.md
│   │   ├── design/                  # DEV が書く：機能・仕様の決定事項
│   │   │   └── DD-{ID}_{title}.md
│   │   └── experiment/              # EXD が書く：実験設計・評価指標
│   │       └── EXP-{ID}_{title}.md
│   │
│   ├── specs/                       # 仕様書（次工程への引き継ぎドキュメント）
│   │   ├── requirements/            # 要件定義（DEV → ARC への引き継ぎ）
│   │   │   └── REQ-{ID}_{title}.md
│   │   └── interfaces/              # インターフェース仕様（ARC → IMP への引き継ぎ）
│   │       └── IF-{ID}_{title}.md
│   │
│   └── context/                     # プロジェクト共通知識（常時参照）
│       ├── project-meta.md          # プロジェクト概要・目標・制約
│       └── glossary.md              # 用語集・略語定義
│
└── logs/                            # 各エージェントの作業ログ・調査記録
    ├── research/                    # RES：調査結果ログ
    │   └── YYYY-MM-DD_RES_{topic}.md
    ├── planning/                    # DEV：プランニングログ
    │   └── YYYY-MM-DD_DEV_{topic}.md
    ├── architecture/                # ARC：設計検討ログ
    │   └── YYYY-MM-DD_ARC_{topic}.md
    ├── implementation/              # IMP：実装・デバッグログ
    │   └── YYYY-MM-DD_IMP_{topic}.md
    ├── review/                      # REV：コードレビューログ
    │   └── YYYY-MM-DD_REV_{topic}.md
    ├── testing/                     # TST：テスト実行ログ
    │   └── YYYY-MM-DD_TST_{topic}.md
    ├── releases/                    # REL：リリースログ
    │   └── YYYY-MM-DD_REL_v{version}.md
    ├── experiments/                 # EXD：実験実施ログ
    │   └── YYYY-MM-DD_EXD_{topic}.md
    └── analysis/                    # ANL：分析・考察ログ
        └── YYYY-MM-DD_ANL_{topic}.md
```

---

## 5. 権限まとめ
| エージェント  | `_inbox/` | `shared/tasks/` | `shared/decisions/` | `shared/specs/` | `shared/context/` | `logs/`                |
| ------- | --------- | --------------- | ------------------- | --------------- | ----------------- | ---------------------- |
| **ORC** | ✅         | ✅ 書込            | 読取のみ                | 読取のみ            | ✅ 管理              | —                      |
| **RES** | ✅         | 読取のみ            | 読取のみ                | 読取のみ            | 追記可               | `logs/research/`       |
| **DEV** | ✅         | 読取のみ            | `design/`           | `requirements/` | 追記可               | `logs/planning/`       |
| **ARC** | ✅         | 読取のみ            | `architecture/`     | `interfaces/`   | 追記可               | `logs/architecture/`   |
| **IMP** | ✅         | 読取のみ            | 読取のみ                | 読取のみ            | —                 | `logs/implementation/` |
| **REV** | ✅         | 読取のみ            | 読取のみ                | 読取のみ            | —                 | `logs/review/`         |
| **TST** | ✅         | 読取のみ            | 読取のみ                | 読取のみ            | —                 | `logs/testing/`        |
| **REL** | ✅         | 読取のみ            | 読取のみ                | 読取のみ            | —                 | `logs/releases/`       |
| **EXD** | ✅         | 読取のみ            | `experiment/`       | —               | 追記可               | `logs/experiments/`    |
| **ANL** | ✅         | 読取のみ            | 読取のみ                | 読取のみ            | 追記可               | `logs/analysis/`       |

---

## 7. 禁止事項・注意事項

1. `_inbox/` をスキップして直接 `shared/` や `logs/` に書き込まない

2. フロントマターなしのファイルを作成しない

3. 他エージェントの書き込み権限外フォルダへの書き込み禁止

4. `shared/` 内の承認済みファイルを無断で書き換えない（差分提案は `_inbox/` 経由）

5. `logs/` 内のファイルは参照専用として扱い、後工程への引き継ぎは `shared/` を使う

6. `shared/context/project-meta.md` と `glossary.md` の更新は必ず Orchestrator の承認を経ること