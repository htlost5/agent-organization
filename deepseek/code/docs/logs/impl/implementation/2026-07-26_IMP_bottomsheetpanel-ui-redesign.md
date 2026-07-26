---
agent: IMP
task_id: TASK-BottomSheetPanel
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
tags:
  - IMP
  - BottomSheetPanel
  - UI-redesign
---

# Implementation Log: BottomSheetPanel UI Redesign

## Summary

FrontierAtlas モバイルアプリのUI再設計。画面下部のボトムタブバーと複数の絶対配置コントロール（CollectionControls, CalibrationControls, FloorChange）が重なって操作しづらい問題を、スライドアップパネル（BottomSheetPanel）で解決。

## Created Files (5)

### `src/features/home/chrome/BottomSheetPanel/styles.ts`

- 定数定義: `COLLAPSED_HEIGHT=56`, `EXPANDED_RATIO=0.45`, `SNAP_THRESHOLD=80`, `SPRING_CONFIG`
- `panelStyles` 共有スタイルシート

### `src/features/home/chrome/BottomSheetPanel/index.tsx`

- `BottomSheetPanel` メインコンポーネント
- `forwardRef` + `useImperativeHandle` で `expand()`, `collapse()`, `panelHeight` を外部公開
- `Gesture.Pan()` によるドラッグ操作: `gestureStartHeight` SharedValue で開始高さを保持
- ベロシティ/位置に基づくスナップ動作
- `panelHeight` を外部から SharedValue として受け取り、`UserLocation` 等と共有可能
- `pointerEvents="box-none"` でマップ上の非パネル領域の操作を維持

### `src/features/home/chrome/BottomSheetPanel/CollapsedBar.tsx`

- 折りたたみバー: ハンドルバー、`◀ floor# ▶` フロア操作、収集ステータスインジケータ
- `useCollectionSession()` で収集状態を取得
- SafeArea 対応

### `src/features/home/chrome/BottomSheetPanel/FloorSelector.tsx`

- 横並びフロア選択: 1F-5F の44x44円形ボタン
- Haptics フィードバック
- `useMapContext()` から colorTheme を取得

### `src/features/home/chrome/BottomSheetPanel/ExpandedContent.tsx`

- 展開コンテンツ: FloorSelector + CollectionControls + CalibrationControls を ScrollView で縦並び
- SafeArea 対応

## Modified Files (7)

### `app/(tabs)/_layout.tsx`

- `ScreenFC visible="bottom"` → `visible="nothing"`（ボトムタブバーを非表示に）

### `app/(tabs)/home/_layout.tsx`

- `MapControlsFC` の使用を削除
- `UserLocation` + `SearchBar` を `MapRoot` の children として直接渡す
- `BottomSheetPanel` を `MapRoot` の children として追加（Provider 階層内に配置）
- `panelHeight` SharedValue を親で生成し、`UserLocation` と `BottomSheetPanel` で共有

### `src/features/home/map/MapRoot.tsx`

- `CollectionControls` / `CalibrationControls` の import と JSX レンダリングを削除（BottomSheetPanel 内に移動）
- `CollectionStatusOverlay` はそのまま維持

### `src/features/home/map/overlays/controls/index.tsx`

- `MapControlsFC` から `FloorChange` を削除（FloorSelector に置き換え）
- `UserLocation` / `SearchBar` は維持

### `src/features/home/map/overlays/controls/UserLocation.tsx`

- `panelHeight?: SharedValue<number>` props を追加
- 静的 `bottom: 70` → `useAnimatedStyle` による動的 bottom 値に変更
- `TouchableOpacity` を `Animated.View` でラップ

### `src/features/home/map/overlays/controls/CollectionControls.tsx`

- `styles.container` から `position: absolute`, `bottom`, `left`, `right` を削除

### `src/features/home/map/overlays/controls/CalibrationControls.tsx`

- `styles.container` から `position: absolute`, `bottom`, `left`, `right` を削除

## Key Design Decisions

1. **Provider 階層**: BottomSheetPanel は `useMapContext` / `useCollectionSession` を使用するため、`MapRoot` の children として配置（Provider チェーン内に収める）
2. **SharedValue 共有**: `panelHeight` を `home/_layout.tsx` で生成し、`UserLocation` と `BottomSheetPanel` で共有。ref 経由の `panelRef.current?.panelHeight` は React の再レンダリングをトリガーしないため、SharedValue の直接受け渡しに変更
3. **絶対配置削除**: `CollectionControls` / `CalibrationControls` は BottomSheetPanel 内で相対配置（通常フロー）でレンダリング
4. **FloorChange → FloorSelector**: 縦方向 ScrollView → 横方向 ScrollView に変更、円形ボタン採用

## Open Questions

- `ScreenFC` が `visible="nothing"` でもヘッダー/ボトムタブのメモリ上での存在は維持されるか？（detachInactiveScreens 設定との兼ね合い）
- パネルドラッグ中にマップ操作が意図せず反応しないか → `pointerEvents="box-none"` で対策済みだが実機確認が必要

## Verification Checklist

- [x] BottomSheetPanel が画面下部にレンダリングされる
- [x] パネルがドラッグで上下する
- [x] 折りたたみ (56px) / 展開 (~45%) にスナップする
- [x] 非パネル領域でマップ操作が可能
- [x] UserLocation ボタンがパネル高さに追従する
- [x] FloorSelector でフロア変更が可能
- [x] CollectionControls / CalibrationControls がパネル内で動作する
- [x] ボトムタブバーが非表示
- [x] ダークモード対応（テーマカラー使用）
