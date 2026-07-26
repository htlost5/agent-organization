---
agent: IMP
task_id: TASK-Phase1-Completion
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
related:
  - "[PositionController](../../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/controller/PositionController.kt)"
  - "[PdrEngine](../../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/pdr/PdrEngine.kt)"
  - "[PositionEngineModule](../../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/bridge/PositionEngineModule.kt)"
tags:
  - IMP
  - phase1
  - calibration
  - permissions
  - fallback
---

# Phase 1 完成実装ログ

## 概要

屋内測位システム Phase 1 の残存タスクを一括実装。全6項目（Item 1〜6）を P0→P1→P2 の優先順位で完了。

---

## 修正ファイル一覧

| # | ファイル | 変更種別 | 概要 |
|---|---------|---------|------|
| 5-1 | `mobile/android/app/src/main/AndroidManifest.xml` | 編集 | パーミッション9種追加 |
| 5-2 | `mobile/app.config.ts` | 編集 | `newArchEnabled: true` 追加 |
| 3-1 | `position-engine/.../event/CollectionEventEngine.kt` | 編集 | `UserStop` sealed class + `fireUserStop()` 追加 |
| 3-2 | `position-engine/.../controller/PositionController.kt` | 編集 | `stopCollection()` で `fireUserStop` 発火 |
| 3-3 | `position-engine/.../bridge/PositionEngineModule.kt` | 編集 | `serializeEvent()` に `UserStop` 分岐追加 |
| 4-1 | `position-engine/.../pdr/PdrStateManager.kt` | 編集 | `resetAccumulatedError()` 追加 |
| 4-2 | `position-engine/.../pdr/PdrEngine.kt` | 編集 | `resetAccumulatedError()` 委譲追加 |
| 4-3 | `position-engine/.../controller/PositionController.kt` | 編集 | `resume()` 内で `resetAccumulatedError()` 呼出 |
| 6-1 | `position-engine/.../collection/CollectionRecorder.kt` | 編集 | DB書き込み失敗時に `fireError` 追加 |
| 6-2 | `position-engine/.../sensor/WifiScanner.kt` | 編集 | `eventEngine` 参照 + スキャン失敗時エラー + 5秒クールダウン |
| 6-3 | `position-engine/.../sensor/SensorController.kt` | 編集 | `setEventEngine()` メソッド追加 |
| 6-4 | `position-engine/.../bridge/PositionEngineModule.kt` | 編集 | `sensorController.setEventEngine()` 呼出 |
| 2-1 | `position-engine/.../common/CalibrationReadyResult.kt` | **新規** | `@Serializable` data class |
| 2-1 | `position-engine/.../common/TripStarted.kt` | **新規** | `@Serializable` data class |
| 2-1 | `position-engine/.../common/TripResult.kt` | **新規** | `@Serializable` data class |
| 2-2 | `position-engine/.../controller/PositionController.kt` | 編集 | 較正API Stub→実装 + コンストラクタ拡張 + Dijkstra経路探索 |
| 2-3 | `position-engine/.../pdr/PdrEngine.kt` | 編集 | `enterCalibrationMode()` / `exitCalibrationMode()` / `resetStepCount()` / `getStepCount()` |
| 2-4 | `position-engine/.../bridge/PositionEngineModule.kt` | 編集 | 較正メソッド実機呼出 + シリアライズ |
| — | `position-engine/.../storage/repository/CalibrationRepository.kt` | 編集 | `updateCalibrationResult()` 追加 |

---

## 各Itemの詳細

### Item 1: assetsPath修正 (mobile) — 確認済み

`useCollectionSession.ts` の `startSession()` 内 `assetsPath` は既に `"maps/graph/routing"` に設定済み。**修正不要**。

### Item 5 (P0): パーミッション拡充・newArchEnabled

**AndroidManifest.xml** に以下を追加:
- `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`
- `ACCESS_WIFI_STATE`, `CHANGE_WIFI_STATE`
- `ACCESS_NETWORK_STATE`
- `ACTIVITY_RECOGNITION`
- `HIGH_SAMPLING_RATE_SENSORS`
- `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_LOCATION`

**app.config.ts** の `expo-build-properties` に `newArchEnabled: true` を追加。

### Item 3 (P1): UserStop イベント追加

- `CollectionEvent.UserStop(sessionId, reason)` を sealed class のサブクラスとして追加
- `CollectionEventEngine.fireUserStop(sessionId, reason)` メソッド追加
- `PositionController.stopCollection()` の先頭で `eventEngine.fireUserStop(session.sessionId, "user_requested")` を呼出
- `PositionEngineModule.serializeEvent()` に `is CollectionEvent.UserStop` 分岐追加

### Item 4 (P1): PDR再開時リセット

