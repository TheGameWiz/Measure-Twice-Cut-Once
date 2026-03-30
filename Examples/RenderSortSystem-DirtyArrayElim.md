# Codex Prompt — RenderSortSystem Dirty Array Elimination

**Created:** 2026-03-24
**Revised:** 2026-03-24 (v1)
**For:** Codex AI agent (self-contained, no additional context required)

---

## Project Context

- Unity 6.2 DOTS/ECS project, mobile-first (60+ FPS mandatory)
- Code style: K&R/Egyptian braces, dense whitespace, no decorative blank lines
- `AGENTS.md` + `TASKS.md` + `LexiWord-Docs/Live Session State.md` are the execution-policy source of truth
- Load `CLAUDE.md` only if explicitly requested or needed to resolve an architecture/policy conflict

---

## Efficiency Principles (NON-NEGOTIABLE)

1. **Every byte justifies its existence.** If data is derivable from existing state, derive it — don't store a parallel copy.
2. **Eliminate dirty-array anti-patterns.** A parallel array that the main thread writes and later reads to decide whether to call a function is deferred work that should happen at the source or not at all. The exception: arrays that feed another processor (GPU, Burst job) or improve cache locality for multithreaded access.
3. **Cardinality must match the problem.** A 5120-element array tracking a global state change (focus transition affects all nodes simultaneously) is the wrong data structure. Use a scalar or derive from existing per-entity data.
4. **Don't duplicate state Unity already tracks.** `GameObject.activeSelf` already knows whether a GO is active. A parallel bool array tracking the same thing is redundant.

---

## Problem Being Solved

`RenderSortSystem.cs` (`Assets/Scripts/Systems/RenderSortSystem.cs`, 1092 lines) maintains **6 per-slot static arrays** sized to the pool (5120 entries). Analysis shows 5 of the 6 are eliminable:

| Array | Type | Size | Why Redundant |
|-------|------|------|---------------|
| `s_LastPortalIconType` | `byte[]` | ~5 KB | **Dead state.** Written at lines 471/476 but never read for any comparison. Portal icon change detection already uses direct renderer checks (`portalGO.activeSelf != showPortalIcon`, `portalRenderer.sprite != resolvedSprite`). |
| `s_LabelCulled` | `bool[]` | ~5 KB | **Duplicates Unity state.** Tracks whether a label was deactivated by camera culling. For non-Hidden nodes (Hidden hits `continue` at line 896), the only path that calls `RootGO.SetActive(false)` is the culling gate itself. `!labelRefs.RootGO.activeSelf` is equivalent. |
| `s_LabelDataInitialized` | `bool[]` | ~5 KB | **Redundant with `s_LastNodeState`.** Every path that sets `s_LabelDataInitialized = false` also sets `s_LastNodeState = UnknownNodeState` — except the culling deactivation path (line 916). Adding `s_LastNodeState = UnknownNodeState` to that one path makes `nodeStateChanged` cover all cases that `!s_LabelDataInitialized` was covering. |
| `s_LastLabelTreasure` | `bool[]` | ~5 KB | **Dirty anti-pattern, wrong cardinality.** Guards `ApplyLabelVisuals` which sets 4 color properties. TMP_Text.color and SpriteRenderer.color have internal early-outs (`SetPropertyUtility.SetColor` compares before marking dirty). The array saves nothing that Unity doesn't already check. |
| `s_LastLabelDimFactor` | `float[]` | ~20 KB | **Dirty anti-pattern, wrong cardinality.** Same guard as treasure. dimFactor is derived from `displayColor` written by FocusDimmingSystem. During focus transitions (~20-30 frames), dimFactor changes for ALL visible nodes every frame — the dirty check fires for every node anyway. During steady state, Unity's internal color early-outs handle it. Heaviest array at 20 KB. |

**Kept:**
- `s_LastNodeState` (`byte[]`, ~5 KB) — load-bearing. Detects Active→Completed transitions. Absorbs `s_LabelDataInitialized`'s role via sentinel `UnknownNodeState`.
- `s_DirtyLabelArray` (`Entity[64]`, 512 bytes) — legitimate cross-system event queue. Fixed size, bounded, drained each frame.

**Total eliminated: ~40 KB** (from ~45 KB down to ~5.5 KB).

---

## Additional Change: Cull Radius Simplification

