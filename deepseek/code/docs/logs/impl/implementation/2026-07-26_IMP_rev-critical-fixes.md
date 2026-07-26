---
agent: IMP
task_id: TASK-REV-CRITICAL-FIXES
date: 2026-07-26
status: done
category: log
destination: shared/impl/logs/implementation/
related:
  - "[PdrEngine.kt](../../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/pdr/PdrEngine.kt)"
  - "[PositionController.kt](../../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/controller/PositionController.kt)"
tags:
  - IMP
  - bugfix
  - stride-calibration
  - pdr
---

# Implementation Log: REV CRITICAL Fixes

## 修正概要

REV レビューで検出された CRITICAL 2件を修正した。

---

## CRITICAL-1: PdrEngine.kt — import/TAG 欠落

**ファイル**: `position-engine/.../pdr/PdrEngine.kt`

### 問題
- `enterCalibrationMode()`, `exitCalibrationMode()`, `resetStepCount()` で `Log.d(TAG, ...)` / `Log.w(TAG, ...)` を使用しているが:
  1. `import android.util.Log` が存在しない
  2. `private const val TAG = "PdrEngine"` が定義されていない
- → コンパイルエラー

### 修正
- **行 3**: `import android.util.Log` を追加
- **行 12**: `private const val TAG = "PdrEngine"` をクラス定義直前に追加

### 該当行
- Import追加: 行 3
- TAG定義追加: 行 12

---

## CRITICAL-2: PositionController.finishCalibration() — 妥当性チェックが設計書と矛盾

**ファイル**: `position-engine/.../controller/PositionController.kt`

### 問題
`finishCalibration()` 内の `effectiveStatus` 決定ロジックが設計書 `11b_stride-calibration.md` §6 と以下の点で矛盾していた:

| 条件 | 修正前（誤） | 設計書要求（正） | 変更 |
|------|-------------|----------------|------|
| `totalSteps == 0` | WARNING | **FAILED** | ✓ |
| `totalDistance < 10f` | WARNING | 削除（設計書は one_way_distance 基準、StrideCalibrator で既に検証済み） | ✓ |
| `stride < 0.3 \|\| stride > 1.5` | WARNING | **FAILED** | ✓ |
| `strideCV > 0.2` | WARNING | WARNING | 変更なし |

### 修正
- `totalSteps == 0` → `CalibrationStatus.FAILED`（デフォルト歩幅 0.7m 使用の意図）
- `totalDistance < 10f` チェックを**削除**（片道距離は `StrideCalibrator.startCalibration()` で検証済みのため重複）
- `stride < 0.3 || stride > 1.5` → `CalibrationStatus.FAILED`
- `strideCV > 0.2` → `CalibrationStatus.WARNING`（維持）
- コメントを設計書参照に更新

### 該当行
- 修正箇所: 行 550-567（`effectiveStatus` when ブロック）

### 動作への影響
- FAILED 条件に該当すると、行 563 の `if (effectiveStatus == CalibrationStatus.SUCCESS || effectiveStatus == CalibrationStatus.WARNING)` を通過しないため、`pdrEngine.strideEstimator.finishCalibration()` は呼ばれず、不正な較正結果が反映されない（正しい挙動）

---

## 確認事項

- [x] PdrEngine.kt: `import android.util.Log` 追加
- [x] PdrEngine.kt: `private const val TAG = "PdrEngine"` 定義追加
- [x] PositionController.kt: `totalSteps == 0` → FAILED
- [x] PositionController.kt: `totalDistance < 10f` チェック削除
- [x] PositionController.kt: `stride < 0.3 || stride > 1.5` → FAILED
- [x] PositionController.kt: `strideCV > 0.2` → WARNING（維持）
