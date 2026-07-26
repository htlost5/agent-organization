---
agent: IMP
task_id: TASK-000
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
tags:
  - IMP
  - implementation
  - geojson
  - manifest
---

# Implementation Log: Fix `generate_geojsonAssetMap.js` — マニフェストパス修正

## Summary

`npm run data:load` がエラーになる原因を修正。zip 解凍後のデータ構造変更（単一マニフェスト → 2マニフェスト分割）に対応するため、`scripts/generate_geojsonAssetMap.js` を修正した。

## Changes

### 1. マニフェスト読取パスの変更

**Before**: `assets/maps/manifest.json` のみ読み取り

**After**: `assets/maps/features/manifest.json` + `assets/maps/graph/manifest.json` の両方を読み取り

- `manifestPath` → `featuresManifestPath`, `graphManifestPath` の2変数に分割

### 2. 新規関数 `processManifestEntries()` の追加

各マニフェストのエントリを処理し、適切なプレフィックスを付与する関数を新設。

- `features` のエントリ → `features/${relPath}` (import も `@/assets/maps/features/...`)
- `graph` のエントリ → `graph/${relPath}` (import も `@/assets/maps/graph/...`)

### 3. `generate()` 関数のインターフェース変更

**Before**: `generate(manifest)` — マニフェストオブジェクト全体を受け取る

**After**: `generate(allEntries)` — 前処理済みエントリ配列を受け取る

各エントリは `{ logicalId, relativePath, importPath }` の形状。

### 4. 重複チェックの追加

`generateGeojsonAssetMap()` 内で両マニフェスト間の論理ID重複をチェックし、重複時は警告を出力する。マージ時は graph が features を上書きする（後勝ち）。

## 生成される import パスの例

| 旧パス | 新パス |
|--------|--------|
| `@/assets/maps/imdf/address.json` | `@/assets/maps/features/address.json` |
| `@/assets/maps/imdf/studyhall/rooms/1F.json` | `@/assets/maps/features/studyhall/rooms/1F.json` |
| `@/assets/maps/imdf/routing/studyhall/1F_node.json` | `@/assets/maps/graph/routing/studyhall/1F_node.json` |

## ファイル

| ファイル | 種別 | 説明 |
|----------|------|------|
| `mobile/scripts/generate_geojsonAssetMap.js` | 修正 | メインスクリプト |
| `mobile/src/data/geojson/geojsonAssetMap.ts` | 自動生成 | 手動編集不要、`npm run data:load` で再生成 |

## Verification

- `geojsonAssetMap.ts` の consumer は他になく、`relativePath` 変更の影響はゼロ
- マニフェストファイルが存在しない環境では `npm run data:load` の zip 展開後に初めて動作するため、静的チェックはスキップ
- `scripts/generate_geojsonAssetMap.js` の構文エラーなし（読み取り確認済み）
