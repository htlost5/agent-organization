# Multi-Agent Development System — Common Rules

このファイルは全エージェントに共通適用される基本ルールを定義します。
Obsidian Vault 構造・権限マトリクス・ファイル命名規則の詳細は `.github/instructions/` 配下の各指示書を参照してください。

---

## 1. エージェントエコシステム概要

| ID | 名称 | 役割 | 種別 |
|----|------|------|------|
| ORC | Orchestrator | タスク受付・フロー設計・サブエージェント統制・完了判定 | 司令塔 |
| RES | Researcher | 技術調査・情報収集・代替案比較 | 共通 |
| DEV | DevPlanner | 要件分析・機能仕様決定・設計判断（何を作るか） | 実装系 |
| ARC | Architect | アーキテクチャ設計・技術スタック選定・IF仕様策定（どう作るか） | 実装系 |
| IMP | Implementer | コード実装・デバッグ | 実装系 |
| REV | Reviewer | コードレビュー・セキュリティチェック・仕様準拠検証 | 実装系 |
| TST | Tester | テスト実行・合否判定 | 実装系 |
| REL | Release Manager | git管理・ビルド・リリース | 実装系 |
| EXD | Experiment Designer | 実験設計・評価指標定義・ベンチマーク設計 | 研究系 |
| ANL | Analyst | 調査/実験結果の分析・最適解導出 | 研究系 |

### トポロジー
- ORC を中心とする**スター型**。全タスクは ORC 経由で発行・返却される。
- 実装系フロー: `ORC → (RES) → DEV → ARC → IMP → REV → TST → REL → ORC`
- 研究系フロー: `ORC → (RES) → EXD → ANL → ORC`
- 差し戻しループ: `REV CRITICAL → IMP`, `TST FAIL → IMP`

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

## 3. Obsidian Vault 構造（簡略リファレンス）

```
vault/
├── _inbox/                  ← 全エージェントの一時書き込み場所（必須経由）
├── shared/
│   ├── tasks/active/        ← 進行中タスク（ORC 管理）
│   ├── decisions/           ← 意思決定記録（architecture/design/experiment）
│   ├── specs/               ← 仕様書（requirements/interfaces/experiment-results）
│   └── context/             ← プロジェクト共通知識（project-meta.md, glossary.md）
└── logs/                    ← 各エージェントの作業ログ
```

**最重要ルール**: `_inbox/` をスキップして直接 `shared/` や `logs/` に書き込まないこと。

詳細な権限マトリクス・禁止事項は `.github/instructions/obsidian_rules.instructions.md` を参照。

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
destination: shared/specs/requirements/  # 正式保存先パス
related:                      # 関連ノートへの Obsidian 内部リンク
  - "[[TASK-042_build-auth]]"
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
- チェーン委譲モード（後述）では、ORC を経由せずサブエージェント間で直接ハンドオフする。
- ハンドオフには必ず `status / confidence / artifacts / open_questions / routing` を含める。

---

## 7. チェーン委譲モード

ORC の負荷軽減のため、以下の条件をすべて満たす実装タスクは **チェーン委譲モード** で実行する：

### 発動条件
1. タスクカテゴリが「実装」である
2. 要件が明確で、DEV の判断余地が小さい
3. 既知の技術スタックを使用する
4. ORC の確信度が 85%（中）以上である
5. 研究要素を含まない純粋な実装タスクである

### 動作
- ORC は `DEV→ARC→IMP→REV→TST→REL` を **1回のバッチ指示で起動** する。
- 各サブエージェントはハンドオフフォーマットを用いて**直接次のエージェントに引き継ぐ**。
- CRITICAL 差し戻し（REV→IMP, TST→IMP）が発生した場合のみ ORC にエスカレーションする。
- REL が完了したら ORC に最終報告する。

### フロー短縮（`ops_config.yml` 参照）
- **trivial**（タイポ修正等）: `IMP → TST → REL`
- **simple**（小規模ユーティリティ追加等）: `DEV → IMP → REV → TST → REL`
- **standard**: フルフロー

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