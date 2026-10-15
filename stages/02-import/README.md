# Stage 02 — Import · KICKOFF

**State:** `KICKOFF`. Threads only; planned into waves and sessions **after Stage 01 exits** (`plan-stage` skill), against the code and G1 rulings as they then stand.
**Horizon outcome:** 2 — "a real camera roll produces a sensible personal map; photos and videos are supported, duplicates avoided, missing metadata handled, flights shown as arcs, and inferred paths distinguished."
**Entry conditions:** Stage 01 `EXITED` (or `EXITED WITH CAVEATS` with no caveat touching reveal, fog or the store); F01 frozen; schema v1 with its migration harness.

## Objective

A user picks photos and videos, FootNoted reads where and when each was taken, orders and connects them into segments, and the map lights up. Flights appear as arcs; inferred lines look different from recorded ones; nothing is duplicated on re-import; items without location are handled in the open.

## Threads

- **Picking and permissions.** `PhotoViewPicker` multi-select is the main path. Reading photo location needs `ohos.permission.MEDIA_LOCATION` (photos are desensitized without it; videos are not), behind a rationale that names import. Stage 01's S01-13 and S01-50 evidence says what actually arrives.
- **Date-range scan — likely a product question.** A library-wide scan needs `ohos.permission.READ_IMAGEVIDEO`, a restricted permission available only for listed special scenarios. Options to put to the owner: drop the scan on HarmonyOS, replace it with a picker-side date filter if the platform offers one, or apply for the permission if a listed scenario genuinely fits. The charter calls the scan optional; changing it still needs an owner decision.
- **Metadata extraction.** Photo EXIF through Image Kit; video location and capture time through the media metadata APIs (field names and formats per S01-13). Time zone/local-time context when available. No native media frameworks that are not open source.
- **Identity, dedupe and access loss.** Dedupe on the platform asset identity chosen in Stage 01. Picked URIs may not stay readable after restart: history must survive asset access loss explicitly (keep observations and app-owned thumbnails; mark the asset unavailable; never silently erase).
- **Items without location.** Listed separately; pin manually (a manual observation with its own provenance) or skip.
- **Segmentation v1.** Time-gap break (candidate ~6 h) and implied-speed plausibility; great-circle above ~150 km/h; `none` when a gap is too large. Thresholds are open: calibrate with fixtures from `tool/journeygen` plus real libraries, and log the owner's ruling.
- **Rendering.** Route layer in the map's routes slot, inferred styling (dashed or lower opacity), photo markers with app-owned thumbnails (derived, rebuildable, stored in the protected area).
- **Scale.** Thousands of items: TaskPool batches, progress, cancellation, resumability after process death, a store write per accepted item.
- **Warning state.** "Every imported photo lacks location" must surface as a warning (missing permission or stripped files), not as an empty map.

## Open questions to settle at planning

- The date-range scan (above).
- Segment-break thresholds.
- Thumbnail size and storage budget; eviction is allowed only for derived thumbnails, never for history.
- Whether video import ships with photos or as a second wave.

## Likely shape (to be confirmed at planning)

- **W1:** metadata extraction + identity ‖ import pipeline + dedupe in the store ‖ segmentation v1 with fixtures ‖ picker UI + rationale.
- **W2:** route rendering + inferred styling ‖ markers/thumbnails ‖ large-library performance.
- **W3 (verification):** real libraries on the owner's phone, including photos from other devices, stripped location and videos.

## Compliance carried

Rationale before every prompt; nothing leaves the device; any later sharing strips location metadata by default and warns first; attribution stays visible with routes and markers.
