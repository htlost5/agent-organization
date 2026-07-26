---
agent: IMP
task_id: TASK-BottomSheetPanel
date: 2026-07-26
status: pending
category: log
destination: logs/impl/
related:
  - "[Implementation Log](../code/docs/logs/impl/implementation/2026-07-26_IMP_bottomsheetpanel-ui-redesign.md)"
tags:
  - IMP
  - handoff
  - forward
  - UI-redesign
---

# HANDOFF: IMP → REV

## Metadata

| Field            | Value                 |
| ---------------- | --------------------- |
| **From**         | IMP                   |
| **To**           | REV                   |
| **Task ID**      | TASK-BottomSheetPanel |
| **Status**       | success               |
| **Confidence**   | high                  |
| **Handoff Type** | forward               |

---

## Task Context（継承・追記セクション）

> ORC 初回指示からの継承コンテキスト。IMP が成果を追記。

### Original Request

FrontierAtlas モバイルアプリ（React Native + Expo）のUI再設計。画面下部のボトムタブバーと複数の絶対配置コントロール（CollectionControls, CalibrationControls, FloorChange, UserLocation）が重なって操作しづらい問題を、スライドアップパネル（BottomSheetPanel）で解決する。

### Constraints

- `react-native-reanimated` 4.5.0
- `react-native-gesture-handler` ~2.32.0
- `react-native-safe-area-context` ~5.7.0
- expo-router (file-based routing with `<Tabs>`)
- 全新規コンポーネントは明示的な Props インターフェースを持つこと
- 既存コードの構造を極力維持すること

### Chain History

| Step | Agent | Status | Summary                                              |
| ---- | ----- | ------ | ---------------------------------------------------- |
| 1    | DEV   | done   | 設計仕様書作成 (BottomSheetPanel 導入によるUI再設計) |
| 2    | IMP   | done   | コード実装完了 (5新規作成 + 7修正)                   |

---

## Key Findings / Decisions

### Implementation Decisions (from Plan)

1. **Provider 階層対応**: BottomSheetPanel は `useMapContext` / `useCollectionSession` を使用するため、`MapRoot` の children として配置（Provider チェーン内に収める）。Plan では MapRoot の外だったが、Provider アクセス不能のため内部配置に変更。
2. **SharedValue 共有方式**: `panelHeight` を `home/_layout.tsx` で `useSharedValue` 生成し、`UserLocation` と `BottomSheetPanel` に props で直接渡す方式に変更。Plan の `panelRef.current?.panelHeight` は React 再レンダリングをトリガーしないため不採用。
3. **FloorChange → FloorSelector**: 縦方向 ScrollView → 横方向 ScrollView に変更。円形44x44ボタン採用。
4. **CollectionControls / CalibrationControls**: `position: absolute` + `bottom/left/right` を削除し、通常フロー（ブロック要素）としてレンダリング。

### Technical Notes

- `Gesture.Pan()` は `gestureStartHeight` SharedValue で開始高さを保持し、`event.translationY`（累積値）で現在高さを計算
- スナップ判定: velocity > 500 → collapse, velocity < -500 → expand, position threshold 80px
- `pointerEvents="box-none"` で非パネル領域のマップ操作を維持
- `CollapsedBar` は内側で `useCollectionSession()` を使用（Provider チェーン内のため問題なし）

---

## Artifacts

| Path                                                                     | Type     | Description                                |
| ------------------------------------------------------------------------ | -------- | ------------------------------------------ |
| `mobile/src/features/home/chrome/BottomSheetPanel/styles.ts`             | new      | 定数・共有スタイル定義                     |
| `mobile/src/features/home/chrome/BottomSheetPanel/index.tsx`             | new      | メインパネル (gesture, snap, animation)    |
| `mobile/src/features/home/chrome/BottomSheetPanel/CollapsedBar.tsx`      | new      | 折りたたみバー                             |
| `mobile/src/features/home/chrome/BottomSheetPanel/FloorSelector.tsx`     | new      | 横並びフロア選択                           |
| `mobile/src/features/home/chrome/BottomSheetPanel/ExpandedContent.tsx`   | new      | 展開コンテンツ                             |
| `mobile/app/(tabs)/_layout.tsx`                                          | modified | ScreenFC visible → "nothing"               |
| `mobile/app/(tabs)/home/_layout.tsx`                                     | modified | MapControlsFC削除, BottomSheetPanel追加    |
| `mobile/src/features/home/map/MapRoot.tsx`                               | modified | CollectionControls/CalibrationControls削除 |
| `mobile/src/features/home/map/overlays/controls/index.tsx`               | modified | FloorChange削除                            |
| `mobile/src/features/home/map/overlays/controls/UserLocation.tsx`        | modified | panelHeight prop追加, animated bottom      |
| `mobile/src/features/home/map/overlays/controls/CollectionControls.tsx`  | modified | absolute positioning削除                   |
| `mobile/src/features/home/map/overlays/controls/CalibrationControls.tsx` | modified | absolute positioning削除                   |

---

## Open Questions

- `ScreenFC` が `visible="nothing"` でもヘッダー/ボトムタブはメモリ上に残るか？（パフォーマンス影響）
- BottomSheetPanel ドラッグ中に裏のマップが誤操作されないか → `pointerEvents="box-none"` で対策済みだが実機確認推奨

---

## Routing

| Field          | Value |
| -------------- | ----- |
| **Next Agent** | REV   |
| **Blockers**   | none  |
| **Priority**   | high  |
| **Deadline**   | —     |

---

## ORC Approval（ORC が最終確認時に記入）

- [ ] Approved — proceed to REV
- [ ] Re-routed to: \_\_\_
- Notes: \_\_\_
