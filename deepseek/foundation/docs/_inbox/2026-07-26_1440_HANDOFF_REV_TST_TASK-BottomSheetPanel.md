---
agent: REV
task_id: TASK-BottomSheetPanel
date: 2026-07-26
status: pending
category: log
destination: logs/impl/
tags:
  - REV
  - handoff
  - forward
  - UI-redesign
---

# HANDOFF: REV → TST

## Metadata
| Field | Value |
|-------|-------|
| **From** | REV |
| **To** | TST |
| **Task ID** | TASK-BottomSheetPanel |
| **Status** | conditional_approval |
| **Confidence** | high |
| **Handoff Type** | forward |

## Task Context (継承)

> ORC → DEV → IMP → REV の継承コンテキスト。

### Original Request
FrontierAtlas モバイルアプリ（React Native + Expo）のUI再設計。画面下部のボトムタブバーと複数の絶対配置コントロール（CollectionControls, CalibrationControls, FloorChange, UserLocation）が重なって操作しづらい問題を、スライドアップパネル（BottomSheetPanel）で解決する。

### Constraints
- react-native-reanimated 4.5.0
- react-native-gesture-handler ~2.32.0
- react-native-safe-area-context ~5.7.0
- expo-router (file-based routing with <Tabs>)
- 全新規コンポーネントは明示的な Props インターフェースを持つこと
- 既存コードの構造を極力維持すること

### Chain History
| Step | Agent | Status | Summary |
|------|-------|--------|---------|
| 1 | DEV | done | 設計仕様書作成 |
| 2 | IMP | done | コード実装完了 (5新規作成 + 7修正) |
| 3 | REV | conditional_approval | レビュー完了 (条件付き承認) |

## Key Findings / Decisions

### Implementation Decisions (from IMP)
1. **Provider 階層**: BottomSheetPanel は MapRoot の children として配置（Provider チェーン内）
2. **SharedValue 共有**: panelHeight を home/_layout.tsx で生成し、UserLocation と BottomSheetPanel に props で直接渡す
3. **FloorChange → FloorSelector**: 縦方向 → 横方向 ScrollView、円形44x44ボタン
4. **CollectionControls / CalibrationControls**: absolute positioning 削除、通常フロー

### Review Findings (from REV)
1. **Unused import** (Condition): CollapsedBar.tsx に useMapContext の未使用 import あり。TST 前に修正推奨。
2. **activeOffsetY 未設定**: Pan gesture に activeOffsetY がない。誤操作防止のため追加推奨。
3. **Gesture conflict potential**: ScrollView と Pan gesture の競合可能性。実機確認推奨。

### Technical Notes
- `Gesture.Pan()` でドラッグ、スナップ判定 (velocity > 500/-500, threshold 80px)
- `pointerEvents="box-none"` でマップ操作維持
- 全コンポーネントは colorTheme (useMapContext) でダークモード対応

## Artifacts

### New Files
| Path | Description |
|------|-------------|
| mobile/src/features/home/chrome/BottomSheetPanel/styles.ts | 定数・共有スタイル |
| mobile/src/features/home/chrome/BottomSheetPanel/index.tsx | メインパネル (gesture, snap, animation) |
| mobile/src/features/home/chrome/BottomSheetPanel/CollapsedBar.tsx | 折りたたみバー |
| mobile/src/features/home/chrome/BottomSheetPanel/FloorSelector.tsx | 横並びフロア選択 |
| mobile/src/features/home/chrome/BottomSheetPanel/ExpandedContent.tsx | 展開コンテンツ |

### Modified Files
| Path | Change |
|------|--------|
| app/(tabs)/_layout.tsx | ScreenFC visible → "nothing" |
| app/(tabs)/home/_layout.tsx | MapControlsFC削除, BottomSheetPanel追加 |
| src/features/home/map/MapRoot.tsx | CollectionControls/CalibrationControls削除 |
| src/features/home/map/overlays/controls/index.tsx | FloorChange削除 |
| src/features/home/map/overlays/controls/UserLocation.tsx | panelHeight prop追加 |
| src/features/home/map/overlays/controls/CollectionControls.tsx | absolute positioning削除 |
| src/features/home/map/overlays/controls/CalibrationControls.tsx | absolute positioning削除 |

## Review Log
- [Review Log](../code/docs/logs/impl/review/2026-07-26_REV_bottomsheetpanel-ui-redesign.md)

## Open Questions
- ScreenFC visible="nothing" 時のメモリフットプリント
- パネルドラッグ中のマップ誤操作（pointerEvents対策済みだが実機確認必要）
- ScrollView 内でのジェスチャ競合（実機確認必要）

## TST Focus Areas
1. **基本動作**: パネルドラッグで Collapsed(56px) / Expanded(45%) にスナップするか
2. **フロア切替**: CollapsedBar の prev/next、FloorSelector の各ボタンが正しく動作するか
3. **収集機能**: CollectionControls がパネル内で正しく表示・動作するか
4. **較正機能**: CalibrationControls がパネル内で正しく表示・動作するか
5. **UserLocation 追従**: UserLocation の bottom 位置がパネル高さ + 10px で追従するか
6. **マップ操作**: パネル外領域でのマップ操作（パン・ズーム）が可能か
7. **ボトムタブ**: visible="nothing" でボトムタブバーが非表示になっているか
8. **ダークモード**: テーマ切替で配色が正しく反映されるか
9. **TypeScript**: npx tsc --noEmit が 0 errors（確認済み）
10. **Lint**: 1 new warning (CollapsedBar unused import) + pre-existing warnings（条件）

## Routing
| Field | Value |
|-------|-------|
| **Next Agent** | TST |
| **Blockers** | none |
| **Priority** | high |
| **Deadline** | — |
