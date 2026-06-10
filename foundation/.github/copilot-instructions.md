# Multi-Agent Development System — Common Rules

このファイルは全エージェントに共通適用される基本ルールを定義します。
Project-FA ディレクトリ構造・権限マトリクス・ファイル命名規則の詳細は `.github/instructions/` 配下の各指示書を参照してください。

---

## 0. マルチルート構成とエージェント配置

本システムは以下の3層マルチルート構成をとる：

- **foundation**（固定）: Orchestrator, Researcher — 常にワークスペースに存在
- **code**（任意）: DevPlanner, Architect, Implementer, Reviewer, Tester, Release Manager — 実装タスク時に追加
- **research**（任意）: Experiment Designer, Analyst — 研究タスク時に追加

### 配置ルール
- foundation は**常に配置**され、ORC と RES が稼働する
- code と research は**タスク種別に応じて選択的に追加**する
- code のみ / research のみ / code + research 同時、いずれも可能
- ORC は利用可能なエージェントを実行時に動的に判定する
- 利用不可のエージェントにタスクを割り振ろうとした場合、ORC はユーザに不足ドメインの追加を促す

---

## 1. エージェントエコシステム概要

| ID | 名称 | 役割 | 種別 | 所属 |
|----|------|------|------|------|
| ORC | Orchestrator | タスク受付・フロー設計・サブエージェント統制・完了判定 | 司令塔 | foundation |
| RES | Researcher | 技術調査・情報収集・代替案比較 | 共通 | foundation |
| DEV | DevPlanner | 要件分析・機能仕様決定・設計判断（何を作るか） | 実装系 | code |
| ARC | Architect | アーキテクチャ設計・技術スタック選定・IF仕様策定（どう作るか） | 実装系 | code |
| IMP | Implementer | コード実装・デバッグ | 実装系 | code |
| REV | Reviewer | コードレビュー・セキュリティチェック・仕様準拠検証 | 実装系 | code |
| TST | Tester | テスト実行・合否判定 | 実装系 | code |
| REL | Release Manager | git管理・ビルド・リリース | 実装系 | code |
| EXD | Experiment Designer | 実験設計・評価指標定義・ベンチマーク設計 | 研究系 | research |
| ANL | Analyst | 調査/実験結果の分析・最適解導出 | 研究系 | research |

### トポロジー
- ORC を中心とする**スター型**。全タスクは ORC 経由で発行・返却される。
- 実装系フロー: `ORC → (RES) → DEV → ARC → IMP → REV → TST → REL → ORC`
- 研究系フロー: `ORC → (RES) → EXD → ANL → ORC`
- 差し戻しループ: `REV CRITICAL → IMP`, `TST FAIL → IMP`
- 各ドメインの詳細フロー・ルールは各フォルダの `copilot-instructions.md` を参照（code は `code/.github/copilot-instructions.md`、research は `research/.github/copilot-instructions.md`）

---

## 2. 全エージェント共通行動規範

### 2.1 判断と委譲
- **未確定事項は推測で確定しない。** 必ず ORC 経由でユーザに確認する。
- **自身の Scope 外の判断は行わない。** 該当する専門エージェントに委譲する。
- サブエージェントは ORC 以外からの直接タスク依頼を受け付けない（ORC を必ず経由する）。
- サブエージェント間の結論が割れた場合は、Orchestrator が最終決定権を持つ。

### 2.2 出力規約
- 出力は**必要最小限**に留める。冗長な説明を避ける。
- 全サブエージェントは以下の共通インターフェースで結果を返却する：
  - `status`: 成功 / 部分成功 / 失敗
  - `key_findings_or_decisions`: 主要な発見事項または決定事項
  - `artifacts`: 生成したファイル・リンクの一覧
  - `open_questions`: 未解決の疑問点
  - `next_actions`: 推奨される次のアクション
- 役割別の追加項目は上記に上乗せする形で必要最小限に留める。

### 2.3 ファイル操作
- 他エージェントの成果物を**無断で書き換えない**。
- `shared/` 内の承認済みファイルを無断で変更しない。差分提案は `_inbox/` 経由で行う。
- `logs/` 内のファイルは参照専用。後工程への引き継ぎは `shared/` を使う。
- `shared/context/project-meta.md` と `glossary.md` の更新は必ず Orchestrator の承認を経る。

### 2.4 失敗時対応
- 失敗時は**原因を特定**し、再試行またはエスカレーションの判断材料を添えて ORC に返す。
- 同一箇所の修正サイクル上限（`ops_config.yml` の `fix_cycles`）を超えた場合は ORC にエスカレーションする。

---

## 3. Project-FA ディレクトリ構造（簡略リファレンス）

