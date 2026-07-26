---
agent: IMP
task_id: TASK-001
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
related:
  - "[CoordinateTransformEngine](../../../../../frontieratlas/mobile/src/core/coordinate/CoordinateTransformEngine.ts)"
  - "[buildingCoordinateRegistry](../../../../../frontieratlas/mobile/src/core/coordinate/buildingCoordinateRegistry.ts)"
  - "[index.ts](../../../../../frontieratlas/mobile/src/core/coordinate/index.ts)"
  - "[building-config.json](../../../../../frontieratlas/tools/map-assets/exports/graph/studyhall/building-config.json)"
  - "[BuildingCoordinateConfig types](../../../../../frontieratlas/mobile/src/core/coordinate/types.ts)"
tags:
  - IMP
  - implementation
  - TASK-001
  - coordinate
  - building-registry
  - mobile
---

# Implementation Log: Building Coordinate Registry for Mobile App

## Summary

Created `buildingCoordinateRegistry.ts` — a side-effect module that registers building coordinate configurations with `CoordinateTransformEngine` on import. Updated barrel export `index.ts` to import the registry at the end.

## Problem

`CoordinateTransformEngine.registerBuilding()` was never called anywhere in the app. This meant that any `toGPS()`/`toLocal()` call with a buildingId would throw `"BuildingCoordinateConfig not registered"` error. The coordinate transform system existed but was unusable.

## Changes Made

### File: `buildingCoordinateRegistry.ts` (NEW)

Side-effect module that registers the following buildings on import:

| buildingId | origin (lat, lng) | northAzimuth | scale | Source |
|-----------|-------------------|-------------|-------|--------|
| `studyhall` | 35.498113, 139.677849 | 0 | 1.0 | `studyhall/building-config.json` |
| `interact` | 35.498113, 139.677849 | 0 | 1.0 | Temporary config (same as studyhall) |
| `bldg-a` | 35.498113, 139.677849 | 0 | 1.0 | Alias for studyhall (MapScreen.tsx placeholder) |

Origin values match `tools/map-assets/exports/graph/studyhall/building-config.json`:
- `lat_0=35.498113`, `lon_0=139.677849`
- `northAzimuth: 0`
- `scale: 1.0`

### File: `index.ts` (MODIFIED)

Added an import-only statement at the end of the barrel export:

```typescript
// Building Coordinate Registry（サイドエフェクト：全建物の座標変換設定を登録）
import "./buildingCoordinateRegistry";
```

Placed after all other exports so dependencies (`CoordinateTransformEngine`, `proj4`) are loaded before the side-effect runs.

## Design Decisions

1. **Side-effect pattern**: The module has no exports — importing it triggers registration. This ensures buildings are registered before any coordinate transform function is used.

2. **Placement in barrel export**: The import is at the end of `index.ts`, after all exports, guaranteeing that `CoordinateTransformEngine` and `proj4` are already initialized when `registerBuilding()` executes.

3. **"bldg-a" alias**: Created as a spread of `STUDYHALL_CONFIG` with overridden `buildingId`, since `MapScreen.tsx` uses "bldg-a" as a placeholder.

4. **Interim "interact" config**: Shares origin with studyhall until a dedicated `interact/building-config.json` is created in the tools pipeline.

## Verification

- ✅ `buildingCoordinateRegistry.ts` — no TypeScript errors
- ✅ `index.ts` — no TypeScript errors
- ✅ `npx tsc --noEmit --pretty` — exit code 0, no errors across the project
- ✅ Origin matches `tools/map-assets/exports/graph/studyhall/building-config.json` exactly
- ✅ Import placed after all other exports to ensure correct load order

## Open Questions

None. Implementation follows the task specification precisely.
