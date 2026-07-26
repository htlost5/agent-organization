---
agent: REV
task_id: TASK-REV-CRITICAL-FIXES
date: 2026-07-26
status: approved
category: log
destination: code/docs/logs/impl/review/
related:
  - "[IMP critical fixes log](../implementation/2026-07-26_IMP_rev-critical-fixes.md)"
  - "[PdrEngine.kt](../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/pdr/PdrEngine.kt)"
  - "[PositionController.kt](../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/controller/PositionController.kt)"
  - "[Design spec §6](../../../../../../frontieratlas/docs/architect/11b_stride-calibration.md)"
tags:
  - REV
  - review
  - re-review
  - TASK-REV-CRITICAL-FIXES
  - stride-calibration
---

# Re-Review Log: CRITICAL Fixes Verification

## Review Result

**判定: ✅ 承認**

両 CRITICAL 修正は正しく実装されている。コンパイルエラーは解消され、妥当性チェックロジックは設計書の要件（および修正指示）に準拠している。

---

## CRITICAL-1: PdrEngine.kt — import/TAG 追加

### 確認結果: ✅ OK

| チェック項目 | 結果 | 該当行 |
|---|---|---|
| `import android.util.Log` 存在 | ✅ | line 3 |
| `private const val TAG = "PdrEngine"` 定義 | ✅ | line 13 |
| `Log.d(TAG, ...)` 使用（enterCalibrationMode） | ✅ | line 119 |
| `Log.w(TAG, ...)` 使用（exitCalibrationMode 失敗） | ✅ | line 129 |
| `Log.d(TAG, ...)` 使用（exitCalibrationMode） | ✅ | line 135 |
| `Log.d(TAG, ...)` 使用（resetStepCount） | ✅ | line 143 |

**判定**: コンパイルエラーは完全に解消されている。

---

## CRITICAL-2: PositionController.finishCalibration() — 妥当性チェック修正

### 確認結果: ✅ OK

**修正前後の比較（設計書 §6 対 実装）**:

| 条件 | 設計書 §6 | 修正前（誤） | 修正後（正） | 結果 |
|---|---|---|---|---|
| `totalSteps == 0` | FAILED | WARNING | **FAILED** | ✅ |
| `totalDistance < 10f` | one_way_distance 基準（設計書） | WARNING | **削除**（重複検証のため） | ✅ |
| `stride < 0.3 \|\| stride > 1.5` | FAILED | WARNING | **FAILED** | ✅ |
| `strideCV > 0.2` | WARNING | WARNING | WARNING（維持） | ✅ |
| ELSE | SUCCESS | — | `calResult.status` に委譲 | ✅ |

**該当コード（lines 551-567）**:
```kotlin
val effectiveStatus = when {
    calResult.totalSteps == 0 -> {
        Log.w(TAG, "Calibration failed: zero steps recorded")
        CalibrationStatus.FAILED
    }
    calResult.calibratedStride != null &&
        (calResult.calibratedStride < 0.3f || calResult.calibratedStride > 1.5f) -> {
        Log.w(TAG, "Calibration failed: stride ${calResult.calibratedStride}m out of range [0.3, 1.5]")
        CalibrationStatus.FAILED
    }
    calResult.strideCv != null && calResult.strideCv > 0.2f -> {
        Log.w(TAG, "Calibration warning: CV ${calResult.strideCv} > 0.2")
        CalibrationStatus.WARNING
    }
    else -> calResult.status
}
```

**補足**: `totalDistance < 10f` チェック削除は妥当。片道距離の検証は `StrideCalibrator.startCalibration()` で既に実施済みのため、`finishCalibration()` での重複チェックは不要。IMP の判断は正しい。

---

## WARNING 引継ぎ

**前回レビューの WARNING 項目（7件）**: 保存されたレビューログに該当する7件の WARNING 項目を確認できなかった。code/docs/logs/impl/review/ 配下の全3ファイルを調査したが、いずれも本改修に関連する7件の WARNING 項目を含んでいない。

本再レビューにおいて新たに検出された WARNING 項目は以下の1件:

### W-1: 設計書と実装の乖離

- **内容**: `finishCalibration()` のコメントに「設計書 11b_stride-calibration.md §6 に準拠」とあるが、§6 の `one_way_distance < 10.0m` → FAILED チェックが実装から削除されている
- **影響**: 削除は妥当な判断だが、設計書が更新されない場合、将来のメンテナンス時に混乱を招く可能性がある
- **推奨**: 設計書 `11b_stride-calibration.md` §6 から `one_way_distance < 10.0m` チェックを削除し、実装と同期する

---

## 変更ファイル一覧

| ファイル | 変更内容 |
|---|---|
| `pdr/PdrEngine.kt` | `import android.util.Log` 追加, `TAG` 定数追加 |
| `controller/PositionController.kt` | `finishCalibration()` の妥当性チェックを設計書 §6 準拠に修正 + `totalDistance` チェック削除 |

---

## ハンドオフ

**status**: approved
**confidence**: high

**TST への引き継ぎ確認項目**:
- [ ] `./gradlew :position-engine:compileDebugKotlin` — BUILD SUCCESSFUL
- [ ] `totalSteps == 0` 時に `CalibrationStatus.FAILED` が返ること
- [ ] `stride < 0.3 || stride > 1.5` 時に `CalibrationStatus.FAILED` が返ること
- [ ] `strideCV > 0.2` 時に `CalibrationStatus.WARNING` が返ること
- [ ] 全条件パス時に `CalibrationStatus.SUCCESS` が返ること
- [ ] FAILED 時に `StrideEstimator.finishCalibration()` が呼ばれないこと（回帰防止）

