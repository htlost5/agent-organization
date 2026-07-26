---
agent: TST
task_id: TASK-geojsonAssetMap-registry-fix
date: 2026-07-26
status: draft
category: log
destination: docs/logs/impl/testing/
related:
  - "[IMP generate_geojsonAssetMap.js log](../implementation/2026-07-25_IMP_heading-alpha-param.md)"
tags:
  - TST
  - testing
  - TASK-geojsonAssetMap-registry-fix
  - geojson
  - registry
  - mobile
  - assets
---

# Testing Log: generate_geojsonAssetMap.js Registry Fix

## 修正概要

`scripts/generate_geojsonAssetMap.js` を修正し、単一マニフェスト（`assets/maps/manifest.json`）読取から
2マニフェスト（`assets/maps/features/manifest.json` + `assets/maps/graph/manifest.json`）読取に変更。
生成パスに `features/` または `graph/` プレフィックスを追加。

---

## テスト結果サマリ

| テスト項目 | 結果 | 備考 |
|-----------|------|------|
| テスト1: `data:gen-registry` 単体実行 | ❌ **不合格** | スクリプトにトップレベル関数呼び出しが不足 |
| テスト2: `data:load` 完全実行 | ✅ **合格** | 正常完了、正しいパスでレジストリ生成 |
| テスト3: TypeScript 型チェック | ⚠️ **条件付き不合格** | 2エラー → いずれも `geojsonAssetMap.ts` 以外のファイル |
| Expo Lint | ✅ **合格** | 0 errors, 16 warnings（既存の警告のみ） |
| 生成ファイル検証 | ✅ **合格** | 40エントリ、features/graph パス、imdf パスなし |

---

## テスト詳細

### テスト1: `data:gen-registry` 単体実行 ❌ 不合格

```bash
$ cd /d/htlost5-workspace/projects/frontieratlas/mobile
$ npm run data:gen-registry
> frontieratlas-app@0.18.1 data:gen-registry
> node ./scripts/generate_geojsonAssetMap.js
```

**結果**: コマンドは exit code 0 で終了したが、出力は空。`geojsonAssetMap.ts` は再生成されず、旧 `imdf/` パスのまま。

**原因**: `scripts/generate_geojsonAssetMap.js` は `export async function generateGeojsonAssetMap()` として関数をエクスポートしているが、ファイル末尾にトップレベルでの関数呼び出しがない。そのため `node` で直接実行しても何も実行されない。

**修正方法**: ファイル末尾に以下を追加する必要がある:

```js
// 直接実行用
await generateGeojsonAssetMap();
```

または:

```js
generateGeojsonAssetMap().catch((e) => {
  console.error(e);
  process.exit(1);
});
```

---

### テスト2: `data:load` 完全実行 ✅ 合格

```bash
$ cd /d/htlost5-workspace/projects/frontieratlas/mobile
$ npm run data:load
> frontieratlas-app@0.18.1 data:load
> node ./scripts/build-data-load.js

[RESET] D:\htlost5-workspace\projects\frontieratlas\mobile\assets\maps deleted
Assets updated successfully
Wrote registry to D:\htlost5-workspace\projects\frontieratlas\mobile\src\data\geojson\geojsonAssetMap.ts
```

**確認結果**:
- ✅ スクリプトがエラーなく最後まで完了（exit code 0）
- ✅ `[RESET] assets/maps deleted` 出力あり
- ✅ `Assets updated successfully` 出力あり
- ✅ `Wrote registry to ...` 出力あり

**生成ファイルパス検証**: `src/data/geojson/geojsonAssetMap.ts` を確認:

- ✅ Import パスが `@/assets/maps/features/...` または `@/assets/maps/graph/...` である
- ✅ 旧パス `@/assets/maps/imdf/...` が含まれていない
- ✅ features マニフェスト（20エントリ）と graph マニフェスト（20エントリ）が両方含まれる（計40エントリ）
- ✅ `relativePath` が `features/...` または `graph/...` プレフィックスを持つ

**生成されたエントリ一覧**:

**features (20)**:
- address, studyhall_rooms_1F-5F, studyhall_stairs, studyhall_surface_1F-5F, studyhall_surface, studyhall_surfaceback, studyhall_walkable_1F-5F, venue_venue

**graph (20)**:
- display_studyhall_1F-5F_edge, display_studyhall_1F-5F_node, routing_studyhall_1F-5F_edge, routing_studyhall_1F-5F_node

---

### テスト3: TypeScript 型チェック ⚠️ 条件付き不合格

```bash
$ cd /d/htlost5-workspace/projects/frontieratlas/mobile
$ npx tsc --noEmit
```

**エラー内容（2件）**:

```
src/data/geojson/index.ts:10:27 - error TS2307: Cannot find module '@/assets/maps/manifest.json'
src/data/geojson/service/AssetRestoreService.ts:12:27 - error TS2307: Cannot find module '@/assets/maps/manifest.json'
```

**分析**:
- いずれのエラーも `generate_geojsonAssetMap.js` の修正に起因するものではなく、**他ファイル**が旧 `@/assets/maps/manifest.json` を参照しているために発生
- `geojsonAssetMap.ts` 自体の型チェックは問題なし
- これらのファイルは `data:load` による `assets/maps/` の再展開後に旧 `manifest.json` が削除されたためエラーが発生

**要修正ファイル**:
1. `src/data/geojson/index.ts` — 行10: `import assetManifest from "@/assets/maps/manifest.json"`
2. `src/data/geojson/service/AssetRestoreService.ts` — 行12: `import assetManifest from "@/assets/maps/manifest.json"`

---

### Expo Lint ✅ 合格

```bash
$ npx expo lint
```
- 0 errors, 16 warnings
- 全警告は既存のもので、本修正とは無関係
- `geojsonAssetMap.ts` 関連の警告・エラーは0件

---

## 総合判定

**判定: ⚠️ 部分成功 — 不具合あり**

| 観点 | 判定 | 理由 |
|------|------|------|
| 修正ロジックの正確性 | ✅ 合格 | 2マニフェスト読取、prefix 付与のロジックは正しい |
| `data:load` 経由の動作 | ✅ 合格 | 正常に動作し、正しいレジストリを生成 |
| `data:gen-registry` 直接実行 | ❌ 不合格 | トップレベル関数呼び出しがないため何も実行されない |
| 生成ファイルの内容 | ✅ 合格 | パス・エントリ数・prefix すべて正しい |
| 型チェック (geojsonAssetMap.ts) | ✅ 合格 | 直接の型エラーなし |
| 型チェック (他ファイル) | ❌ 不合格 | 2ファイルが旧 manifest.json を参照 |
| Expo Lint | ✅ 合格 | 0 errors |

---

## 次のアクション

1. **IMP に委譲**: `scripts/generate_geojsonAssetMap.js` にトップレベル関数呼び出しを追加
2. **IMP に委譲**: 以下の2ファイルの旧 manifest.json 参照を新しい2マニフェスト構造に対応させる
   - `src/data/geojson/index.ts`
   - `src/data/geojson/service/AssetRestoreService.ts`
3. **再テスト**: 上記修正後に全テストを再実行

