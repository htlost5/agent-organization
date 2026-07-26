---
agent: IMP
task_id: TASK-001
date: 2026-07-26
status: draft
category: log
destination: logs/impl/implementation/
related:
  - "[Graph Transform Design](../../_inbox/2026-07-26_DWR_graph-transform-design.md)"
  - "[NavGraphLoader.kt](../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/graph/NavGraphLoader.kt)"
  - "[GeoJsonModels.kt](../../../../../frontieratlas/position-engine/position-engine/src/main/java/com/htlost5/frontieratlas/position/graph/GeoJsonModels.kt)"
tags:
  - IMP
  - implementation
  - TASK-001
  - navgraph-loader
  - position-engine
---

# Implementation Log: NavGraphLoader Update for New Graph Data Format

## Summary

Updated `NavGraphLoader.kt` to read the new pre-split node/edge file format produced by the graph transform pipeline, instead of the old combined `.geojson` format.

## Changes Made

### File: `NavGraphLoader.kt`

**Before (old behavior):**
- Scanned `assets/{assetsPath}/{buildingId}/` for `.geojson` files
- Each file contained both Point (node) and LineString (edge) features in one FeatureCollection
- Auto-generated node IDs (`node_0`, `node_1`, ...) if `nodeId` property was missing
- Auto-generated edge IDs (`edge_0`, `edge_1`, ...) if `edgeId` property was missing
- When LineStrings lacked `fromNodeId`/`toNodeId`, created synthetic nodes from each coordinate pair

**After (new behavior):**
- Scans `assets/{assetsPath}/{buildingId}/graph/` for `*_node.json` files
- Extracts `floorId` from filename: `"1F_node.json"` → `"1F"`
- Reads two files per floor: `{floorId}_node.json` (Point features only) and `{floorId}_edge.json` (LineString features only)
- All Point features → directly map to `Node` objects (nodeId is required, no auto-generation)
- All LineString features → directly map to `Edge` objects (edgeId, fromNodeId, toNodeId are required)
- Edge file is optional — gracefully falls back to empty edges collection if missing
- `calculateEdgeLength()`/`calculateDirection()` still work using `Node.localX`/`localY` (now in local meter coords)

### File: `GeoJsonModels.kt`

No changes needed — the output files are still GeoJSON FeatureCollection format, so existing serialization models work as-is.

## Verification

- ✅ `load()` method signature unchanged: `suspend fun load(assetsPath: String, buildingId: String): Result<Map<String, NavigationGraph>>`
- ✅ `loadWithIndex()` method signature unchanged
- ✅ All ID auto-generation removed
- ✅ All LineString fallback code removed
- ✅ Edge length/direction calculation preserved
- ✅ Graceful handling of missing edge files

## Open Questions

None. Implementation follows the graph transform design document (§10.1) precisely.