The camera culling radius is currently computed as:
```
maxLabelScale = max(labelDefaultScale, labelFocusedCompleteScale, labelFocusedIncompleteScale)
labelCullRadius = LabelCullBaseRadius * maxLabelScale + LabelCullPadding
                = 0.6 * 2.0 + 0.75 = 1.95
```

The actual node visual extent is `GameConstants.BaseNodeScale * 0.5f = 0.625f` (unit quad scaled by 1.25, half-extent is 0.625). The current formula keeps labels active ~1.3 units beyond the sphere edge — triple the actual visual radius. Labels don't extend beyond the sphere in any meaningful way.

Replace the entire formula with `GameConstants.BaseNodeScale * 0.5f`. Delete `LabelCullBaseRadius` and `LabelCullPadding` constants. Delete the `maxLabelScale` computation at line 760. The `FocusVisualSettings` label scale values (`labelFocusedCompleteScale`, `labelFocusedIncompleteScale`) are still needed for the scale animation system below — only their use in the cull radius formula is removed.

---

## Design

### s_LastPortalIconType — Delete, No Replacement

Remove the field declaration (line 62), all writes (lines 471, 476), the reset in `ResetLabelRuntimeStateForPoolIndexInternal` (line 276), the clear in `InvalidateLabelCaches` (line 234), the resize in `EnsurePoolCaches` (line 139), and the bounds checks in `IsValidPoolIndexForLabelCaches` (lines 262-263). No behavioral replacement needed — the renderer comparisons already do the job.

### s_LabelCulled — Replace Reads with !RootGO.activeSelf

Three reads become direct GO state checks:
- Line 913: `if (!s_LabelCulled[poolIndex])` → `if (labelRefs.RootGO.activeSelf)`
- Line 931: `else if (s_LabelCulled[poolIndex])` → `else if (!labelRefs.RootGO.activeSelf)`
- Line 938: `else if (s_LabelCulled[poolIndex])` → `else if (!labelRefs.RootGO.activeSelf)`

All writes (lines 927, 935, 275) become unnecessary. Remove the field, resize, bounds check, and clear entries.

The safety net at line 967 (`if (!needsFullLabelUpdate && !labelRefs.RootGO.activeSelf)`) already uses activeSelf directly and continues to catch any unexpected inactive label.

### s_LabelDataInitialized — Merge into s_LastNodeState

Add `s_LastNodeState[poolIndex] = UnknownNodeState` to the culling deactivation path (currently at line 916, where `s_LabelDataInitialized[poolIndex] = false` is set). This ensures that when a culled node returns to view, `nodeStateChanged` fires and triggers `needsFullLabelUpdate`.

Then remove `!s_LabelDataInitialized[poolIndex]` from the `needsFullLabelUpdate` expression (line 966). The expression becomes:
```
bool needsFullLabelUpdate = isFocusedActive || nodeStateChanged || isDirty;
```

Remove `s_LabelDataInitialized[poolIndex] = true` at line 1032 — no longer needed since `s_LastNodeState` is written at line 1033 and provides the equivalent "initialized" signal.

Remove the field, resize, bounds check, and clear entries.

### s_LastLabelTreasure + s_LastLabelDimFactor — Delete, Call ApplyLabelVisuals Unconditionally

Replace the conditional block (lines 1076-1082):
```csharp
bool visualsChanged = s_LastLabelTreasure[poolIndex] != isTreasure ||
                      math.abs(s_LastLabelDimFactor[poolIndex] - dimFactor) > 0.001f;
if (visualsChanged) {
    ApplyLabelVisuals(labelManager, labelRefs, poolIndex, isTreasure, dimFactor);
    s_LastLabelTreasure[poolIndex] = isTreasure;
    s_LastLabelDimFactor[poolIndex] = dimFactor;
}
```

With an unconditional call:
```csharp
ApplyLabelVisuals(labelManager, labelRefs, isTreasure, dimFactor);
```

`ApplyLabelVisuals` sets 4 color properties. Each has an internal early-out (TMP_Text.color via `SetPropertyUtility.SetColor`, SpriteRenderer.color via native check). During steady state, no vertices are dirtied and no material updates fire. During transitions, the colors genuinely change and must be written anyway.

Remove both field declarations, all writes (lines 925, 926, 273, 274), resize entries, bounds checks, and clear entries.

### ApplyLabelVisuals — Remove Unused poolIndex Parameter

