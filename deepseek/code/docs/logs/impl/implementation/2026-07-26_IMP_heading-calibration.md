---
agent: IMP
task_id: TASK-HeadingCalibration
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
tags:
  - IMP
  - heading-calibration
  - PDR
  - Kotlin
---

# Implementation Log: Heading Calibration

## Summary

PDR 歩幅較正に「方位オフセット較正」機能を追加。PDR 方位とグラフ Edge 方向のオフセットを較正する。

## Files Modified

### 1. `config/TuningParameters.kt` — CalibrationParams 拡張

`CalibrationParams` に以下 4 フィールドを追加:

| フィールド | 型 | デフォルト | 説明 |
|-----------|-----|----------|------|
| `enableHeadingCalibration` | Boolean | false | 方位較正の有効/無効 |
| `maxHeadingStddev` | Float | 15.0 | 方位標準偏差の最大許容値（度） |
| `maxHeadingOffset` | Float | 45.0 | 方位オフセットの最大許容値（度） |
| `minHeadingTrips` | Int | 2 | 方位較正に必要な最低トリップ数 |

### 2. `tuning_parameters.json` — 設定追加

`pdr.calibration` セクションに上記 4 フィールドを追加。

### 3. `pdr/RelativePositionCalculator.kt` — 座標系修正 + 方位オフセット対応

- `headingOffset: Float = 0f` プロパティを追加
- deltaX/deltaY の式を修正（`+Y=北, +X=東` 座標系に整合）:
  - `deltaX = stride × sin(heading)` （旧: `cos`）
  - `deltaY = stride × cos(heading)` （旧: `sin`）
- 方位オフセット補正を適用:
  ```kotlin
  val correctedHeading = ((rawHeading - headingOffset + 360f) % 360f)
  val correctedHeadingRad = Math.toRadians(correctedHeading.toDouble())
  val deltaX = stride * sin(correctedHeadingRad).toFloat()
  val deltaY = stride * cos(correctedHeadingRad).toFloat()
  ```

### 4. `pdr/HeadingEstimator.kt` — 方位サンプリング機能追加

- `headingSamples` / `isSampling` の内部状態追加
- `startHeadingSampling()`: サンプリング開始（バッファクリア）
- `stopHeadingSampling(): HeadingSampleResult`: サンプリング停止＋循環平均統計を返す
  - 循環平均（atan2(sinSum, cosSum)）：0°/360°境界対応
  - 循環標準偏差：差のラップ処理
- `processMagnetic()` 内で `if (isSampling) headingSamples.add(filteredHeading)` によりサンプル収集
- `HeadingSampleResult` data class: `mean`, `stddev`, `sampleCount`

### 5. `pdr/StrideCalibrator.kt` — 方位較正ロジック追加

- Import: `TripDirection`, `atan2` 追加
- `CalibrationResult` に `headingOffset: Float?`, `headingStddev: Float?` を追加
- 内部状態: `headingMeans`, `headingStddevs`, `tripDirections` リストを追加
- `startCalibration()`: 上記リストをクリア
- `endTrip()`: シグネチャ拡張（`direction`, `headingMean`, `headingStddev` パラメータ追加、デフォルト値あり）
- `finishCalibration()`: 方位オフセット計算を追加
  - `params.enableHeadingCalibration && headingMeans.isNotEmpty()` の場合のみ計算
  - `computeHeadingOffset()` 内部メソッド:
    - 最初の Edge の方向ベクトルから graph_heading を計算（`+Y=北=0°`, `+X=東=90°`）
    - FORWARD/BACKWARD で期待方位を反転
    - 各 trip の差分を計算（循環ラップ処理）
    - プール方式でオフセット平均・標準偏差を算出
    - 妥当性チェック: minHeadingTrips, maxHeadingStddev, maxHeadingOffset

### 6. `pdr/PdrEngine.kt` — headingOffset プロパティ追加

- `headingOffset` プロパティを追加（getter/setter で `calculator.headingOffset` に委譲）

### 7. `storage/CollectionDatabaseHelper.kt` — スキーママイグレーション

- `DATABASE_VERSION`: 1 → 2
- `onUpgrade()`: v1→v2 の ALTER TABLE マイグレーション追加:
  - `collection_stride_calibrations` に `heading_offset REAL` 追加
  - `collection_calibration_trips` に `heading_mean REAL`, `heading_stddev REAL` 追加
- CREATE TABLE 文にも新カラムを含める（新規インストール対応）

## Design Decisions

1. **座標系**: `+Y=北, +X=東` に統一。deltaX = stride × sin(heading), deltaY = stride × cos(heading)
2. **TripDirection**: 既存の `StrideCalibration.kt` の `TripDirection` enum を再利用（再定義せず）
3. **enableHeadingCalibration=false**: 既存動作と完全互換。headingOffset は常に 0f
4. **null safety**: headingMean/headingStddev が null の場合、既存の endTrip() 呼び出しはデフォルト値で動作

## Pending / Known Issues

- PositionController.kt の `endCalibrationTrip()` は旧シグネチャで呼び出し中（デフォルト値で動作）。方位サンプリングを有効にするには別途修正が必要（React Native Bridge API 変更含む）
- ビルド確認: `./gradlew build` は Gradle プラグイン設定の問題（Kotlin Android プラグイン二重登録）で失敗するが、これは事前の既存問題

## Handoff

REV へのハンドオフ準備完了。
