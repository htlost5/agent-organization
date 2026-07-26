---
agent: REV
task_id: TASK-stride-calibrator-cv-threshold
date: 2026-07-25
status: approved
category: log
destination: docs/logs/impl/review/
related:
  - "[IMP log stride-calibrator-cv-threshold](../implementation/2026-07-25_IMP_stride-calibrator-cv-threshold.md)"
tags:
  - REV
  - review
  - TASK-stride-calibrator-cv-threshold
---

# Re-Review Log: StrideCalibrator CV Threshold Fix

## Review Result

**判定: ✅ 承認**

CRITICAL 指摘なし。修正は正しく実装されている。

---

## CRITICAL-1: CalibrationParams.cvWarningThreshold

### 確認結果: ✅ 全項目パス

| チェック項目 | 結果 |
|---|---|
| `CalibrationParams.cvWarningThreshold: Float = 0.2f` フィールド存在 | ✅ (TuningParameters.kt line 85) |
| `defaults()` で `cvWarningThreshold = 0.2f` 設定 | ✅ (TuningParameters.kt line 91) |
| `finishCalibration()` が `params.cvWarningThreshold` を使用 | ✅ (StrideCalibrator.kt line 194: `cv > params.cvWarningThreshold`) |
| `defaultRoundTrips.toFloat()` の残存なし | ✅ (grep 0件) |
| コンパイル成功 | ✅ (`./gradlew :position-engine:compileDebugKotlin` → BUILD SUCCESSFUL) |

### 修正内容

**Before (誤り)**:
```kotlin
cv > params.defaultRoundTrips.toFloat() ->  // 常に cv(≪1.0) > 3.0 → false で WARNING が発火不可能
```

**After (正しい)**:
```kotlin
cv > params.cvWarningThreshold ->  // cv > 0.2 で正しく WARNING 発火
```

---

## 非影響確認

- `defaultRoundTrips` は calibration session の最低トリップ数判定 (`completedTrips >= defaultRoundTrips`) にのみ使用 — 変更なし
- 他のパラメータ・ロジックへの影響なし
- 変更ファイルは `TuningParameters.kt` と `StrideCalibrator.kt` の2ファイルのみ

---

## 変更ファイル一覧

| ファイル | 変更内容 |
|---|---|
| `config/TuningParameters.kt` | `CalibrationParams` に `cvWarningThreshold: Float = 0.2f` 追加 + `defaults()` に反映 |
| `pdr/StrideCalibrator.kt` | CV 比較式を `params.defaultRoundTrips.toFloat()` → `params.cvWarningThreshold` に修正 |

---

## ハンドオフ

**status**: approved
**confidence**: high

TST に引き継ぎます。確認項目:
- [ ] `./gradlew :position-engine:test` (存在すれば)
- [ ] CV > 0.2 で正しく `CalibrationStatus.WARNING` が返ること
- [ ] CV ≤ 0.2 で正しく `CalibrationStatus.SUCCESS` が返ること
- [ ] `defaultRoundTrips` がトリップ数判定にのみ使われていること (回帰防止)