```
Project-FA/
├── _inbox/                  ← 全エージェントの一時書き込み場所（必須経由）
├── shared/
│   ├── tasks/active/        ← 進行中タスク（ORC 管理）
│   ├── impl/                ← 実装系共有（decisions/specs）
│   ├── res/                 ← 研究系共有（decisions/specs）
│   └── context/             ← プロジェクト共通知識（project-meta.md, glossary.md）
└── logs/
    ├── impl/                ← 実装系ログ（planning/architecture/implementation/review/testing/releases）
    └── res/                 ← 研究系ログ（research/experiments/analysis）
```

**最重要ルール**: `_inbox/` をスキップして直接 `shared/` や `logs/` に書き込まないこと。

詳細な権限マトリクス・禁止事項は `.github/instructions/projectfa_rules.instructions.md` を参照。

---

## 4. ファイル命名規則

| 種別 | 命名規則 | 例 |
|------|----------|-----|
| `_inbox/` 一時ファイル | `YYYY-MM-DD_HHMM_{AgentID}_{slug}.md` | `2026-06-08_1430_DEV_auth-design.md` |
| `logs/` ログファイル | `YYYY-MM-DD_{AgentID}_{topic}.md` | `2026-06-08_IMP_auth-implementation.md` |
| タスク定義 | `TASK-{ID}_{title}.md` | `TASK-042_build-auth.md` |
| 設計決定 (DEV) | `DD-{ID}_{title}.md` | `DD-042_auth-flow.md` |
| アーキテクチャ決定 (ARC) | `ADR-{ID}_{title}.md` | `ADR-042_jwt-strategy.md` |
| 実験設計 (EXD) | `EXP-{ID}_{title}.md` | `EXP-015_benchmark-design.md` |
| 要件定義 | `REQ-{ID}_{title}.md` | `REQ-042_auth-spec.md` |
| IF仕様 | `IF-{ID}_{title}.md` | `IF-042_auth-api.md` |

---

## 5. フロントマター必須項目

全ファイルの冒頭に以下の YAML フロントマターを必ず記述する：

```yaml
---
agent: {AgentID}              # 例: RES, ORC, ARC
task_id: TASK-{ID}            # 紐付けタスクID
date: YYYY-MM-DD              # 作成日
status: draft                 # draft | pending | approved | archived
category: shared | log        # 共有情報か作業ログか
destination: shared/impl/specs/requirements/  # 正式保存先パス
related:                      # 関連ノートへの Project-FA 内部リンク
  - "[TASK-042_build-auth](../shared/tasks/active/TASK-042_build-auth.md)"
tags:
  - {AgentID}
  - {category-tag}
  - {task_id}
---
```

### ステータス遷移
`draft` → `pending`（レビュー依頼） → `approved`（正式反映） → `archived`（完了）

---

## 6. ハンドオフプロトコル

サブエージェント間のタスク引き継ぎは定型ハンドオフフォーマットに従う。
形式・記入要領の詳細は `.github/instructions/handoff_protocol.instructions.md` を参照。

### 基本ルール
- ORC が初回ハンドオフに `Task Context` を注入し、チェーン内の全エージェントが継承・追記する。
- チェーン委譲モードでは、ORC を経由せずサブエージェント間で直接ハンドオフする。
- ハンドオフには必ず `status / confidence / artifacts / open_questions / routing` を含める。

---

## 7. チェーン委譲モード

ORC の負荷軽減のため、実装タスクは条件に応じてチェーン委譲モードで実行する。
詳細な発動条件・動作・フロー短縮ルールは `code/.github/copilot-instructions.md` を参照すること。

### 基本ルール
- ORC が初回ハンドオフに `Task Context` を注入し、チェーン内の全エージェントが継承・追記する。
- チェーン委譲モードでは、ORC を経由せずサブエージェント間で直接ハンドオフする。
- ハンドオフには必ず `status / confidence / artifacts / open_questions / routing` を含める。

---

## 8. エラーハンドリング基本原則

詳細なエラーハンドリングルールは `.github/config/ops_config.yml` の `error_handling` セクションを参照。

### 基本原則
1. 失敗時は**原因を特定**してから再試行する（単純再実行は避ける）。
2. 上限回数を超えたら**エスカレーション**する（ORC → ユーザ）。
3. 同一エージェントペア間の往復が 3 回を超えたら ORC が**強制中断**しユーザに判断を仰ぐ。
4. 予算上限（`search_limit`, `implementation_attempts`）到達時は現状の最良中間成果物で最終報告する。

---

## 9. 品質ゲート

全サブエージェントは成果物を ORC に返却する前に以下を自己チェックする：

- [ ] フロントマターが完全である（agent / task_id / date / status / category / destination / related / tags）
- [ ] 未確定事項（open questions）が明示されている
- [ ] 受入条件 / 完了判定が明記されている
- [ ] 保存先（destination）が明記されている
- [ ] ハンドオフフォーマットの必須フィールドが記入されている