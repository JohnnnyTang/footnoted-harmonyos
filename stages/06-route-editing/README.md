# Stage 06 — Route editing · KICKOFF

**State:** `KICKOFF`. Threads only; planned after the previous stage exits. Depends functionally on Stage 02 (routes exist) and on Stage 01's provenance pipeline; schema v1 already holds vias, corrections and `is_user_edited`.
**Horizon outcome:** 6 — "vias, anchor corrections, undo, reset to automatic, and responsive recomputation after edits."

## Objective

Let the user fix any connection by hand. Every edit is reversible and never overwrites raw data (invariants 3, 4, 8).

## Threads

- **Vias:** add, move, delete on the map; touch targets sized for fingers (fat-fingered vias are the common case).
- **Anchor drag:** writes a correction record; the original observation is untouched; deleting corrections reverts.
- **Bounded recompute:** on drag end only, limited to the affected segment's bounding box; never per frame.
- **Undo:** per-segment undo stack; persistence across restarts is an open question.
- **Reset to automatic:** clears the segment's vias and corrections and drops `is_user_edited`; tested semantics.
- **Mode switch per segment** keeping vias (already proven at the data layer in Stage 01).
- **Split and merge** segments (from import's "user can split and merge").
- **Engine-specific gestures:** drag handling depends on the G1 engine (for example events crossing the ArkWeb bridge vs native gestures); prototype early.
- **Out:** live road-snapping while dragging (road routing, horizon 8); freehand drawing.

## Open questions to settle at planning

- Undo persistence.
- Gesture model and magnifier on small screens.

## Likely shape

- **W1:** editing interaction layer ‖ bounded recompute + performance ‖ undo/redo model.
- **W2:** split/merge ‖ reset and mode-switch UI.
- **W3:** phone verification of drag-end latency against a budget set at planning.
