---
agent: REV
task_id: TASK-HeadingCalibration
date: 2026-07-26
status: pending
category: log
destination: logs/impl/review/
tags:
  - REV
  - TST
  - handoff
  - forward
  - heading-calibration
---

# HANDOFF: REV → TST

## Metadata

| Field | Value |
|-------|-------|
| **From** | REV |
| **To** | TST |
| **Task ID** | TASK-HeadingCalibration |
| **Status** | approved |
| **Confidence** | high |
| **Handoff Type** | forward |

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
| 2 | REV (1st) | critical | CRITICAL #1: return 不足, CRITICAL #2: スレッド安全性 |
| 3 | IMP (fix) | done | 2ファイル修正 (StrideCalibrator.kt, HeadingEstimator.kt) |
| 4 | **REV (2nd)** | **approved** | **両 CRITICAL 修正確認済み → TST へ** |

---

## Fix Verification Summary

| CRITICAL # | Description | Status |
|------------|-------------|--------|
| #1 | `computeHeadingOffset()` の `return when { ... }` | ✅ Fixed |
| #2 | `headingSamples` の `synchronized(samplesLock)` 保護 | ✅ Fixed |

## Files Modified (by IMP)

1. `position-engine/.../pdr/StrideCalibrator.kt` — `return when { ... }` 追加
2. `position-engine/.../pdr/HeadingEstimator.kt` — `samplesLock` 導入、各アクセスを synchronized 化

## Artifacts

- [Re-review Log](../code/docs/logs/impl/review/2026-07-26_REV_heading-calibration-fix-review.md)
- [Previous Review Log](../code/docs/logs/impl/review/2026-07-26_REV_heading-calibration.md)

## Open Questions

なし — 前回の Open Question（Bridge API 変更）は本タスクスコープ外。

## Routing

| Field | Value |
|-------|-------|
| **Next Agent** | TST |
| **Blockers** | none |
| **Priority** | high |
| **Deadline** | — |
