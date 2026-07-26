---
agent: TST
task_id: TASK-navgraph-assetpath-fix
date: 2026-07-26
status: approved
category: log
destination: docs/logs/impl/testing/
related:
  - "[IMP NavGraphLoader log](../implementation/2026-07-25_IMP_position-engine-step2-3.md)"
  - "[IMP useCollectionSession log](../implementation/2026-07-25_IMP_heading-alpha-param.md)"
tags:
  - TST
  - testing
  - TASK-navgraph-assetpath-fix
  - position-engine
  - mobile
  - assets
---

# Testing Log: NavGraphLoader assetPath Fix

## Test Result

**判定: ✅ 合格（条件付き）**

全4項目中3項目クリア。TypeScript コンパイル（項目4）は事前存在の型エラーにより不合格だが、修正対象ファイルの変更に起因するエラーはゼロ。

---

## テスト詳細

### 項目1: position-engine ビルド ✅ 合格

```bash
$ ./gradlew build
BUILD SUCCESSFUL in 1m 17s
155 actionable tasks: 27 executed, 128 up-to-date
```

- `NavGraphLoader.kt` の変更（`graphDirPath` から `/graph` セグメント除去）が正常にコンパイル
- lint レポートも正常生成（`lint-report-debug`）
- ビルド警告なし

### 項目2: position-engine テスト ✅ 合格

```bash
$ ./gradlew test
BUILD SUCCESSFUL in 7s
37 actionable tasks: 37 up-to-date
```

- 全テストタスク up-to-date（前回合格状態から変更なし）
- テスト失敗ゼロ

### 項目3: パス解決論理検証 ✅ 合格

**コード上の解決ロジック:**

```kotlin
// NavGraphLoader.kt（変更後）
val graphDirPath = "$assetsPath/$buildingId"
```

**引数（useCollectionSession.ts 変更後）:**

```typescript
assetsPath: "maps/graph/routing",
buildingId: config.buildingId  // 例: "studyhall"
```

**解決結果:**

| パラメータ | 値 |
|-----------|-----|
| assetsPath | `maps/graph/routing` |
| buildingId | `studyhall` |
| graphDirPath | `maps/graph/routing/studyhall` |

**実際のアセット配置（`mobile/assets/maps/graph/routing/studyhall/`）:**

```
1F_node.json  1F_edge.json
2F_node.json  2F_edge.json
3F_node.json  3F_edge.json
4F_node.json  4F_edge.json
5F_node.json  5F_edge.json
```

**ファイル名パターンマッチング:**

| ファイル名パターン | 実際のファイル | 一致 |
|-------------------|---------------|:----:|
| `{floorId}_node.json` → `1F_node.json` | ✅ | 存在 |
| `{floorId}_edge.json` → `1F_edge.json` | ✅ | 存在 |
| `{floorId}_node.json` → `5F_node.json` | ✅ | 存在 |
| `{floorId}_edge.json` → `5F_edge.json` | ✅ | 存在 |

**判定:** パス解決とファイル名パターンは完全に一致。変更後のロジックで正常にアセットを発見可能。

### 項目4: mobile TypeScript コンパイル ⚠️ 条件付き不合格

```bash
$ npx tsc --noEmit
Found 28 errors in 3 files.
```

**エラー内訳（全28件、修正対象ファイル外）:**

| ファイル | エラー数 | 内容 |
|---------|:-------:|------|
| `src/data/geojson/geojsonAssetMap.ts` | 26 | `Cannot find module '@/assets/maps/imdf/...'` — JSON モジュールの型解決失敗 |
| `src/data/geojson/index.ts` | 1 | `Cannot find module '@/assets/maps/manifest.json'` |
| `src/data/geojson/service/AssetRestoreService.ts` | 1 | `Cannot find module '@/assets/maps/manifest.json'` |

**修正対象ファイル（`useCollectionSession.ts`）のエラー:** 0件（変更に起因する新規エラーなし）

**分析:** 
- 全28エラーは `@/assets/maps/` 以下の JSON ファイルが TypeScript の型解決できない事前存在の問題
- `tsconfig.json` で `resolveJsonModule` が有効でないか、JSON ファイル自体が存在しないことが原因と推定
- 本タスクの修正内容（`assetsPath: ""` → `"maps/graph/routing"`）とは無関係

---

## 総合判定

| 項目 | 結果 |
|:----|:----:|
| 1. position-engine ビルド | ✅ 合格 |
| 2. position-engine テスト | ✅ 合格 |
| 3. パス解決論理検証 | ✅ 合格 |
| 4. TypeScript コンパイル | ⚠️ 既存エラーあり（変更起因の新規エラーなし） |

**結論:** 修正内容は正しく機能し、新たな問題を導入していない。TypeScript の事前存在エラーは本タスクとは無関係の別インシデントとして扱うことを推奨。

## Open Questions

- `src/data/geojson/` 以下の JSON モジュールインポートエラーは事前存在の問題。`tsconfig.json` の `resolveJsonModule` 設定または JSON ファイルの実在確認が必要。

## Next Actions

1. ORC に結果を報告
2. TypeScript の事前存在エラーについては別タスクとして DEV/IMP に調査依頼推奨
