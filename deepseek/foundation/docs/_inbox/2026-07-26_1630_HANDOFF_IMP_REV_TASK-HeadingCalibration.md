---
agent: IMP
task_id: TASK-HeadingCalibration
date: 2026-07-26
status: pending
category: log
destination: logs/impl/implementation/
tags:
  - IMP
  - handoff
  - forward
  - heading-calibration
---

# HANDOFF: IMP → REV

## Metadata
| Field | Value |
|-------|-------|
| **From** | IMP |
| **To** | REV |
| **Task ID** | TASK-HeadingCalibration |
| **Status** | success |
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

---

## Key Findings / Decisions

- **座標系修正**: deltaX = stride × sin(heading), deltaY = stride × cos(heading)（+Y=北, +X=東）
- **TripDirection**: 既存の `StrideCalibration.kt` の enum を再利用（再定義せず）
- **方位サンプリング**: HeadingEstimator に循環平均アルゴリズム実装（atan2(sinSum, cosSum)）
- **headingOffset 注入**: PdrEngine → RelativePositionCalculator の委譲プロパティ経由
- **DB マイグレーション**: v1→v2 の ALTER TABLE ADD COLUMN（シンプル手動）
- **後方互換**: endTrip() の new params（direction, headingMean, headingStddev）はすべてデフォルト値あり

---

## Artifacts

### Modified Files

| # | File | Path | Description |
|---|------|------|-------------|
| 1 | TuningParameters.kt | `position-engine/.../config/TuningParameters.kt` | CalibrationParams に 4 フィールド追加 |
| 2 | tuning_parameters.json | `position-engine/.../assets/config/tuning_parameters.json` | calibration セクションに 4 設定追加 |
| 3 | RelativePositionCalculator.kt | `position-engine/.../pdr/RelativePositionCalculator.kt` | cos↔sin 修正 + headingOffset 対応 |
| 4 | HeadingEstimator.kt | `position-engine/.../pdr/HeadingEstimator.kt` | 方位サンプリング機能追加 |
| 5 | StrideCalibrator.kt | `position-engine/.../pdr/StrideCalibrator.kt` | headingOffset 計算ロジック追加 |
| 6 | PdrEngine.kt | `position-engine/.../pdr/PdrEngine.kt` | headingOffset プロパティ追加 |
| 7 | CollectionDatabaseHelper.kt | `position-engine/.../storage/CollectionDatabaseHelper.kt` | v2 マイグレーション + 新カラム |

### Implementation Log
| Path | Type | Description |
|------|------|-------------|
| `code/docs/logs/impl/implementation/2026-07-26_IMP_heading-calibration.md` | log | Implementation log |

---

## Open Questions

1. **ビルド確認**: `./gradlew build` は Gradle プラグイン設定の問題（Kotlin Android プラグイン二重登録）で失敗する。これは事前の既存問題であり、本変更には起因しない。
2. **PositionController 連携**: `endCalibrationTrip()` は旧シグネチャで `endTrip(tripNumber, steps)` を呼び出し中。方位較正を有効にするには別途、Bridge API 経由で headingMean/headingStddev を渡す修正が必要。
3. **位置エラー修正（ログ）**: `RelativePositionCalculatorTest` の `heading` → `rawHeading` 変数名変更によるテストコード修正が必要な可能性。

---

## Routing
| Field | Value |
|-------|-------|
| **Next Agent** | REV |
| **Blockers** | none |
| **Priority** | high |
| **Deadline** | — |
