---
agent: TST
task_id: TASK-compile-fix
date: 2026-07-26
status: draft
category: log
destination: logs/impl/testing/
related:
  - "[position-engine build.gradle.kts](../../../../../../../position-engine/position-engine/build.gradle.kts)"
  - "[mobile android build.gradle](../../../../../../../mobile/android/build.gradle)"
tags:
  - TST
  - testing
  - compile-fix
  - position-engine
---

# Test Report: PositionEngine Compile Fix (TASK-compile-fix)

## Test Objective

`kotlin("android") version "2.1.20"` を position-engine の plugins ブロックに追加した修正を検証。

## Test Command

```
cd /d/htlost5-workspace/projects/frontieratlas/mobile/android
./gradlew.bat :app:compileDebugKotlin --configure-on-demand --build-cache
```

## Test Result

**FAILED** ❌

## Error Detail

```
FAILURE: Build failed with an exception.

* Where:
Build file '.../position-engine/position-engine/build.gradle.kts' line: 1

* What went wrong:
Error resolving plugin [id: 'org.jetbrains.kotlin.android', version: '2.1.20']
> The request for this plugin could not be satisfied because the plugin is
  already on the classpath with an unknown version, so compatibility cannot
  be checked.
```

## Root Cause

ルートの `android/build.gradle` の buildscript ブロックですでに
`classpath('org.jetbrains.kotlin:kotlin-gradle-plugin')` が宣言されており、
Kotlin Gradle プラグインがクラスパス上に存在する（バージョン未指定 → 未知のバージョン）。
その状態でサブプロジェクトの plugins ブロックで `kotlin("android") version "2.1.20"`
とバージョン指定すると、Gradle は互換性検証ができずエラーとなる。

## Verification Items

| # | Item | Result |
|---|------|--------|
| 1 | `:app:compileDebugKotlin` success | ❌ Fail (configuration phase error) |
| 2 | No "Cannot add extension with name 'kotlin'" error | N/A — different error occurred |
| 3 | No new errors | N/A — build did not reach compilation |

## Recommended Fix

`position-engine/build.gradle.kts` の plugins ブロックから
`kotlin("android")` のバージョン指定を削除する：

```kotlin
plugins {
    id("com.android.library")
    kotlin("android")  // version "2.1.20" を削除
    kotlin("plugin.serialization") version "2.1.20"
    id("org.jlleitschuh.gradle.ktlint")
}
```

`kotlin("plugin.serialization")` のバージョン指定はルートの buildscript
依存関係に含まれていない可能性があるため維持。ただし同様のエラーが発生した場合は
こちらもバージョン指定を削除する。

## Next Actions

1. IMP に差し戻し — バージョン指定を削除した修正を依頼
2. 修正後、再度 `:app:compileDebugKotlin` で再テスト
