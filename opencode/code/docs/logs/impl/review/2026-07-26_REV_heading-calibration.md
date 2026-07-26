---
agent: REV
task_id: TASK-HeadingCalibration
date: 2026-07-26
status: pending
category: log
destination: logs/impl/review/
tags:
  - REV
  - heading-calibration
  - review
  - CRITICAL
---

# Review Log: Heading Calibration

## Verdict: 🔴 CRITICAL — 差し戻し (Return to IMP)

## Verification Summary

| # | 要件 | ステータス | 詳細 |
|---|------|-----------|------|
| FR-1 | deltaX/deltaY 座標系修正 | ✅ | `sin(heading)/cos(heading)` 正しい。heading=0°→(0,+stride), heading=90°→(+stride,0) |
| FR-2 | CalibrationParams 4パラメータ | ✅ | 4フィールド追加、デフォルト値あり。JSON にも反映済み |
| FR-3 | 方位サンプリング | 🔴 | 循環平均アルゴリズムは正しいが、スレッド安全性に問題（CRITICAL #2） |
| FR-4 | StrideCalibrator 拡張 | ✅ | `CalibrationResult` 拡張、`endTrip()` デフォルト引数、後方互換性確保 |
| FR-5 | 妥当性チェック | 🔴 | チェックロジック自体は正しいが CRITICAL #1 のためコンパイル不可 |
| FR-6 | 補正適用 | ✅ | `rawHeading - headingOffset` 正しく補正、[0,360) 正規化済み |
| FR-7 | SQLite マイグレーション | ✅ | DATABASE_VERSION=2、onUpgrade+CREATE TABLE 整合 |
| NFR-1 | 後方互換性 | ✅ | デフォルト値・null 許容で既存呼び出し互換 |

## 🔴 CRITICAL Issues

### CRITICAL #1: `computeHeadingOffset()` に `return` 不足（コンパイルエラー）

**File**: `StrideCalibrator.kt` L310-330  
**Severity**: CRITICAL — コンパイル不可能

**問題**: `computeHeadingOffset()` の `when` 式の結果が関数から返されていない。

```kotlin
private fun computeHeadingOffset(): HeadingOffsetResult {
    // ...
    when {
        tripCount < params.minHeadingTrips -> {
            Log.w(TAG, ...)
            HeadingOffsetResult(0f, offsetStddev, tripCount)  // ← return なし
        }
        offsetStddev > params.maxHeadingStddev -> {
            Log.w(TAG, ...)
            HeadingOffsetResult(0f, offsetStddev, tripCount)  // ← return なし
        }
        abs(offsetMean) > params.maxHeadingOffset -> {
            Log.w(TAG, ...)
            HeadingOffsetResult(offsetMean, offsetStddev, tripCount)  // ← return なし
        }
        else -> {
            Log.d(TAG, ...)
            HeadingOffsetResult(offsetMean, offsetStddev, tripCount)  // ← return なし
        }
    }
}
```

Kotlin のブロックボディ関数では最後の式が自動的に戻り値にならないため、コンパイルエラーとなる。

**参考**: 同じ関数内の最初の分岐（`edgePath.isEmpty()`）は正しく `return` を使用している。

```kotlin
if (edgePath.isEmpty()) {
    Log.w(TAG, ...)
    return HeadingOffsetResult(0f, 0f, 0)  // ✅ 正しい
}
```

**修正案**:
```kotlin
return when {
    tripCount < params.minHeadingTrips -> {
        Log.w(TAG, ...)
        HeadingOffsetResult(0f, offsetStddev, tripCount)
    }
    // ...
}
```

### CRITICAL #2: `headingSamples` のスレッド安全性欠如（ConcurrentModificationException リスク）

**File**: `HeadingEstimator.kt` L40 (フィールド), L133 (add), L146 (clear), L158-181 (iterate)  
**Severity**: CRITICAL — 実行時 ConcurrentModificationException の可能性

**問題**: `headingSamples: MutableList<Float>` が同期機構なしで複数コルーチン/スレッドからアクセスされている。

| メソッド | 操作 | 呼び出し元 |
|---------|------|-----------|
| `processMagnetic()` L133 | `.add(filteredHeading)` | センサー収集コルーチン（PdrEngine） |
| `startHeadingSampling()` L146 | `.clear()` | 較正ロジック（例: PositionController → Bridge API） |
| `stopHeadingSampling()` L158-181 | `.sumOf {}` で反復 | 較正ロジック |

これらが同時に実行されると `ConcurrentModificationException` が発生する可能性がある。

また、`isSampling` フラグ（L41）が `@Volatile` または `AtomicBoolean` でないため、異なるスレッドからの変更可視性も保証されない。

**修正案**:
```kotlin
// Option A: synchronizedList の使用
private val headingSamples = Collections.synchronizedList(mutableListOf<Float>())

// Option B: stopHeadingSampling() でコピー作成
fun stopHeadingSampling(): HeadingSampleResult {
    isSampling = false
    val snapshot = synchronized(headingSamples) { headingSamples.toList() }
    // snapshot に対して計算
}
```

## 🟡 Minor Issues

### Issue 3: JSON `calibration` セクションに `cvWarningThreshold` なし
**File**: `tuning_parameters.json`  
`CalibrationParams` の `cvWarningThreshold`（デフォルト 0.2f）が JSON の `pdr.calibration` に含まれていない。`pdr.stride` セクションには存在する。デフォルト値と一致しているため動作問題はないが、JSON からの上書きが不可。

### Issue 4: 方位較正のテスト未実装
**File**: `StrideCalibratorTest.kt`  
`computeHeadingOffset()` の各妥当性チェック（minHeadingTrips、maxHeadingStddev、maxHeadingOffset）のテストケースが存在しない。CRITICAL #1 修正後に追加が必要。

## ✅ Confirmed Correct

### 座標変換
- `deltaX = stride × sin(heading)`, `deltaY = stride × cos(heading)` ✅（+Y=北, +X=東）
- heading=0°(北) → deltaX=0, deltaY=+stride ✅
- heading=90°(東) → deltaX=+stride, deltaY=0 ✅

### Graph heading 計算
- `atan2(directionY, directionX)` → `(90 - mathAngle + 360) % 360` ✅
- direction=(+Y, 0)=北 → graphHeading=0° ✅
- direction=(+X, 0)=東 → graphHeading=90° ✅

### FORWARD/BACKWARD 反転
- BACKWARD に +180° 加算 ✅

### 妥当性チェックロジック（コンパイルできれば正しい）
- minHeadingTrips > 条件 → WARNING, offset=0 ✅
- maxHeadingStddev > 条件 → WARNING, offset=0 ✅
- maxHeadingOffset > 条件 → WARNING（採用）✅
- enableHeadingCalibration=false → スキップ ✅

### SQLite マイグレーション
- `DATABASE_VERSION = 2` ✅
- v1→v2: ALTER TABLE 3行 ✅
- CREATE TABLE に新カラム含む ✅

### PdrEngine 委譲
- `headingOffset` getter/setter が `calculator.headingOffset` に委譲 ✅

## 結論

**CRITICAL 2件を検出。IMP に差し戻しが必要。**

| CRITICAL | 影響 | 修正難易度 |
|----------|------|-----------|
| #1: `return` 不足 | コンパイル不可 | 簡単（`return` 追加のみ） |
| #2: スレッド安全性 | 実行時例外リスク | 中（同期機構の導入） |

CRITICAL #1 はコンパイルを完全にブロックする。CRITICAL #2 は修正が推奨されるが、条件次第では後続タスクでも対応可能。