The `poolIndex` parameter (line 503) is passed to `ApplyLabelVisuals` but never used in the function body. Remove it from the signature and from the call site.

### Cull Radius — Simplify to Node Extent

At line 761, replace:
```csharp
float labelCullRadius = LabelCullBaseRadius * maxLabelScale + LabelCullPadding;
```
With:
```csharp
float labelCullRadius = GameConstants.BaseNodeScale * 0.5f;
```

Delete line 760 (`float maxLabelScale = ...`). Delete constants `LabelCullBaseRadius` (line 34) and `LabelCullPadding` (line 35).

The `labelFocusedCompleteScale` and `labelFocusedIncompleteScale` locals remain — they're used by the scale animation system (lines 1044-1047). Only their participation in the cull radius formula is removed.

---

## Cascading Cleanup

### EnsurePoolCaches (lines 115-145)
- Early-exit guard: check only `s_LastNodeState` (the sole remaining dynamic array)
- Resize: only `ResizeByteCache` for `s_LastNodeState`
- The scratch buffer allocations (`s_Scratch`, `s_CursorOnBuffer`, `s_CursorOffBuffer`) remain unchanged

### IsValidPoolIndexForLabelCaches (lines 251-265)
- Reduces to checking `s_LastNodeState` only: `return poolIndex >= 0 && s_LastNodeState != null && poolIndex < s_LastNodeState.Length`
- Consider inlining this into `ResetLabelRuntimeStateForPoolIndexInternal` since it's now a single expression

### ResetLabelRuntimeStateForPoolIndexInternal (lines 268-284)
- Only resets `s_LastNodeState[poolIndex] = UnknownNodeState` (plus scale slot cleanup which remains unchanged)

### InvalidateLabelCaches (lines 218-248)
- Remove clears for: `s_LabelDataInitialized`, `s_LastLabelTreasure`, `s_LastLabelDimFactor`, `s_LabelCulled`, `s_LastPortalIconType`
- Keep: `s_LastNodeState` fill loop (lines 235-237), scale slot resets, dirty count clear, camera valid flag, cached focus resets

### Dead Helper Functions
- `ResizeBoolCache` (lines 162-171): only callers were `s_LabelDataInitialized`, `s_LabelCulled`, `s_LastLabelTreasure` — all deleted. **Delete this function.**
- `ResizeFloatCache` (lines 173-187): only caller was `s_LastLabelDimFactor` — deleted. **Delete this function.**
- `ResizeStringCache` (lines 147-160): **zero callers even before this refactor.** Already dead code. **Delete this function.**
- `ResizeByteCache` (lines 189-203): still called for `s_LastNodeState`. **Keep.**

---

## Scope Boundary

### In Scope
- Remove 5 array declarations: `s_LastPortalIconType`, `s_LabelCulled`, `s_LabelDataInitialized`, `s_LastLabelTreasure`, `s_LastLabelDimFactor`
- Remove 2 constants: `LabelCullBaseRadius`, `LabelCullPadding`
- Remove 3 helper functions: `ResizeBoolCache`, `ResizeFloatCache`, `ResizeStringCache`
- Simplify `EnsurePoolCaches`: single array guard + single resize
- Simplify `IsValidPoolIndexForLabelCaches`: single array check (or inline into caller)
- Simplify `ResetLabelRuntimeStateForPoolIndexInternal`: single reset + scale slots
- Simplify `InvalidateLabelCaches`: remove 5 array clear blocks
- Simplify cull radius: replace formula with `GameConstants.BaseNodeScale * 0.5f`, delete `maxLabelScale` computation on line 760
- Culling gate (lines 908-941): replace `s_LabelCulled` reads with `!labelRefs.RootGO.activeSelf`, add `s_LastNodeState[poolIndex] = UnknownNodeState` in deactivation path, remove all `s_LabelCulled`/`s_LabelDataInitialized`/`s_LastLabelTreasure`/`s_LastLabelDimFactor` writes
- `needsFullLabelUpdate` expression (line 966): remove `!s_LabelDataInitialized[poolIndex]` term
- Remove `s_LabelDataInitialized[poolIndex] = true` at line 1032
- Visuals block (lines 1076-1082): replace conditional with unconditional `ApplyLabelVisuals` call
- `ApplyLabelVisuals` signature: remove unused `poolIndex` parameter
- Portal icon writes (lines 471, 476): remove `s_LastPortalIconType` writes

