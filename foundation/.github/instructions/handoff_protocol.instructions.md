---
name: Handoff Protocol
description: Standard handoff format and rules for inter-agent task handover
applyTo: "**/_inbox/**"
---

# Handoff Protocol

サブエージェント間のタスク引き継ぎに使用する定型フォーマットとルールを定義します。

---

## 1. 基本ルール

- ORC が初回ハンドオフで **Task Context** セクションを作成し、チェーン内の全エージェントがこれを**継承・追記**する。
- Task Context の内容は**削除せず**、各エージェントは自分の成果を末尾に**追記**する。
- ハンドオフファイルは `_inbox/` に保存し、必要に応じて `logs/` に複製する。
- 命名規則: `YYYY-MM-DD_HHMM_HANDOFF_{from}_{to}_TASK-{ID}.md`

---

## 2. ハンドオフ種別

| Type | 説明 | 使用場面 |
|------|------|----------|
| `forward` | 次の工程への通常引き継ぎ | DEV→ARC, ARC→IMP など |
| `return` | 完了結果の ORC への返却 | 全エージェントの最終返却 |
| `escalate` | 異常・判断不能時のエスカレーション | 上限超過・矛盾発生時 |

---

## 3. ハンドオフフォーマット（必須テンプレート）

```markdown
---
agent: {from_agent}
task_id: TASK-{ID}
date: YYYY-MM-DD
status: pending
category: log
destination: logs/{from_agent}/
related:
  - "[TASK-{ID}](../shared/tasks/active/TASK-{ID}_{title}.md)"
tags:
  - {from_agent}
  - handoff
  - {category_tag}
---

# HANDOFF: {from_agent} → {to_agent}

## Metadata
| Field | Value |
|-------|-------|
| **From** | {from_agent} |
| **To** | {to_agent} |
| **Task ID** | [TASK-{ID}](../shared/tasks/active/TASK-{ID}_{title}.md) |
| **Status** | success / partial / failed |
| **Confidence** | high / medium / low |
| **Handoff Type** | forward / return / escalate |

---

## Task Context（継承・追記セクション）

> このセクションは ORC が初回に記述し、チェーン内の全エージェントが継承する。
> 各エージェントは自分の成果を **追記** し、既存内容は **削除しない**。

### Original Request
{ユーザの元の依頼内容を ORC が記入}

### Constraints
- {制約1}
- {制約2}

### Chain History
| Step | Agent | Status | Summary |
|------|-------|--------|---------|
| 1 | SRC | done | {要約} |
| 2 | DEV | done | {要約} |
| 3 | ARC | in_progress | — |

---

## Key Findings / Decisions

{自由記述セクション。箇条書き・表・コードブロックなど形式自由。}
{例:
- 決定事項1: xxx
- 発見事項1: yyy
- 推奨案: zzz（根拠: ...）
}

---

## Artifacts
| Path | Type | Description |
|------|------|-------------|
| `shared/decisions/design/DD-042.md` | decision | 認証機能の設計決定 |
| `shared/specs/requirements/REQ-042.md` | spec | 認証機能の要件定義 |

---

## Open Questions
- {未解決の疑問点1}
- {未解決の疑問点2}

---

## Routing
| Field | Value |
|-------|-------|
| **Next Agent** | {to_agent} |
| **Blockers** | none / {blocker_description} |
| **Priority** | high / medium / low |
| **Deadline** | {if applicable} |

---

## ORC Approval（ORC が最終確認時に記入）
- [ ] Approved — proceed to {to_agent}
- [ ] Re-routed to: ___
- Notes: ___
```

---

## 4. ORC からの初回指示テンプレート

ORC がサブエージェントを呼び出す際のプロンプト構造：

```
## ORC Instruction — Task: TASK-{ID}

### Your Mission
{このエージェント固有の簡潔な指示。1-3行で。}

### Input Context
{参照すべきファイルパス・先行エージェントの成果物}
- `shared/specs/requirements/REQ-042.md`
- `shared/context/project-meta.md`

### Expected Output
| Artifact | Destination |
|----------|-------------|
| {成果物1の説明} | {保存先パス} |
| {成果物2の説明} | {保存先パス} |

### Handoff
完了後、以下の HANDOFF を記入して返却すること：
- Metadata / Task Context（追記）/ Key Findings / Artifacts / Open Questions / Routing
```

---

## 5. サブエージェント返却テンプレート

```
## Result Report — TASK-{ID}

### HANDOFF: {from_agent} → {to_agent}
{上記ハンドオフフォーマットの全セクションを記入}

### Role-Specific Output
{役割別の追加項目:
- IMP: 変更ファイル一覧、ビルド結果
- REV: 指摘事項リスト、判定（承認/条件付き/CRITICAL）
- TST: テスト結果サマリ、失敗ケース詳細
- REL: コミットハッシュ、ビルド成果物パス
- EXD: 実験条件・評価指標の詳細
- ANL: 最適解の推奨と根拠
}
```

---

## 6. チェーン委譲モード時の連続ハンドオフ

チェーン委譲モードでは、各エージェントが完了後**直接次のエージェントにハンドオフ**する。

```
【通常モード】
ORC → DEV → ORC → ARC → ORC → IMP → ...

【チェーン委譲モード】
ORC → DEV → ARC → IMP → REV → TST → REL → ORC
        ↑       ↑      ↑     ↑      ↑      ↑
        各矢印で HANDOFF フォーマットを使用
```

### チェーン委譲時の特則
- ORC は最初の DEV にのみ Task Context を注入する。
- 各エージェントは HANDOFF の `Routing.Next Agent` を確認し、指定されたエージェントに直接委譲する。
- CRITICAL 差し戻し（REV→IMP, TST→IMP）が発生した場合、差し戻し元は ORC にも通知する（CC）。
- REL が最終 HANDOFF を ORC に返却して終了。

---

## 7. エスカレーションハンドオフ

`handoff_type: escalate` の場合、以下の追加項目を必須とする：

```markdown
## Escalation Details
| Field | Value |
|-------|-------|
| **Reason** | {エスカレーション理由} |
| **Attempts** | {試行回数} / {上限} |
| **Last Error** | {最後のエラー内容} |
| **Suggested Action** | {推奨される次のアクション} |
```