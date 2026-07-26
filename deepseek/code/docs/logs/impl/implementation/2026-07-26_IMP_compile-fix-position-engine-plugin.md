---
agent: IMP
task_id: TASK-XXX
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
tags:
  - IMP
  - position-engine
  - build-fix
  - kotlin-plugin
---

# Implementation Log: position-engine build.gradle.kts — Kotlin Android プラグイン追加

## Task

`mobile/android/app/src/main/java/com/htlost/frontieratlas/dev/PositionEnginePackage.kt` のコンパイルエラーを修正する。

## Root Cause

`position-engine/position-engine/build.gradle.kts` の plugins ブロックに `kotlin("android")` プラグインが欠落していた。

- `kotlin("plugin.serialization")` は Kotlin シリアライゼーションのコンパイラプラグインであり、Kotlin Android プラグインではない
- このため、Kotlin ソースファイル（`PositionEngineModule.kt` など）が正しくコンパイルされず、`PositionEngineModule` クラスが生成されていなかった
- 結果として、`PositionEnginePackage.kt` で `PositionEngineModule` への参照解決が失敗していた

## Changes Made

**File**: `position-engine/position-engine/build.gradle.kts`

plugins ブロックに `kotlin("android") version "2.1.20"` を追加:

```kotlin
plugins {
    id("com.android.library")
    kotlin("android") version "2.1.20"         // ← 追加
    kotlin("plugin.serialization") version "2.1.20"
    id("org.jlleitschuh.gradle.ktlint")
}
```

- バージョンは `libs.versions.toml` の Kotlin バージョン（2.1.20）と一致するよう明示指定
- `kotlin("plugin.serialization")` の直前に配置（logical order: Android プラグイン → コンパイラプラグイン）

## Expected Effect

- position-engine モジュール内の Kotlin ソースファイルが正しくコンパイルされる
- `PositionEngineModule` クラスが生成され、`PositionEnginePackage.kt` からの参照解決が成功する
- `:app:compileDebugKotlin` のビルドエラーが解消される

## Verification

- [x] ファイル編集完了
- [ ] ビルド確認（IMP の Scope 外 — TST または手動で実施）