### Out of Scope — DO NOT TOUCH
- `s_LastNodeState` — kept, load-bearing
- `s_DirtyLabelArray` / `s_DirtyLabelCount` / `MarkLabelDirty` — kept, legitimate event queue
- `ApplyLabelTextAndChildren` logic (lines 370-500) — only removing the two `s_LastPortalIconType` writes from within it; all other label text/portal/secondary logic is untouched
- Scale animation system (`ScaleSlot`, `UpdateLabelScale`, `s_ScaleSlot0`/`s_ScaleSlot1`, `s_LastKnownLabelDefaultScale`) — untouched
- Cursor blink system (`UpdateCursorBlink`, `s_CursorVisible`, cursor buffers) — untouched
- Solver display name cache (`s_CachedSolverDisplayNames`, `RefreshSolverDisplayNameCache`) — untouched
- `FocusVisualSettings` reads (lines 751-757) — these remain; only the `maxLabelScale` line derived from them is removed
- `FocusDimmingSystem` — no changes
- `WordLabelManager` — no changes
- `GameConstants` — no changes (reading `BaseNodeScale`, not modifying it)
- Any file other than `RenderSortSystem.cs`

### Invariants That Must Be Preserved
1. **Camera culling still works.** Nodes outside the orthographic frustum (expanded by the node's visual radius) still have their label RootGO deactivated. The `IsInsideExpandedBounds` function and `TryGetOrthographicCameraBounds` are untouched — only the radius value passed in changes.
2. **Culled nodes still skip processing.** The camera-steady skip (`continue` at line 939) still fires for nodes whose RootGO is inactive (the activeSelf check replaces the array check with identical semantics).
3. **Returning-to-view nodes get full rebuild.** When a culled node becomes visible again, `s_LastNodeState` is `UnknownNodeState` → `nodeStateChanged = true` → `needsFullLabelUpdate = true`. This triggers the same full label rebuild that `!s_LabelDataInitialized` previously triggered.
4. **Hidden nodes still full-rebuild on return.** `ResetLabelRuntimeStateForPoolIndexInternal` sets `s_LastNodeState = UnknownNodeState`. When the node transitions from Hidden to Active, `nodeStateChanged` fires.
5. **Portal icons still render correctly.** The direct renderer checks (`portalGO.activeSelf`, `portalRenderer.sprite != resolvedSprite`) are the actual change-detection mechanism and remain untouched.
6. **Label colors still update correctly.** `ApplyLabelVisuals` is called unconditionally for every visible, non-culled node. TMP_Text.color and SpriteRenderer.color have internal early-outs that prevent unnecessary vertex/material dirtying when values haven't changed.
7. **The safety net (line 967) still catches inconsistent state.** Any non-hidden, non-culled node with `!RootGO.activeSelf` that reaches line 967 still triggers `ResetLabelRuntimeStateForPoolIndexInternal` + forced rebuild.
8. **Focused nodes bypass culling.** The culling gate is guarded by `!isFocused` (line 908). The focused node is never culled regardless of position.
9. **`needsFullLabelUpdate` still covers all necessary cases.** `isFocusedActive` (cursor blink), `nodeStateChanged` (state transitions + initialization via sentinel), and `isDirty` (external events via MarkLabelDirty) — these three terms cover everything `s_LabelDataInitialized` was covering plus all previous cases.

### Risk Areas
- **Culling gate activeSelf semantics.** The replacement assumes that for non-Hidden nodes reaching the culling gate, `!RootGO.activeSelf` means "was camera-culled." Verify there is no other path between the Hidden check (line 889) and the culling gate (line 908) that could deactivate RootGO. Currently there isn't — but if future code adds one, the activeSelf check would misidentify it as "culled."
- **ApplyLabelVisuals call cost.** Calling unconditionally for ~1000 visible nodes per frame adds ~4000 color property setter calls (with internal early-outs). On low-end mobile this is negligible compared to the 25 KB memory savings and eliminated per-frame array comparisons, but measure if frame time increases.
- **`maxLabelScale` removal.** Verify that `labelFocusedCompleteScale` and `labelFocusedIncompleteScale` are only used for scale animation (lines 1044-1047) and the deleted cull formula. They should have no other consumers.

---

## Step Ordering

### Step 1: Delete dead state and helper functions
- Remove field declarations: `s_LastPortalIconType` (line 62), `s_LabelCulled` (line 49), `s_LabelDataInitialized` (line 46), `s_LastLabelTreasure` (line 60), `s_LastLabelDimFactor` (line 61)
- Remove constants: `LabelCullBaseRadius` (line 34), `LabelCullPadding` (line 35)
- Remove helper functions: `ResizeBoolCache` (lines 162-171), `ResizeFloatCache` (lines 173-187), `ResizeStringCache` (lines 147-160)
- Remove `s_LastPortalIconType` writes in `ApplyLabelTextAndChildren` (lines 471, 476)

### Step 2: Simplify infrastructure functions
- `EnsurePoolCaches`: reduce early-exit guard to check only `s_LastNodeState`, remove 5 resize calls, keep only `ResizeByteCache` for `s_LastNodeState` and the scratch buffer allocations
- `IsValidPoolIndexForLabelCaches`: reduce to `s_LastNodeState` check only (or inline into `ResetLabelRuntimeStateForPoolIndexInternal`)
- `ResetLabelRuntimeStateForPoolIndexInternal`: remove 5 array resets, keep only `s_LastNodeState[poolIndex] = UnknownNodeState` and the scale slot cleanup
- `InvalidateLabelCaches`: remove 5 array clear blocks, keep `s_LastNodeState` fill loop, scale slot resets, dirty count clear, camera flag, cached focus resets

### Step 3: Simplify cull radius
- Delete line 760 (`float maxLabelScale = math.max(...)`)
- Replace line 761 (`float labelCullRadius = LabelCullBaseRadius * maxLabelScale + LabelCullPadding`) with `float labelCullRadius = GameConstants.BaseNodeScale * 0.5f`
- This requires adding `using LexiWord.Data;` if not already present (it is — line 10)

### Step 4: Rewrite culling gate
- Replace `s_LabelCulled` reads at lines 913, 931, 938 with `labelRefs.RootGO.activeSelf` / `!labelRefs.RootGO.activeSelf`
- In the deactivation block (around line 916): add `s_LastNodeState[poolIndex] = UnknownNodeState`, remove writes to `s_LabelDataInitialized`, `s_LastLabelTreasure`, `s_LastLabelDimFactor`, `s_LabelCulled`

### Step 5: Simplify needsFullLabelUpdate and visuals
- Line 966: remove `!s_LabelDataInitialized[poolIndex]` from the `needsFullLabelUpdate` OR expression
- Line 1032: remove `s_LabelDataInitialized[poolIndex] = true`
- Lines 1076-1082: replace conditional `visualsChanged` check + guarded call with unconditional `ApplyLabelVisuals(labelManager, labelRefs, isTreasure, dimFactor)`

### Step 6: Clean up ApplyLabelVisuals signature
- Remove unused `poolIndex` parameter from `ApplyLabelVisuals` (line 503) and its call site

### Step 7: Final verification
- Search the entire file for any remaining references to deleted fields/constants/functions
- Verify no compile errors from removed references
- Verify `s_LastNodeState` is the only remaining dynamic per-slot array (besides `s_DirtyLabelArray` which is fixed-size)

---

## Validation Expectations

After implementation:
1. No remaining references to `s_LastPortalIconType`, `s_LabelCulled`, `s_LabelDataInitialized`, `s_LastLabelTreasure`, `s_LastLabelDimFactor`, `LabelCullBaseRadius`, `LabelCullPadding`, `ResizeBoolCache`, `ResizeFloatCache`, `ResizeStringCache` anywhere in the file.
2. `EnsurePoolCaches` only resizes `s_LastNodeState` (plus scratch buffers).
3. `InvalidateLabelCaches` only clears `s_LastNodeState` (plus scale slots, dirty count, camera flag, cached focus).
4. `ResetLabelRuntimeStateForPoolIndexInternal` only writes `s_LastNodeState[poolIndex] = UnknownNodeState` (plus scale slot cleanup).
5. Culling gate uses `labelRefs.RootGO.activeSelf` for all cull-state decisions.
6. Culling deactivation path sets `s_LastNodeState[poolIndex] = UnknownNodeState` (ensuring full rebuild on return to view).
7. `needsFullLabelUpdate = isFocusedActive || nodeStateChanged || isDirty` — three terms, no initialization check.
8. `ApplyLabelVisuals` is called unconditionally for every visible non-culled node, with no `poolIndex` parameter.
9. Cull radius is `GameConstants.BaseNodeScale * 0.5f` with no label-scale or padding computation.
10. File compiles with no errors or warnings related to these changes.
11. All out-of-scope systems (scale animation, cursor blink, solver cache, dirty label queue) are unchanged.

--- SPEC DIVIDER: Everything above is the spec. Everything below is the log. ---

--- Agent 1 | Pass 1 | Design | 2026-03-25 09:00 ---
Verified all spec claims against RenderSortSystem.cs (1092 lines). Findings:

**All line references accurate:**
- s_LastPortalIconType declared line 62, written lines 471/476, never read for comparison ✓
- s_LabelCulled declared line 49, reads at 913/931/938, writes at 927/935/275 ✓
- s_LabelDataInitialized declared line 46, set false at 272/916, set true at 1032, read at 966 ✓
- s_LastLabelTreasure declared line 60, s_LastLabelDimFactor declared line 61, used at 1076-1082 ✓
- ApplyLabelVisuals at line 503: poolIndex parameter confirmed unused in body (lines 504-526) ✓
- Cull radius at lines 760-761: maxLabelScale=2.0, result=1.95. BaseNodeScale=1.25 (GameConstants.cs:58), half-extent=0.625 ✓
- ResizeStringCache (147-160) zero callers, ResizeBoolCache (162-171) only deleted-array callers, ResizeFloatCache (173-187) only s_LastLabelDimFactor caller ✓

**s_LabelCulled → activeSelf equivalence verified:**
- Between Hidden check (line 889, hits continue at 896) and culling gate (line 908), no path deactivates RootGO. Only lines 899-906 execute (nodeStateChanged, isFocused, transform read). So for non-Hidden nodes reaching the culling gate, !RootGO.activeSelf ↔ was-camera-culled.
- Safety net at line 967 still catches any unexpected inactive label.

**s_LabelDataInitialized → s_LastNodeState merger sound:**
- Adding UnknownNodeState at the culling deactivation (line 916) ensures nodeStateChanged fires on return-to-view, covering the same case s_LabelDataInitialized covered.
- ResetLabelRuntimeStateForPoolIndexInternal already sets UnknownNodeState (line 277), so the Hidden→Active path is covered too.

**No spec changes needed.** Spec is complete and accurate. Ready for Agent 2 review.
!NEXT: Agent 2

--- Agent 2 | Pass 2 | Design | 2026-03-25 08:25 ---
Review verdict: ready.
Re-checked the revised prompt end-to-end against workflow constraints and execution risks.

- Scope boundary is explicit and preserves single-file change limits.
- State-migration plan (s_LabelDataInitialized -> s_LastNodeState) is complete, including cull-deactivate sentinel reset.
- s_LabelCulled replacement with RootGO.activeSelf preserves cull skip semantics and keeps the existing safety-net path intact.
- Unconditional ApplyLabelVisuals rationale is technically sound, and the residual performance risk is already documented for measurement.
- Step ordering is dependency-safe and ends with a full symbol-sweep verification pass.

No revisions requested.
!NEXT: Agent 1

--- Agent 1 | Pass 3 | Design | 2026-03-25 09:10 ---
Agent 2 verdict: ready. Spec verified against source, no revisions needed. Collaboration complete.
!NEXT: collab-done

--- Agent 2 | Pass 4 | Design | 2026-03-25 08:26 ---
Acknowledgment: !NEXT: collab-done received from Agent 1 after ready verdict and final signoff. Agent 2 closing workflow per completion contract.
!NEXT: collab-done

--- Agent 1 | Pass 5 | Implement | 2026-03-25 09:30 ---
Director approved transition to Implement. Spec is frozen from this point forward.

Execute the spec in full, following the 7-step ordering exactly as written. Single file: `Assets/Scripts/Systems/RenderSortSystem.cs`.

Key reminders:
- Step 4 (culling gate): the `s_LastNodeState[poolIndex] = UnknownNodeState` addition in the deactivation path is the critical semantic bridge — don't miss it.
- Step 5: `needsFullLabelUpdate` reduces to three terms: `isFocusedActive || nodeStateChanged || isDirty`.
- Step 6: remove `poolIndex` from both `ApplyLabelVisuals` signature and call site.
- Step 7: grep the entire file for all deleted symbols to confirm zero remaining references.

Report what you changed, any surprises, and the final line count.
!NEXT: Agent 2

--- Agent 2 | Pass 6 | Implement | 2026-03-25 09:42 ---
Implemented the spec end-to-end in `Assets/Scripts/Systems/RenderSortSystem.cs`.

Changes applied:
- Deleted 5 per-slot arrays: `s_LastPortalIconType`, `s_LabelCulled`, `s_LabelDataInitialized`, `s_LastLabelTreasure`, `s_LastLabelDimFactor`.
- Deleted cull constants `LabelCullBaseRadius` and `LabelCullPadding`.
- Deleted dead helpers `ResizeStringCache`, `ResizeBoolCache`, `ResizeFloatCache`; retained `ResizeByteCache`.
- Simplified `EnsurePoolCaches(...)` to guard/resize only `s_LastNodeState` (scratch/cursor buffers unchanged).
- Simplified `InvalidateLabelCaches(...)` to clear only `s_LastNodeState` (plus existing scale-slot/dirty/focus resets).
- Simplified `IsValidPoolIndexForLabelCaches(...)` to check only `s_LastNodeState`.
- Simplified `ResetLabelRuntimeStateForPoolIndexInternal(...)` to reset only `s_LastNodeState` (plus existing scale/focus slot cleanup).
- Removed `s_LastPortalIconType` writes from `ApplyLabelTextAndChildren(...)`.
- Simplified cull radius to `GameConstants.BaseNodeScale * 0.5f`; removed `maxLabelScale` computation.
- Rewrote culling gate to use `labelRefs.RootGO.activeSelf` directly (no `s_LabelCulled`), and added `s_LastNodeState[poolIndex] = UnknownNodeState` in the cull deactivation path.
- Reduced full-rebuild predicate to `isFocusedActive || nodeStateChanged || isDirty`.
- Removed `s_LabelDataInitialized[poolIndex] = true` write.
- Replaced guarded visuals dirty-check block with unconditional `ApplyLabelVisuals(...)` call.
- Removed unused `poolIndex` parameter from `ApplyLabelVisuals(...)` signature and call site.

Verification:
- Grep sweep confirms zero remaining references to all deleted fields/constants/helpers.
- Remaining dynamic per-slot array is `s_LastNodeState` only (`s_DirtyLabelArray` remains fixed-size event queue).
- Final line count: 988.

Surprises/open items:
- No implementation surprises.
!NEXT: Agent 1

--- Agent 1 | Pass 7 | Test | 2026-03-25 09:55 ---
Director will test in Editor. While waiting, fix this minor issue:

**Redundant activeSelf check (line 835-838):**
```
else if (!labelRefs.RootGO.activeSelf) {
    if (!labelRefs.RootGO.activeSelf)   // ← redundant, outer else-if already guarantees this
        labelRefs.RootGO.SetActive(true);
}
```
Collapse to:
```
else if (!labelRefs.RootGO.activeSelf) {
    labelRefs.RootGO.SetActive(true);
}
```
The inner `if` is a leftover from when the outer check was `s_LabelCulled` and the inner was `activeSelf`. Now both test the same thing.

Report the fix, then wait for Director test results.
!NEXT: Agent 2

--- Agent 2 | Pass 8 | Test | 2026-03-25 09:56 ---
Applied the requested Test-phase cleanup in `Assets/Scripts/Systems/RenderSortSystem.cs`:
- Collapsed the redundant nested `if (!labelRefs.RootGO.activeSelf)` inside the `else if (!labelRefs.RootGO.activeSelf)` reactivation block.
- Reactivation path is now a single direct call:
  - `else if (!labelRefs.RootGO.activeSelf) labelRefs.RootGO.SetActive(true);`

No other changes in this pass.
!NEXT: Agent 1

--- Agent 1 | Pass 9 | Test | 2026-03-25 10:05 ---
COMPLETION:
Director tested in Unity Editor — all visual behaviors confirmed correct: camera culling, focus labels, dim colors, return-to-view rebuild. Approved for completion.

Summary: Eliminated 5 of 6 per-slot dirty arrays (~40 KB), 3 dead helper functions, 2 constants. Simplified cull radius formula. RenderSortSystem reduced from 1092 to ~986 lines.
!NEXT: Agent 2

--- Agent 2 | Pass 10 | Test | 2026-03-25 10:06 ---
Completion acknowledged. Director-approved Test completion received after Unity validation.
!NEXT: done
