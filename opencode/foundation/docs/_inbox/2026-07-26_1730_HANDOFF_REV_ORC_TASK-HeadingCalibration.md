---
agent: REV
task_id: TASK-HeadingCalibration
date: 2026-07-26
status: pending
category: log
destination: logs/impl/review/
tags:
  - REV
  - handoff
  - escalate
  - heading-calibration
---

# HANDOFF: REV → ORC

## Metadata
| Field | Value |
|-------|-------|
| **From** | REV |
| **To** | ORC |
| **Task ID** | TASK-HeadingCalibration |
| **Status** | failed |
| **Confidence** | high |
| **Handoff Type** | escalate |

---

## Task Context（継承）

> PDR 歩幅較正に「方位オフセット較正」機能を追加。
> 既存の Node-to-Node 歩行による歩幅較正と同時に、PDR 方位とグラフ Edge 方向のオフセットを較正する。

### Constraints
- `enableHeadingCalibration=false` の場合は既存動作と完全互換
- 方位の循環性（0°/360°境界）を考慮した平均・差分計算
- デフォルト値は `CalibrationParams.defaults()` に一元定義
- Kotlin コードスタイル: 4スペースインデント、`lowerCamelCase`

### Chain History
| Step | Agent | Status | Summary |
|------|-------|--------|---------|
| 1 | IMP | done | 7ファイル実装完了 |

---

## Review Result: 🔴 CRITICAL

### CRITICAL Issues Found

**CRITICAL #1 — `computeHeadingOffset()` に `return` 不足（コンパイルエラー）**
- **File**: `StrideCalibrator.kt` L310-330
- **Detail**: `when` 式の結果が関数から返されていない。Kotlin のブロックボディ関数では最後の式が自動的に戻り値にならないため、コンパイルエラーとなる。
- **Fix**: `return when { ... }` に修正

**CRITICAL #2 — `headingSamples` のスレッド安全性欠如**
- **File**: `HeadingEstimator.kt` L40, L133, L146, L158-181
- **Detail**: `mutableListOf<Float>()` が複数コルーチンから同期なしでアクセスされている。ConcurrentModificationException のリスク。
- **Fix**: `Collections.synchronizedList()` またはコピーオンリードの導入

### Minor Issues
- JSON `calibration` セクションに `cvWarningThreshold` なし（動作影響なし）
- 方位較正のテストケース未実装

### Confirmed Correct
- FR-1 ✅: deltaX/deltaY 座標系修正
- FR-2 ✅: CalibrationParams 4パラメータ
- FR-4 ✅: StrideCalibrator 拡張・後方互換性
- FR-6 ✅: 補正適用・正規化
- FR-7 ✅: SQLite マイグレーション
- NFR-1 ✅: 後方互換性

---

## Key Findings / Decisions

1. **座標系修正は正しい**: `sin(heading)/cos(heading)` + headingOffset 補正
2. **Graph heading 計算は正しい**: atan2 → 方位角変換、FORWARD/BACKWARD 反転
3. **妥当性チェックロジックは正しい**: minHeadingTrips, maxHeadingStddev, maxHeadingOffset の各条件
4. **コンパイルエラーあり**: CRITICAL #1 によりコードがコンパイル不可能
5. **スレッド安全性に問題**: CRITICAL #2 により実行時例外リスク

---

## Artifacts
| Path | Type | Description |
|------|------|-------------|
| `code/docs/logs/impl/review/2026-07-26_REV_heading-calibration.md` | log | Full review log with detailed findings |

---

## Open Questions
1. PositionController の `endCalibrationTrip()` → `endTrip()` 呼び出しはデフォルト引数で動作するが、headingMean/headingStddev を渡すための Bridge API 変更は別途必要

---

## Routing
| Field | Value |
|-------|-------|
| **Next Agent** | ORC (escalation) → IMP (差し戻し) |
| **Blockers** | CRITICAL #1 (compile error), CRITICAL #2 (thread safety) |
| **Priority** | high |
| **Deadline** | — |
