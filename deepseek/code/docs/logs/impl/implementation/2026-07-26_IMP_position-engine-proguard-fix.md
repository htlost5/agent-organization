---
agent: IMP
task_id: Position Engine Android Integration Fixes
date: 2026-07-26
status: done
category: log
destination: logs/impl/implementation/
tags:
  - IMP
  - position-engine
  - proguard
  - android
  - kotlinx.serialization
---

# Implementation Log: Position Engine Android Integration Fixes

## Summary

Fixed three Android integration issues for the position-engine library and its consuming mobile app.

## Changes

### Fix 1: Add ProGuard Consumer Rules for kotlinx.serialization

**File**: `position-engine/position-engine/consumer-rules.pro`

Added ProGuard keep rules for kotlinx.serialization so that when the consuming app builds with R8/ProGuard enabled, serialization classes used by position-engine are not stripped.

Rules added:
- `keepattributes *Annotation*, InnerClasses` — required for kotlinx.serialization
- `dontnote kotlinx.serialization.AnnotationsKt` — suppress warnings
- `keepclassmembers` / `keepclasseswithmembers` for `kotlinx.serialization.json.**` — keep serializers
- `keep,allowobfuscation` for `com.htlost5.frontieratlas.position.common.**` — keep data model classes
- `keep class kotlin.Metadata` — keep reflection-based serialization metadata

### Fix 2: Add HIGH_SAMPLING_RATE_SENSORS Permission

**File**: `position-engine/position-engine/src/main/AndroidManifest.xml`

Added `<uses-permission android:name="android.permission.HIGH_SAMPLING_RATE_SENSORS" />` before `</manifest>`. This permission is needed on Android 12+ (API 31+) for high-frequency sensor data (>200Hz).

### Fix 3: Add Position Engine ProGuard Keep Rules to Mobile App

**File**: `mobile/android/app/proguard-rules.pro`

Added safety-net ProGuard rules at the end of the file for position-engine classes, ensuring they are preserved even if the library's consumer-rules.pro is not automatically picked up.

Rules added:
- `keep,allowobfuscation` for `com.htlost5.frontieratlas.position.**` — keep all position-engine classes
- `keepattributes`, `dontnote` for kotlinx.serialization support
- `keepclassmembers` / `keepclasseswithmembers` for `kotlinx.serialization.json.**`

## Verification

All three files were read back after editing to confirm correctness:
- `consumer-rules.pro` — contains all required kotlinx.serialization rules ✓
- `AndroidManifest.xml` — well-formed XML with new permission added before `</manifest>` ✓
- `proguard-rules.pro` — position-engine rules appended after existing content ✓
