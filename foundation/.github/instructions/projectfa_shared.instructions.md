---
name: Project-FA Shared Folder Rules
description: Rules for shared task, decision, spec, and context files
---

# shared/ フォルダ詳細仕様

## 目的

- 複数エージェントが参照・引き継ぐ情報を格納する。
- ここに保存された情報は「合意済み・正式決定事項」として扱う。

## shared/tasks/

- Orchestrator（主）が管理する。
- 全エージェントは status 更新のみ可とする。
- タスク定義、進捗、担当順、受け入れ条件、タイムラインを記録する。

## shared/decisions/

- 実装系（`shared/impl/decisions/`）:
  - ARC は `architecture/` に構成・技術スタック・実装方針の決定を記録する。
  - DEV は `design/` に機能・仕様・UX などの決定を記録する。
- 研究系（`shared/res/decisions/`）:
  - EXD は `experiment/` に実験条件・評価指標・ベンチマーク設計を記録する。
- 承認済みの決定は無断で書き換えない。

## shared/specs/

- 実装系（`shared/impl/specs/`）:
  - `requirements/` は DEV → ARC への要件引き継ぎに使う。
  - `interfaces/` は ARC → IMP へのインターフェース引き継ぎに使う。
- 研究系（`shared/res/specs/`）:
  - `experiment-results/` は EXD → ANL への実験結果分析仕様の引き継ぎに使う。
- 仕様は次工程がそのまま実装・設計に使える粒度で書く。

## shared/context/

- `project-meta.md` と `glossary.md` を置く。
- プロジェクト概要、目標、制約、用語定義を管理する。
- 更新は Orchestrator の承認を経る。

## 変更ルール

- 既存ファイルの変更は最小差分で行う。
- 承認済み情報は差分提案から更新する。
- 不一致がある場合は、根拠と整合性で優先順位を決める。

## 禁止事項

共通の禁止事項は [projectfa_rules.instructions.md](projectfa_rules.instructions.md) を参照。