---
agent: IMP
task_id: TASK-FIX-MISSINGPERMISSION-WIFISCANNER
date: 2026-07-25
status: done
category: log
destination: logs/impl/implementation/
tags:
  - IMP
  - implementation
  - wifi-scanner
  - permission
---

# Implementation Log: Fix MissingPermission lint errors in WifiScanner.kt

## Summary

Fixed 2 `MissingPermission` lint errors in `WifiScanner.kt` by adding explicit `ACCESS_FINE_LOCATION` permission checks using `ContextCompat.checkSelfPermission`.

## Changes

### File: `position-engine/src/main/java/com/htlost5/frontieratlas/position/sensor/WifiScanner.kt`

#### Error 1 — `BroadcastReceiver.onReceive` (line 60)

Added permission guard before accessing `wifiManager?.scanResults`:

```kotlin
if (ContextCompat.checkSelfPermission(
        context,
        Manifest.permission.ACCESS_FINE_LOCATION
    ) != PackageManager.PERMISSION_GRANTED
) {
    return
}
```

- Behavior: silently skips emitting scan results when permission is missing (defensive check; `start()` already checks `isAvailable`)

#### Error 2 — `scanOnce()` (line 126)

Added permission guard before accessing `wifiManager?.scanResults`:

```kotlin
if (ContextCompat.checkSelfPermission(
        context,
        Manifest.permission.ACCESS_FINE_LOCATION
    ) != PackageManager.PERMISSION_GRANTED
) {
    throw SecurityException("ACCESS_FINE_LOCATION permission is required for WiFi scanning")
}
```

- Behavior: throws `SecurityException` since this is a public API method

## Verification

- `ktlintCheck`: BUILD SUCCESSFUL (5 executed tasks on clean build)
- `lint` (Android Lint): BUILD SUCCESSFUL (61 tasks, 18 executed)
- All existing logic preserved (permission checks are additive, no existing guards modified)
- `isAvailable` property left unchanged

## Notes

- Reused existing imports (`ContextCompat`, `Manifest`, `PackageManager`)
- Initial attempt used `return@BroadcastReceiver` which caused "Unresolved label" compilation error; fixed by using bare `return` (valid inside anonymous object methods)
- 4-space Kotlin indentation preserved throughout
