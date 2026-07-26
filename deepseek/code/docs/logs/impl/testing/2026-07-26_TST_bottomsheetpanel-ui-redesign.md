---
agent: TST
task_id: TASK-BottomSheetPanel
date: 2026-07-26
status: completed
category: log
destination: logs/impl/testing/
tags:
  - TST
  - BottomSheetPanel
  - UI-redesign
  - testing
---

# Test Log: BottomSheetPanel UI Redesign

## Verdict: ⚠️ 条件付き合格 (Conditionally Passed)

## Test Summary

| Test | Result | Details |
|------|--------|---------|
| TypeScript compilation (`npx tsc --noEmit`) | ✅ PASS | 0 errors |
| ESLint (`npx expo lint`) | ⚠️ 17 warnings | 0 errors. 1 new (CollapsedBar unused import) + 16 pre-existing |
| Existing unit/integration tests | ⏭️ N/A | No test framework configured in project |
| Structural verification | ✅ PASS | All 5 new files exist at expected paths |
| Import resolution | ✅ PASS | All imports resolve correctly (verified via tsc) |
| CollectionControls absolute positioning removed | ✅ PASS | No `position: 'absolute'` in styles |
| CalibrationControls absolute positioning removed | ✅ PASS | No `position: 'absolute'` in styles |
| FloorChange removed from layout | ✅ PASS | Not imported by `app/` files anymore (dead code only) |
| MapControlsFC not used in layout | ✅ PASS | `home/_layout.tsx` uses direct imports |
| UserLocation panelHeight prop | ✅ PASS | `useAnimatedStyle` with `(panelHeight?.value ?? 0) + 10` |
| ScreenFC visible="nothing" | ✅ PASS | Applied in `app/(tabs)/_layout.tsx` |
| BottomSheetPanel ref + SharedValue | ✅ PASS | `panelRef` + `panelHeight` SharedValue shared with UserLocation |

## TypeScript Compilation

```
$ npx tsc --noEmit
→ No output (0 errors)
```

- **Result**: ✅ PASS

## ESLint

```
$ npx expo lint
→ 17 problems (0 errors, 17 warnings)
```

| Warning | File | Type |
|---------|------|------|
| Import in body of module; reorder to top | `src/core/coordinate/index.ts:41` | Pre-existing |
| 'useMapContext' is defined but never used | `src/features/home/chrome/BottomSheetPanel/CollapsedBar.tsx:7` | **NEW** (REV issue #1) |
| 'setNavGraphNodes' is assigned a value but never used | `src/features/home/map/hooks/useBatchMapData.ts:68` | Pre-existing |
| 'e' is defined but never used (×4) | `src/features/home/map/hooks/useCollectionSession.ts` | Pre-existing |
| 'MAP_LOAD_TIMEOUT_MS'/'isTimeout' unused | `src/features/home/map/overlays/MapContainer.tsx` | Pre-existing |
| 'Point'/'selectedCollection' unused | `src/features/home/map/overlays/NodeSelectionLayer.tsx` | Pre-existing |
| 'selectedNodeIds'/'handleNodeSelect' unused | `src/features/home/map/overlays/controls/CalibrationControls.tsx` | Pre-existing |
| 'startSession'/'stopSession'/'correctPosition'/'setSnapState' unused | `src/features/home/map/overlays/controls/CollectionControls.tsx` | Pre-existing |

- **Result**: ⚠️ 0 errors. 1 new warning (REV condition: unused `useMapContext` import in CollapsedBar.tsx L7). 16 pre-existing warnings unrelated to this change.

## Structural Verification

### New Files (all confirmed present)
| File | Status |
|------|--------|
| `src/features/home/chrome/BottomSheetPanel/styles.ts` | ✅ |
| `src/features/home/chrome/BottomSheetPanel/index.tsx` | ✅ |
| `src/features/home/chrome/BottomSheetPanel/CollapsedBar.tsx` | ✅ |
| `src/features/home/chrome/BottomSheetPanel/FloorSelector.tsx` | ✅ |
| `src/features/home/chrome/BottomSheetPanel/ExpandedContent.tsx` | ✅ |

### Modified Files (key changes verified)
| File | Change Verified |
|------|-----------------|
| `app/(tabs)/_layout.tsx` | `ScreenFC visible="nothing"` ✅ |
| `app/(tabs)/home/_layout.tsx` | SharedValue panelHeight, BottomSheetPanel added, MapControlsFC removed ✅ |
| `src/features/home/map/MapRoot.tsx` | CollectionControls/CalibrationControls removed from children ✅ |
| `src/features/home/map/overlays/controls/index.tsx` | FloorChange/CollectionControls/CalibrationControls exports removed ✅ |
| `src/features/home/map/overlays/controls/UserLocation.tsx` | panelHeight prop + useAnimatedStyle ✅ |
| `src/features/home/map/overlays/controls/CollectionControls.tsx` | Absolute positioning removed ✅ |
| `src/features/home/map/overlays/controls/CalibrationControls.tsx` | Absolute positioning removed ✅ |

### Unchanged Dead Code (no impact)
- `FloorChange.tsx` — file remains as dead code, not imported by any active component
- `MapControlsFC` in `controls/index.tsx` — function remains but not used anywhere

## REV Issues Re-evaluation

| Issue | Severity | Status | Notes |
|-------|----------|--------|-------|
| 🟡 Issue 1: Unused import `useMapContext` in CollapsedBar.tsx L7 | Minor | ⚠️ Confirmed (lint warning) | REV's condition for approval |
| 🟡 Issue 2: No activeOffsetY on Pan gesture | Minor | ⚠️ Not fixed (recommendation) | Does not block testing |
| 🟡 Issue 3: Gesture conflict Pan vs ScrollView | Minor | ⚠️ Not tested (needs device) | Requires device testing |

## Conclusion

- **TypeScript**: ✅ 0 errors — strict mode compliance confirmed
- **ESLint**: ⚠️ 17 warnings (0 errors) — 1 new warning matches REV's condition
- **Existing tests**: ⏭️ No test framework configured; no test files exist
- **Structural integrity**: ✅ All files present, imports resolve correctly
- **Verdict**: ⚠️ **Conditionally Passed** — all functional requirements structurally verified. The only new issue is the REV-flagged unused import (lint warning). No TypeScript errors, no broken imports.
