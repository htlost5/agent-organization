---
agent: IMP
task_id: TASK-UNKNOWN
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
related:
  - "[NavGraphLoader.kt](../../../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/graph/NavGraphLoader.kt)"
  - "[useCollectionSession.ts](../../../../../../../frontieratlas/mobile/src/features/home/map/hooks/useCollectionSession.ts)"
tags:
  - IMP
  - implementation
  - nav-graph
  - path-fix
---

# Implementation Log: NavGraph Path Fix

## Summary

position-engine の `NavGraphLoader.kt` が期待する graph ファイルパスを mobile/assets の実際のアセット配置に合わせて修正。また `useCollectionSession.ts` の空文字 `assetsPath` に正しい値を設定した。

## Changes

### 1. `NavGraphLoader.kt` — JavaDoc コメント修正

- **変更 A**: Javadoc 内のパス記述を実際の構造に更新
  - 変更前: `assets/{assetsPath}/{buildingId}/graph/{floorId}_node.json`
  - 変更後: `{assetsPath}/{buildingId}/{floorId}_node.json`
  - 例コメントを追加: `maps/graph/routing/studyhall/1F_node.json`

- **変更 B**: メソッドコメント内の "graph/ subdirectory" → "building directory"

### 2. `NavGraphLoader.kt` — パス構築修正

- **変更 C**: 変数 `graphDirPath` から `/graph` を除去
  - 変更前: `val graphDirPath = "$assetsPath/$buildingId/graph"`
  - 変更後: `val graphDirPath = "$assetsPath/$buildingId"`

### 3. `useCollectionSession.ts` — assetsPath 設定

- **変更**: `assetsPath: ""` → `assetsPath: "maps/graph/routing"`

## Verification

- `assetsPath="maps/graph/routing"`, `buildingId="studyhall"` の場合:
  - 変更後: `graphDirPath` = `maps/graph/routing/studyhall` → 実際のアセットパスと一致 ✅
- `loadWithIndex()` は内部で `load()` を呼んでいるため自動的に対応済み ✅

## Files Modified

| File | Change |
|------|--------|
| `position-engine/.../NavGraphLoader.kt` | 3箇所修正（Javadoc×2, パス構文×1） |
| `mobile/.../useCollectionSession.ts` | 1箇所修正（assetsPath の値を設定） |

## Handoff

→ REV にハンドオフします。レビュー後、TST での検証をお願いします。
