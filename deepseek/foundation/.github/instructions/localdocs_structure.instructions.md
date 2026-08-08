---
name: Local Docs Structure Rules
description: Define the Local Docs Directory Structure
---

## 1. エージェント一覧

`ORC`: Orchestrator — foundation

`SRC`: Searcher — surfing

`AGM`: Agent Manager (Architect) — foundation

`AGI`: Agent Manager (Implementer) — foundation

`DWR`: Document Writer — foundation

`DEV`: DevPlanner — code

`ARC`: Architect — code

`IMP`: Implementer — code

`REV`: Reviewer — code

`TST`: Tester — code

`REL`: Release Manager — code

`EXD`: Experiment Designer — research

`ANL`: Analyst — research

---

## 0. 配置場所の前提（重要）

本共通基盤はエージェント定義・共通指示の**専用領域**であり、**実際のローカルドキュメント（ログ・タスク・共有・設計書）は保存しない**。

各プロジェクト（`mobile/`・`position-engine/`・`research/` 等）の AI 用ファイルは、**各プロジェクト内の `docs/_agent/`** 配下に配置する。

```
<project>/docs/_agent/
├── _inbox/          # 一時書き込み（承認前の提案・ハンドオフ）
├── _archive/        # 破棄・完了済みの一時ファイル
├── logs/            # 各エージェントの作業ログ（impl/ 以下に planning/architecture/implementation/review/testing/releases）
└── shared/          # エージェント間共有（tasks/・context/・impl/・search/・res/）
```

- **人間が読む設計・仕様ドキュメント**（複数プロジェクト横断）はルート `docs/architect/`・`docs/research/`・`docs/knowledge/`、プロジェクト固有のものは各プロジェクトの `docs/` に保存する。
- **`foundation/`・`code/` には実コンテンツ（ログ・タスク・設計書）を一切保存しない**。AI 用ファイルの配置先は各プロジェクトの `docs/_agent/`、人間用ドキュメントの配置先は各プロジェクトの `docs/`（またはルート `docs/`）である。
- 下記の構造図は、`<project>/docs/_agent/` 配下を基準として読むこと（`docs/` と表記のある箇所は `docs/_agent/` と読み替える）。

---

## 2. ディレクトリ構造

```
docs/
│
├── _inbox/                          # 全エージェント必須の一時書き込み場所
│   └── YYYY-MM-DD_HHMM_{ID}_{slug}.md
│
├── shared/                          # エージェント間共有情報
│   ├── tasks/                       # タスク定義・進捗管理（foundation: ORC）
│   │   ├── active/
│   │   │   └── TASK-{ID}_{title}.md
│   │   └── archive/
│   │       └── TASK-{ID}_{title}.md
│   │
│   ├── context/                     # プロジェクト共通知識（foundation）
│   │   ├── project-meta.md
│   │   └── glossary.md
│   │
│   ├── impl/                        # 実装系共有情報（code ドメイン）
│   │   ├── decisions/
│   │   │   ├── design/              # DEV: 機能・仕様の決定事項
│   │   │   │   └── DD-{ID}_{title}.md
│   │   │   └── architecture/        # ARC: 構成・技術スタック選定
│   │   │       └── ADR-{ID}_{title}.md
│   │   └── specs/
│   │       ├── requirements/        # DEV → ARC: 要件定義
│   │       │   └── REQ-{ID}_{title}.md
│   │       └── interfaces/          # ARC → IMP: インターフェース仕様
│   │           └── IF-{ID}_{title}.md
│   │
│   ├── search/                      # 検索系共有情報（surfing ドメイン）
│   │   ├── decisions/
│   │   │   └── search/              # SRC: 調査方針
│   │   │       └── SD-{ID}_{title}.md
│   │   └── specs/
│   │       └── search-results/      # SRC: 調査結果
│   │           └── SR-{ID}_{title}.md
│   │
│   └── res/                         # 研究系共有情報（research ドメイン）
│       ├── decisions/
│       │   └── experiment/          # EXD: 実験設計・評価指標
│       │       └── EXP-{ID}_{title}.md
│       └── specs/
│           └── experiment-results/  # EXD → ANL: 実験結果
│               └── ER-{ID}_{title}.md
│
└── logs/                            # 各エージェントの作業ログ
    ├── impl/                        # 実装系ログ
    │   ├── planning/                # DEV
    │   │   └── YYYY-MM-DD_DEV_{topic}.md
    │   ├── architecture/            # ARC
    │   │   └── YYYY-MM-DD_ARC_{topic}.md
    │   ├── implementation/          # IMP
    │   │   └── YYYY-MM-DD_IMP_{topic}.md
    │   ├── review/                  # REV
    │   │   └── YYYY-MM-DD_REV_{topic}.md
    │   ├── testing/                 # TST
    │   │   └── YYYY-MM-DD_TST_{topic}.md
    │   └── releases/                # REL
    │       └── YYYY-MM-DD_REL_v{version}.md
├── search/                      # 検索系ログ
    │   └── YYYY-MM-DD_SRC_{topic}.md
    │
    ├── res/                         # 研究系ログ
    │   ├── research/                # SRC
    │   │   └── YYYY-MM-DD_SRC_{topic}.md
        ├── experiments/             # EXD
        │   └── YYYY-MM-DD_EXD_{topic}.md
        └── analysis/                # ANL
            └── YYYY-MM-DD_ANL_{topic}.md
```

---

## 3. 参照

- 権限まとめと禁止事項は [localdocs_rules.instructions.md](localdocs_rules.instructions.md) を参照。

---

## 4. docs/ ディレクトリ自動作成ルール

- 全エージェントはローカルドキュメントを保存する前に、対象プロジェクトのルートディレクトリ直下に `docs/_agent/` が存在するか確認する
- `docs/_agent/` が存在しない場合、以下のディレクトリを自動作成する:
  - `docs/_agent/_inbox/`
  - `docs/_agent/_archive/`
  - `docs/_agent/shared/tasks/active/`
  - `docs/_agent/shared/tasks/archive/`
  - `docs/_agent/shared/context/`
  - `docs/_agent/shared/impl/decisions/design/`
  - `docs/_agent/shared/impl/decisions/architecture/`
  - `docs/_agent/shared/impl/specs/requirements/`
  - `docs/_agent/shared/impl/specs/interfaces/`
  - `docs/_agent/shared/search/decisions/search/`
  - `docs/_agent/shared/search/specs/search-results/`
  - `docs/_agent/shared/res/decisions/experiment/`
  - `docs/_agent/shared/res/specs/experiment-results/`
  - `docs/_agent/logs/impl/planning/`
  - `docs/_agent/logs/impl/architecture/`
  - `docs/_agent/logs/impl/implementation/`
  - `docs/_agent/logs/impl/review/`
  - `docs/_agent/logs/impl/testing/`
  - `docs/_agent/logs/impl/releases/`
  - `docs/_agent/logs/search/`
  - `docs/_agent/logs/res/research/`
  - `docs/_agent/logs/res/experiments/`
  - `docs/_agent/logs/res/analysis/`
- `docs/_agent/` が既に存在する場合はそのまま使用する
- 自動作成処理は各エージェントの初回書き込み時に一度だけ実行する
- **`foundation/`・`code/` 直下に `docs/` を自動作成・利用しない**（エージェント定義専用領域のため）