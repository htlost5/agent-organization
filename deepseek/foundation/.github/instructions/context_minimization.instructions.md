---
name: Context Minimization Rules
description: >
  Universal context minimization rules applicable to all agents across all domains.
  Defines read-before-search protocol, full-file read prohibition, project index
  usage, and token budget guidelines. Use when: any agent reads files or explores
  a codebase. Applies to all agents (ORC/SRC/AGM/AGI/DEV/ARC/IMP/REV/TST/REL/EXD/ANL).
applyTo: "**/*"
---

# Context Minimization Rules

全エージェント共通のコンテキスト最小化ルール。トークン消費を抑え、必要な情報のみを効率的に取得するための行動規範。

---

## 1. 読取前検索プロトコル（Search-Before-Read）

未知のコードベース・プロジェクトを探索する場合、以下の順序を**必ず**遵守する:

1. **インデックス参照**: まず `shared/context/project-index.md`（および該当ドメインのインデックス）を読み、対象ファイルの候補を絞り込む
2. **検索で候補特定**: `grep_search` または `file_search` で関連ファイル・関連行を特定する
3. **行範囲読み取り**: `read_file` は必ず `startLine`/`endLine` を指定し、必要最小限の範囲のみ読み取る

**禁止パターン**:

- インデックス未参照での闇雲な `file_search`
- 検索未実施での `read_file` 全文読み取り
- 行範囲未指定の `read_file`

---

## 2. 全文読み取り禁止ルール

以下の例外を除き、**行範囲未指定の `read_file` は禁止**する:

### 許可される例外

- `shared/context/project-index.md` および各ドメイン・フォルダの `index.md`（インデックスファイルは全体把握のため全文読み取り可）
- 50行未満の設定ファイル（`.yml`, `.json`, `.toml`, `.env` 等）
- ORC の `Input Context` で明示的に「全文読み取り」が指示されたファイル
- 新規作成するファイルのテンプレート（書き込み前に雛形を読む場合）

### 判断基準

- 読み取り前に必ず「このファイルの全文が必要か」を自問する
- 必要なセクションが特定できている場合は、その行範囲のみを読み取る

---

## 3. プロジェクトインデックス

### インデックス構造（2階層分散）

```
foundation/docs/shared/context/
├── project-index.md          ← 全体マスタインデックス（ORC 管理）
├── code/project-index.md     ← code ドメインインデックス（DEV/IMP が更新）
├── research/project-index.md ← research ドメインインデックス（EXD が更新）
└── search/project-index.md   ← search ドメインインデックス（SRC が更新）
```

各ドメインの `project-index.md` は、フォルダ単位の `index.md` へのリンクを持つ。

### 責務

| 責務                                       | 担当                                    |
| ------------------------------------------ | --------------------------------------- |
| 全体マスタインデックス生成・更新           | ORC                                     |
| ドメインインデックス生成・更新             | DEV(code) / EXD(research) / SRC(search) |
| フォルダ単位 index.md 生成・更新           | IMP(code) / EXD(research) / SRC(search) |
| インデックス参照（タスク開始時の初回行動） | 全エージェント                          |
| インデックス不足検知・追記提案             | DEV / ARC / IMP                         |

---

## 4. ORC の Input Context 注入ルール

ORC はサブエージェント起動時に以下を `Input Context` として渡す:

1. `project-index.md` の関連セクション（該当モジュールのエントリ行）
2. 前工程の成果物パス（1〜2ファイルに限定）
3. 参照すべき既存コードのキーワード（grep 用）

サブエージェントは `Input Context` でパスが明示されている場合、そのファイルの検索をスキップしてよい。

---

## 5. トークン予算ガイドライン

| フェーズ                            | 目安            |
| ----------------------------------- | --------------- |
| 初回探索（インデックス参照 + grep） | 全体の 20% 以内 |
| 関連コード読み取り                  | 全体の 50% 以内 |
| 出力生成                            | 全体の 30% 以内 |

- 読み取りは常に「必要最小限の行範囲」で行い、総読み取り行数を意識する
- 同一ファイルを複数回に分けて読むより、やや広めの行範囲で1回で読むことを推奨

---

## 6. 品質チェック（自己検証）

全エージェントはファイル読み取り前に以下を自己チェックする:

- [ ] インデックスを参照したか
- [ ] 検索で候補を絞ったか
- [ ] 行範囲を指定したか
- [ ] 全文読み取りの例外条件に該当するか（該当する場合のみ全文読み取り許可）
