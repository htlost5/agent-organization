---
name: Obsidian Format Rules
description: Rules for All Files Construction
applyTo: '**'
---

## 共通ルール

### 3.1 フロントマター仕様（全エージェント共通）

全ファイルの冒頭に以下の YAML フロントマターを必ず記述する。

```
---
agent: {AgentID}              # 例: RES, ORC, ARC
task_id: TASK-{ID}            # 紐付けタスクID（例: TASK-042）
date: YYYY-MM-DD              # 作成日
status: draft                 # draft | pending | approved | archived
category: shared | log        # 共有情報か作業ログか
destination: shared/specs/requirements/   # 正式保存先パス
related:                      # 関連ノートへのリンク（Obsidian内部リンク）
  - "[[TASK-042_build-auth]]"
tags:
  - {AgentID}                 # エージェントID（例: RES）
  - {category-tag}            # 内容カテゴリタグ（例: research, architecture）
  - {task_id}                 # タスクIDタグ
---
```

### 3.2 ステータス定義

| status       | 意味               | 次のアクション                                |
|--------------|--------------------|---------------------------------------------|
| `draft`      | 作成中・未完了      | 作成エージェントが完成後 `pending` へ更新       |
| `pending`    | レビュー・承認待ち  | Orchestrator または指定エージェントが確認       |
| `approved`   | 承認済み・正式反映  | 正式ディレクトリへ移動                        |
| `archived`   | 完了・参照専用      | `archive/` サブフォルダへ移動                 |

### 3.3 タグ体系

```
タグ種別          例
─────────────────────────────────────
エージェント       #ORC #RES #DEV #ARC #IMP #REV #TST #REL #EXD #ANL
カテゴリ           #task #decision #spec #research #review #test #release #experiment #analysis
フォルダ系統       #shared #log
優先度             #priority-high #priority-medium #priority-low
状態               #blocked #in-progress #done
```

### 3.4 命名規則

- `_inbox/` 内のファイル名は `YYYY-MM-DD_HHMM_{AgentID}_{slug}.md` に従う。
  詳細は [obsidian_structure.instructions.md](obsidian_structure.instructions.md) を参照。
