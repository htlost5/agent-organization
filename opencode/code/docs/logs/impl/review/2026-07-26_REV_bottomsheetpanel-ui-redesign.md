---
agent: REV
task_id: TASK-BottomSheetPanel
date: 2026-07-26
status: pending
category: log
destination: logs/impl/review/
tags:
  - REV
  - BottomSheetPanel
  - UI-redesign
  - review
---

# Review Log: BottomSheetPanel UI Redesign

## Verdict: 条件付き承認 (Conditionally Approved)

## Verification Summary

| Requirement | Status | Notes |
|-------------|--------|-------|
| FR-1: BottomSheetPanel at screen bottom, 2 states, drag gesture, spring animation | ✅ | Gesture.Pan() + withSpring correctly implemented. pointerEvents="box-none" applied |
| FR-2: CollapsedBar with handle, floor display, collection status, 56px | ✅ | CollapsedBar renders handle, prev/next floor controls, status indicator. COLLAPSED_HEIGHT=56 |
| FR-3: ExpandedContent with ScrollView, FloorSelector, CollectionControls, CalibrationControls | ✅ | All controls rendered inside ScrollView. FloorSelector horizontal, circular 44x44 buttons |
| FR-4: UserLocation follows panel height + 10px, right-aligned | ✅ | useAnimatedStyle with panelHeight SharedValue + 10px bottom offset |
| FR-5: Bottom tab bar hidden, tab navigation still works | ✅ | visible="nothing" hides both header and bottom tabs |
| NFR-1: 60fps animation (reanimated worklet) | ✅ | useAnimatedStyle runs on UI thread. clamp is worklet-compatible |
| NFR-2: No memory leaks | ⚠️ | No obvious issues. useCallback deps correctly specified |
| NFR-3: Dark mode support | ✅ | All components use colorTheme from useMapContext() |
| NFR-4: Existing collection/calibration functionality preserved | ✅ | Controls moved inside panel but logic unchanged (absolute positioning removed) |
| NFR-5: TypeScript strict mode | ✅ | npx tsc --noEmit passes with 0 errors |

## Tools Check

| Tool | Result | Notes |
|------|--------|-------|
| npx tsc --noEmit | 0 errors | TypeScript strict mode passes |
| npx expo lint | 17 warnings | 1 new warning (CollapsedBar unused import) + 16 pre-existing |

## Issues Found

### Issue 1: Unused import in CollapsedBar.tsx
- **File**: `src/features/home/chrome/BottomSheetPanel/CollapsedBar.tsx` line 7
- **Severity**: Minor (lint warning)
- **Description**: `import { useMapContext } from "../../map/hooks/useMapContext"` is never used. The component receives `colorTheme` via props, so `useMapContext()` is never called.
- **Fix**: Remove the unused import line.

### Issue 2: No activeOffsetY on Pan gesture
- **File**: `src/features/home/chrome/BottomSheetPanel/index.tsx`
- **Severity**: Minor (recommendation)
- **Description**: `Gesture.Pan()` has no `activeOffsetY` set, meaning any vertical finger movement will activate the gesture. This could cause accidental panel dragging when the user intends to interact with content inside.
- **Suggestion**: Add `.activeOffsetY(-5, 5)` to require minimal vertical displacement before activation.

### Issue 3: Potential gesture conflict - Pan vs ScrollView
- **File**: BottomSheetPanel/index.tsx + ExpandedContent.tsx
- **Severity**: Minor (needs device testing)
- **Description**: The `GestureDetector` wraps the entire panel, while `ExpandedContent` contains a `ScrollView`. In RNGH v2, a parent Pan gesture can interfere with a child ScrollView's native scroll handling. When expanded, vertical dragging on the ScrollView content might activate panel drag instead of scroll.
- **Mitigation**: If observed, use `Gesture.Native()` for ScrollView or check `scrollOffset` before Pan activation.

## Observations

- useImperativeHandle deps correctly include panelHeight/maxHeight/minHeight
- pointerEvents="box-none" correctly applied for map pass-through
- CollectionControls/CalibrationControls absolute positioning removed correctly
- ColorTheme.controls properties all exist in type definition (colorPalette.ts)
- Dependencies verified: expo-haptics in package.json, FONT_SIZE.floorControl in typography.ts, useCollectionSession returns isActive/isPaused/isIdle

## Conclusion

実装は全体的に仕様通り。TypeScript エラーなし、アニメーションは worklet 準拠、テーマ対応、SafeArea 対応、既存機能維持のすべてを満たしている。

軽微な lint warning（未使用 import）の修正を条件に TST へ進める。
Issue 2（activeOffsetY）と Issue 3（gesture conflict）は実機確認が推奨されるが、TST をブロックするものではない。
