---
name: Local Docs Structure Rules
description: Define the Local Docs Directory Structure
---

## 1. エージェント一覧

`ORC`: Orchestrator — foundation

`RES`: Researcher — foundation

`DEV`: DevPlanner — code

`ARC`: Architect — code

`IMP`: Implementer — code

`REV`: Reviewer — code

`TST`: Tester — code

`REL`: Release Manager — code

`EXD`: Experiment Designer — research

`ANL`: Analyst — research

---

## 2. ディレクトリ構造

```
{project_name}/
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
    └── res/                         # 研究系ログ
        ├── research/                # RES
        │   └── YYYY-MM-DD_RES_{topic}.md
        ├── experiments/             # EXD
        │   └── YYYY-MM-DD_EXD_{topic}.md
        └── analysis/                # ANL
            └── YYYY-MM-DD_ANL_{topic}.md
```

---

## 3. 参照

- 権限まとめと禁止事項は [localdocs_rules.instructions.md](localdocs_rules.instructions.md) を参照。