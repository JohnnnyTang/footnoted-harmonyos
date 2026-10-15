# Stage 01 — Discovery brief

The alternatives, measurements and acceptance criteria for each open technical area (Stage 00 step 3). W1 spikes fill in the evidence; W2 measures the leading candidates on the real dataset; the owner rules at G1. Nothing here is decided until a G1 H-row says so.

**Evidence rules.** Label every number with device model, HarmonyOS version/API, build mode, workload and method. Emulator and host numbers inform; only the target phone decides a performance criterion. The reference project's measurements are hypotheses, never evidence. Cite the Huawei page actually read (slug or URL) and the API level the fact applies to. A candidate that breaks an invariant is rejected regardless of speed.

## Budgets (set before measuring)

Proposed by S01-00 from the HarmonyOS experience standards distilled in the harmonyos-app-lifecycle skill (`testing-quality.md` §5); the W1 kickoff confirms or amends them, and G1 rules on the measured results.

| Budget | Proposed value | Source / reasoning |
| --- | --- | --- |
| Map pan with full fog, scripted, target phone | jank rate ≤ 5 ms/s and no run of consecutive dropped frames during the drag | Official swipe standard (jank ≤ 5 ms/s; 0 consecutive dropped frames for swipe). Map panning is a drag gesture; no map-specific standard was found. |
| Fog update after the camera settles | ≤ 500 ms to visibly correct fog for the new viewport | Proposal; no official figure. The map must feel like exploration, not a loading screen. |
| UI-thread blocking | no single I/O or compute slice > 8.3 ms on the UI thread | AppAnalyzer heuristic (no UI-thread I/O > 8.3 ms). |
| Reveal one long leg (1,000 km, 500 m buffer) | ≤ 2 s off the UI thread on the target phone | Proposal; an import of thousands of items must finish in minutes, not hours. |
| Cold start to a usable map | ≤ 1,100 ms launch-to-loaded | Official cold-start standard for apps. Measured, not necessarily met, in Stage 01. |

## A. Map engine and fog (G1 (a), (e)) — S01-12

**Question.** What open renderer draws a dark, quiet world map on HarmonyOS NEXT, keeps attribution visible, and draws fog fast enough at every zoom?

| Candidate | What it is | Must verify |
| --- | --- | --- |
| A1 Web map in ArkWeb | The ArkUI `Web` component hosting MapLibre GL JS (BSD-3-Clause), bundled in the app (no runtime CDN), with an openly licensed vector style and tiles. | Bridge cost (ArkTS → page) for fog geometry; memory; gesture and overlay composition with ArkUI; offline asset loading; WebGL availability on the emulator and phone. **A Web-based map is an explicit architecture choice (decisions.md) and needs its own H-row.** |
| A2 Native open renderer | An existing open-source HarmonyOS/OpenHarmony port of a vector map renderer (e.g. MapLibre Native) drawn into an `XComponent` through NAPI. | Whether such a port exists today, its licence, maintenance, API level and ABI coverage. Do not assume it exists. |
| A3 Own renderer | Vector or raster tiles drawn with ArkUI Canvas / `XComponent` + OpenGL ES. | Effort; label rendering; whether an openly licensed, permitted tile source fits. Fallback only. |
| ✗ Map Kit / Petal Maps | Huawei's map service SDK. | Rejected: proprietary SDK (invariant 1). |

**Tile and style sources** (each needs a `docs/network-hosts.md` entry before the app uses it): OpenFreeMap (hosted, keyless, OpenMapTiles schema) and Protomaps (PMTiles, self-hosted or bundled extracts). Record licence, credit text and offline behaviour.

**Fog strategies** to compare on the winning engine: (a) per-tile polygons with holes for covered areas; (b) the inverse, dark shapes over uncovered runs only; (c) a custom GPU layer or raster mask (the soft-fog path, noted for later). Measure serialisation and bridge time separately from draw time.

**Acceptance.** A measured fog strategy that meets the pan and fog-update budgets on the phone with the full dataset; attribution visible in every state; no runtime host outside `docs/network-hosts.md`; a reproducible frame-time method written in `spike/map/MEASURE.md` from documented tools (for example DevEco Profiler frame analysis or PerfTest where applicable).

## B. Coverage representation (G1 (b)) — S01-10, S01-20

**Question.** What unit records "revealed by segment S" so that reveal follows the corridor (invariant 5), provenance is exact (6), deletion restores the map, area accounts for latitude, and fog draws fast?

| Candidate | What it is | Must verify |
| --- | --- | --- |
| B1 Web Mercator grid cells | Fixed-level square tiles; supercover rasterisation; metric dilation computed per row at its latitude; run-length runs per row for storage. | Unit count and storage for the dataset at two or three candidate levels; dilation error vs the requested metres; antimeridian and near-pole behaviour; cost in ArkTS. |
| B2 Hierarchical global grid (H3 or S2) | Library cells via native code (both Apache-2.0). | Native build for the HarmonyOS ABIs; cover cost for a long buffered leg; exact parents for aggregation; fog vertex cost. Desk evaluation first; prototype only if B1 fails. |
| B3 Vector corridors | Per-segment buffered polygons; union for display and area. | Union cost on overlaps (the home city); exact delete; area with latitude; fog vertex counts at low zoom. |