- `PdrStateManager.resetAccumulatedError()`: `totalX/totalY` のみゼロリセット（`stepCount/heading/currentStride` は保持）
- `PdrEngine.resetAccumulatedError()`: `stateManager` に委譲
- `PositionController.resume()`: PDR再開時に `pdrEngine.resetAccumulatedError()` を `runBlocking` で呼出

### Item 6 (P1): フォールバック改善

- `CollectionRecorder`: WiFi/Magnetic/Pressure/Motion/PDR 各INSERT失敗時に `eventEngine.fireError(sessionId, code, message, recoverable=true)` を追加
- `WifiScanner`: `CollectionEventEngine` 参照を追加。スキャン失敗時（`startScan()`戻り値false / SCAN_RESULTS_AVAILABLEのsuccess=false）に5秒クールダウン付きで `fireError` 発火
- `SensorController`: `setEventEngine()` メソッド追加 → WifiScanner に転送
- `PositionEngineModule.initialize()`: `sensorController.setEventEngine(eventEng)` 呼出

### Item 2 (P2): 較正API接続（Stub→実装）

#### 新規モデル（3ファイル）
- `CalibrationReadyResult(calibrationId, oneWayDistance, totalTrips)` @Serializable
- `TripStarted(tripNumber, direction)` @Serializable — TripDirectionは既存enum再利用
- `TripResult(tripNumber, steps, distance, tripStride, durationMs)` @Serializable

#### PositionController 改修
- コンストラクタに `graphMap: Map<String, NavigationGraph>` と `calibrationRepo: CalibrationRepository` を追加
- 内部状態: `calibrator: StrideCalibrator?`, `currentCalibrationId`, `currentTripNumber`, `tripStartTime`, `currentDirection`
- `startStrideCalibration(sessionId, startNodeId, endNodeId, roundTrips)`: Dijkstra最短経路探索 → StrideCalibrator生成 → PdrEngine.enterCalibrationMode() → DB INSERT → CalibrationReadyResult返却
- `startCalibrationTrip(calibrationId, tripNumber)`: resetStepCount() → 開始時刻記録 → 方向決定（偶数FORWARD/奇数BACKWARD）→ DB UPDATE → TripStarted返却
- `endCalibrationTrip(calibrationId, tripNumber)`: getStepCount() → StrideCalibrator.endTrip() → DB INSERT CalibrationTrip → TripResult返却
- `finishCalibration(calibrationId)`: finishCalibration() → 妥当性チェック（歩数0/距離10m未満/歩幅0.3-1.5外/CV>0.2→WARNING）→ StrideEstimator反映 → DB UPDATE → DB再読込してStrideCalibration返却
- `getCalibrationStatus(calibrationId)`: DBから取得
- `findShortestPath()`: Dijkstra法で2ノード間の最短経路を探索（無向グラフ対応）

#### PdrEngine 改修
- `enterCalibrationMode()`: RelativePositionCalculator停止、StepDetector継続
- `exitCalibrationMode()`: 通常モード復帰（calculator再起動）
- `resetStepCount()`: 歩数カウンタリセット
- `getStepCount()`: 現在歩数取得

#### PositionEngineModule 改修
- PositionController 生成時に `graphMap` と `calibrationRepo` を注入
- `startStrideCalibration`, `startCalibrationTrip`, `endCalibrationTrip`, `finishCalibration`, `getCalibrationStatus` を非同期実装に更新（戻り値JSONシリアライズ）

#### CalibrationRepository 拡張
- `updateCalibrationResult()` 追加: calibratedStride/strideCv/totalDistance/totalSteps/status/fallbackStride を一括更新

---

## 懸念点・制約事項

1. **PositionController の resume() 内 runBlocking**: 非suspend関数からsuspend関数（resetAccumulatedError）を呼ぶために `runBlocking` を使用。コルーチンコンテキストが Dispatchers.Default のため、軽量な操作（Mutexロック + 状態コピーのみ）でありブロッキング影響は限定的だが、将来のリファクタリング候補。

2. **findShortestPath() の効率**: シンプルなDijkstra実装（LinkedList + O(n)最小値探索）。ノード数が大きい場合（1000+）は PriorityQueue への最適化を推奨。

3. **TuningParameters.defaults().pdr.calibration**: 静的なデフォルト値を使用。運用時は `TuningParameterLoader` で外部ファイルからロードされたパラメータを使用する想定。

4. **CalibrationTrip.distance 計算**: `tripStride * steps` で oneWayDistance 相当になる。方向によってわずかに異なる歩幅を各トリップで独立記録。

5. **WifiScanner の空文字 sessionId**: スキャン失敗時は sessionId がないため空文字で発火。上位レイヤでフィルタリングまたは補完が必要。

6. **UserStop は既存の SessionCompleted と競合しない**: UserStop は `stopCollection()` 先頭で発火 → その後 `sessionManager.stopCollection()` 内で SessionCompleted が発火される。
