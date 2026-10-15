# Stage 05 — Statistics · KICKOFF

**State:** `KICKOFF`. Threads only; planned after Stage 04 exits (trip and place counts depend on it).
**Horizon outcome:** 5 — "explored area, repeat visits, counts, and administrative-area coverage."

## Objective

Show how much of the world the user has explored and how often they return, computed on the device from derived, rebuildable data.

## Threads

- **Explored area:** from the coverage representation ruled at G1, weighted by latitude, with overlaps counted once (Stage 01 already provides per-unit area).
- **Repeat visits:** the definition is an open product question (segment, day, trip or other). A day-based definition needs local-time context, which the store retains.
- **Counts:** trips, places, countries, cities.
- **Coverage percentage** of a city or country needs openly licensed administrative boundaries, bundled or downloaded with an offline-capable path; city polygons may need a second open source. Selection is open.
- **Depth features** (detailed stats, heatmaps) are earmarked for a future lifetime tier: build them behind `Entitlements` checks that return true; never test purchase state; never gate existing history.
- **Rebuild check extended to statistics:** the derived-state digest covers every statistic.

## Open questions to settle at planning

- The repeat-visit definition.
- Boundary datasets, resolution and app size.

## Likely shape

- **W1:** boundary dataset + attribution of coverage to areas ‖ statistics derivation + rebuild tests ‖ the visit definition.
- **W2:** statistics UI ‖ heatmap layer.
- **W3:** verification.

**Out:** year-in-review, posters, sharing (Later; sharing needs location stripping by default and a warning first).