**Acceptance.** Delete-restores-baseline and rebuild-identical are exact, not approximate; the 1,000 km leg meets its budget; storage for the five-year dataset is small enough for a phone (number reported, budget ruled at G1); fog read cost fits the pan budget.

## C. Protected persistence (G1 (c)) — S01-11, S01-13

**Question.** Where and how does the app keep raw observations, edits, structure and derived data encrypted at rest, with the key protected by the platform, fast enough for bulk reveal and fog reads?

| Candidate | What it is | Must verify |
| --- | --- | --- |
| C1 ArkData RDB, encrypted | `relationalStore` with `encrypt: true`; custom key parameters exist (`cryptoParam`; `rekeyEx` from API 22 per the `data-encryption` page). | Default vs custom key and the API level of each; whether the file header is plaintext; WITHOUT ROWID and R\*Tree via `executeSql` (R\*Tree is not mentioned anywhere in the docs archive); transaction and batch insert speed; large read speed; behaviour while the screen is locked. |
| C2 SQLite + SQLCipher via NDK | A NAPI module with SQLCipher (BSD-3) and its crypto backend; key kept in HUKS or Asset Store Kit. | Build effort for the phone and emulator ABIs; licences; same functional checks as C1. Time-boxed; pursue only if C1 fails. |
| C3 Hybrid | RDB for raw, edit and structure tables; derived coverage in encrypted binary files. | Rebuildability; atomicity across the two stores; encryption of the files. |

**Encryption area** (sandbox `EL1`–`EL5`, `app-sandbox-directory` page): EL2 is the documented default and stays readable after first unlock; EL4 cannot be created or read while locked; EL3 can create but not read new files while locked; EL5 needs `acquireAccess`. A recording with the screen locked must keep writing, so the area is chosen from S01-13's locked-screen probe.

**Acceptance.** File not readable as plaintext SQLite; wrong or missing key fails loudly; the key never appears in logs or files outside platform protection; locked-screen writes succeed in the chosen area; insert, delete and tile-read timings reported for each layout; size on disk reported.

## D. Compute placement (G1 (d)) — S01-10, S01-20

**Question.** Does rasterise/dilate/aggregate/merge run fast enough in ArkTS on a TaskPool worker, or does it need C++ through NAPI?

**Measure** the 1,000 km × 500 m leg and a dense home-city tile on the phone: ArkTS on TaskPool (with Sendable data where it avoids copies) vs, only if the budget is missed, a native prototype. Record serialisation cost between threads separately (AppAnalyzer heuristic: TaskPool/Worker (de)serialisation ≤ 8 ms).

**Acceptance.** The chosen placement meets the reveal budget without blocking the UI thread; native code, if chosen, has a stated ABI, build and licence story.

## E. Located media (input to Stage 02 and exit 12) — S01-13

**Question.** How does a picked photo or video yield its original location and capture time, what identity survives re-picking and restarts for dedupe, and what happens when access to the asset is lost?

**Facts to build on.** `PhotoViewPicker` returns user-selected assets. Without `ohos.permission.MEDIA_LOCATION` (a user grant), photo EXIF location is desensitized; videos are not (`medialibrary-asset-hidesensitive-faq`). `ohos.permission.READ_IMAGEVIDEO` is a restricted permission available only for listed special scenarios (`restricted-permissions`), which bears on the charter's optional date-range library scan (a Stage 02 product question).

**Acceptance.** For photos with and without the permission, and for videos: which fields arrive, in what form, at which API level; the identity used for dedupe and whether it is stable across restarts; whether picked URIs stay readable after a restart; how capture time and local-time context are obtained.

## F. Recording lifecycle (input to Stage 03) — S01-13

**Question.** What keeps an explicit recording alive with the screen off, what survives process death, and what does the platform actually do on Recents removal, power saving and reboot?

**Facts to build on.** A continuous task (`backgroundTaskManager.startBackgroundRunning`, location type) needs `ohos.permission.KEEP_BACKGROUND_RUNNING`; the system suspends or kills apps whose background work does not match the declared type (`continuous-task` constraints). Location needs `APPROXIMATELY_LOCATION` (+ `LOCATION` for precise). `LOCATION_IN_BACKGROUND` belongs to passive mode only and is never declared in Stage 01. A Recents clear skips `onDestroy`, so nothing may depend on a final callback.

**Acceptance.** A short screen-off probe on the phone with each accepted fix written durably as it arrives; observed behaviour for screen lock, Recents removal, force-stop and relaunch recorded as observed, never assumed.
