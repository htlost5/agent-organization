---
agent: IMP
task_id: TASK-XXX
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
tags:
  - IMP
  - implementation
  - position-engine
  - turbomodule
  - native-bridge
---

# Implementation Log: Position Engine TurboModule Bridge Integration

## Summary

Implemented TurboModule-compatible bridge logic for the Position Engine integration on the React Native (Expo SDK 57 / RN 0.86) side. Created new service layer (AssetService, PermissionService), engine lifecycle hook, and restructured the nativeBridge module for dual-architecture compatibility.

## Files Created

| # | File | Description |
|---|------|-------------|
| 1 | `mobile/src/infra/nativeBridge/PositionEngineEvents.ts` | Event name constants + payload types (extracted from PositionEngineModule.ts) |
| 2 | `mobile/src/infra/nativeBridge/NativePositionEngine.ts` | Codegen-compatible TurboModule spec (interface Spec + TurboModuleRegistry.getEnforcing) |
| 3 | `mobile/src/infra/nativeBridge/PositionEngineAccessor.ts` | Dual-architecture resolver: TurboModule → NativeModules fallback |
| 4 | `mobile/src/infra/nativeBridge/index.ts` | Barrel exports for nativeBridge |
| 5 | `mobile/src/shared/services/AssetService.ts` | Map Assets download/verify/extract/cache management (adapted to existing Record-based manifestTypes) |
| 6 | `mobile/src/shared/services/PermissionService.ts` | Sensor/location/storage permission management (Platform-specific + native bridge fallback) |
| 7 | `mobile/src/features/home/map/hooks/useEngineLifecycle.ts` | Engine lifecycle hook (fatal/temporary error handling, recovery, timeout recovery) |

## Files Modified

| # | File | Change |
|---|------|--------|
| 8 | `mobile/src/infra/nativeBridge/PositionEngineModule.ts` | Removed ENGINE_EVENTS/EngineEventPayloads (now re-exported from PositionEngineEvents); changed export to use PositionEngineAccessor.module |
| 9 | `mobile/src/shared/services/PositionService.ts` | Changed ensureModule() to use PositionEngineAccessor.getModule(); removed PositionEngineNotAvailableError class |
| 10 | `mobile/src/features/home/map/context/PositionEngineContext.tsx` | Changed NativeEventEmitter to use PositionEngineAccessor.module; removed NativeModules import |

## Key Design Decisions

### Dual-Architecture Module Resolution
- `PositionEngineAccessor` tries TurboModule first (New Architecture), falls back to NativeModules (Old Architecture)
- `PositionEngineModule` module export now delegates to `PositionEngineAccessor.module`
- Backward compatible: existing imports from `PositionEngineModule` still resolve ENGINE_EVENTS, PositionEngineNativeModule, etc.

### Event Extraction
- Moved `ENGINE_EVENTS` and `EngineEventPayloads` to dedicated `PositionEngineEvents.ts`
- `PositionEngineModule.ts` re-exports them via `export { ... } from "./PositionEngineEvents"`
- Existing consumers (`PositionEngineContext.tsx`) continue to import from `PositionEngineModule` unchanged

### AssetService Domain Type Alignment
- Adapted to match existing `manifestTypes.ts` where `files` is `Record<string, BuildManifestItem>` (object, not array)
- Uses `Object.entries()` iteration for changed-file detection
- ZIP extraction deferred to Phase 2 (expo-file-system lacks native unzip in SDK 57)

### PermissionService
- Location + Storage: Android `PermissionsAndroid` API (with Platform.Version check for scoped storage)
- Wi-Fi + Sensors: delegated to native bridge `checkPermissions()`/`requestPermissions()` methods
- Network: always "granted" (no runtime permission)

### useEngineLifecycle
- Maps to MapLibre degradation states: fatal → stop(), temporary → pause(), recovery → initialize+start() or resume()
- Timeout recovery escalates to fatal on failure

## Verification Stack

- [x] No circular imports in nativeBridge module chain
- [x] Backward-compatible exports from PositionEngineModule.ts
- [x] All imports use `@/src/` path alias consistently
- [x] Domain types from `domain/positionEngine.ts` and `domain/manifestTypes.ts` used correctly
- [x] TurboModule spec mirrors PositionEngineNativeModule interface methods

## Open Questions

- `FileSystem.getFreeDiskStorageAsync()` availability in Expo SDK 57 — may need try/catch fallback (already implemented)
- ZIP extraction library not yet integrated — Phase 2 work item
- Native `checkPermissions()`/`requestPermissions()` methods on the Android side need to match the expected return shape `{ wifi?: string, sensors?: string }`

## Fixes Applied After Initial Implementation

### TypeScript Error Fixes

| Error | File | Fix |
|-------|------|-----|
| `Type 'PositionEngineNativeModule' has no properties in common with type 'TurboModule'` | `PositionEngineAccessor.ts` | Removed generic parameter from `TurboModuleRegistry.get<>()`, used type assertion instead |
| `Property 'documentDirectory' does not exist on type 'expo-file-system'` | `AssetService.ts` | Rewrote to use new SDK 57 API (`Paths`, `File`, `Directory` classes) |
| `Property 'cacheDirectory' does not exist on type 'expo-file-system'` | `AssetService.ts` | Same as above |
| `Property 'checkPermissions' / 'requestPermissions' does not exist on type 'PositionEngineNativeModule'` | `PermissionService.ts` | Used local structural type alias with the specific methods needed |

### Lint Warning Fixes

| Warning | File | Fix |
|---------|------|-----|
| `Prefer using primitive object` (21 occurrences) | `NativePositionEngine.ts` | Changed `Object` → `object` |
| `'tempZipDir' is assigned a value but never used` | `AssetService.ts` | Removed unused variable |

### Expo File System API Change (SDK 57)

In Expo SDK 57, the `expo-file-system` API was completely rewritten. AssetService was updated from the legacy API to the new API:

| Operation | Legacy API | New SDK 57 API |
|-----------|-----------|----------------|
| Document directory | `FileSystem.documentDirectory` | `Paths.document` (returns `Directory`) |
| Cache directory | `FileSystem.cacheDirectory` | `Paths.cache` (returns `Directory`) |
| Free disk space | `FileSystem.getFreeDiskStorageAsync()` | `Paths.availableDiskSpace` |
| Download file | `FileSystem.downloadAsync(url, path)` | `File.downloadFileAsync(url, destination)` |
| Directory exists | `FileSystem.getInfoAsync(path)` | `dir.exists` |
| Create directory | `FileSystem.makeDirectoryAsync(path)` | `dir.create({ intermediates: true })` |
| Delete directory | `FileSystem.deleteAsync(path)` | `dir.delete()` |

## Next Actions

Hand off to REV for code review.
