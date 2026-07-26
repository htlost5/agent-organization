---
agent: IMP
task_id: TASK-stride-calibrator-cv-fix
date: 2026-07-25
status: draft
category: log
destination: docs/logs/impl/implementation/
related:
  - "[StrideCalibrator](../../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/pdr/StrideCalibrator.kt)"
  - "[TuningParameters](../../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/config/TuningParameters.kt)"
tags:
  - IMP
  - implementation
  - stride-calibrator
  - cv-threshold
  - pdr
  - bugfix
---

# Implementation Log: StrideCalibrator CV Threshold Fix

## Problem

`StrideCalibrator.finishCalibration()` used `params.defaultRoundTrips.toFloat()` (= 3.0) as the CV threshold for WARNING status. Since CV (coefficient of variation) rarely exceeds 3.0 in practice, the WARNING status was never triggered.

The correct threshold should be `0.2f` (from `StrideParams.cvWarningThreshold`).

## Changes Made

### 1. `TuningParameters.kt` — `CalibrationParams` に `cvWarningThreshold` 追加

```kotlin
@Serializable
data class CalibrationParams(
    val minDistanceMeters: Float,
    val defaultRoundTrips: Int,
    val edgeParallelismThreshold: Float,
    val cvWarningThreshold: Float = 0.2f   // ← ADDED
) {
    companion object {
        fun defaults(): CalibrationParams = CalibrationParams(
            minDistanceMeters = 10.0f,
            defaultRoundTrips = 3,
            edgeParallelismThreshold = 0.996f,
            cvWarningThreshold = 0.2f       // ← ADDED
        )
    }
}
```

### 2. `StrideCalibrator.kt` — `finishCalibration()` の CV 閾値を修正

```kotlin
// BEFORE (bug): cv > params.defaultRoundTrips.toFloat()
// AFTER  (fix): cv > params.cvWarningThreshold
cv > params.cvWarningThreshold -> {
    CalibrationStatus.WARNING
}
```

## Verification

- `./gradlew :position-engine:compileDebugKotlin` → **BUILD SUCCESSFUL** (6s)
- CV > 0.2 で WARNING が正しく発行されるようになった（以前は CV > 3.0 が必要で実質的に発動不可だった）

## Handoff to REV

**status**: 成功
**key_findings_or_decisions**: CV 閾値を `defaultRoundTrips.toFloat()` (3.0) から `cvWarningThreshold` (0.2) に修正。これにより CV > 0.2 で WARNING ステータスが正しく発行される。
**artifacts**:
- `TuningParameters.kt`: `CalibrationParams` に `cvWarningThreshold` フィールド追加
- `StrideCalibrator.kt`: `finishCalibration()` の CV 閾値修正
**open_questions**: なし
**next_actions**: REV による再レビュー
