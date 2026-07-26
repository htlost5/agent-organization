---
agent: IMP
task_id: TASK-position-engine-step4-5
date: 2026-07-25
status: approved
category: log
destination: docs/logs/impl/implementation/
related:
  - "[TASK-position-engine-step4-5](../shared/tasks/active/TASK-position-engine-step4-5.md)"
tags:
  - IMP
  - implementation
  - TASK-position-engine-step4-5
---

# Implementation Log: Step 4-5 CRITICAL Fixes

## Overview

Fixed 2 CRITICAL issues identified by REV in the sensor layer implementation.

---

## CRITICAL-1: resume() no-op in 3 Providers

### Problem
After `pause()` calls `stop()` (which unregisters sensor listeners via `sensorManager.unregisterListener()`), `resume()` was a no-op with only a comment `// Will be restarted by SensorController`. This meant sensor data flow stopped permanently after a pause→resume cycle.

### Fix Applied (Option A)
Extracted listener creation + registration into a private `registerListener()` method in all 3 providers. This method is called from both `start()` and `resume()`.

**Changes per file**:

1. **MotionSensorProvider.kt**
   - Added `private var scope: CoroutineScope? = null` field
   - Extracted listener registration into `private fun registerListener()`
   - `start()` now stores scope + calls `registerListener()`
   - `resume()` now calls `registerListener()` if available and scope exists

2. **MagneticProvider.kt**
   - Same pattern: added scope field, extracted `registerListener()`, updated `start()` and `resume()`

3. **PressureProvider.kt**
   - Same pattern: added scope field, extracted `registerListener()`, updated `start()` and `resume()`

### Rationale
- Option A chosen over Option B (calling `start()` from `resume()`) because it avoids re-executing the `stop()` call at the top of `start()` unnecessarily, and keeps the scope reference intact.
- The listener is created fresh each time to avoid stale references.

---

## CRITICAL-2: recoverable=false for non-critical sensor absence

### Problem
In `SensorController.initialize()`, gyroscope and rotation vector sensor absence emitted `EngineError(recoverable = false)`. Per spec (09a §10), only accelerometer absence should be fatal (recoverable = false).

### Fix Applied
Changed `recoverable = false` → `recoverable = true` for:
- `GYROSCOPE_UNAVAILABLE` error
- `ROTATION_VECTOR_UNAVAILABLE` error

Magnetic and pressure sensor absence already correctly logged without errors (no change needed).

### Result
- Accelerometer absence: `recoverable = false` (fatal) ✅ — unchanged
- Gyroscope absence: `recoverable = true` (non-fatal) ✅ — fixed
- Rotation vector absence: `recoverable = true` (non-fatal) ✅ — fixed
- Magnetic absence: no error, just log ✅ — already correct
- Pressure absence: no error, just log ✅ — already correct

---

## Verification

```
./gradlew :position-engine:compileDebugKotlin → BUILD SUCCESSFUL in 4s
```

No compile errors, no warnings.

---

## Handoff to REV

Please re-review the following files:
1. `position-engine/.../sensor/MotionSensorProvider.kt`
2. `position-engine/.../sensor/MagneticProvider.kt`
3. `position-engine/.../sensor/PressureProvider.kt`
4. `position-engine/.../sensor/SensorController.kt`
