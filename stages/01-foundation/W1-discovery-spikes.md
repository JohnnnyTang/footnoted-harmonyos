# Stage 01 · W1 — Discovery spikes

**Opens after:** S01-01 (verified toolchain and baseline). **Parallel sessions:** S01-10, S01-11, S01-12, S01-13 (one dispatch). **Closes into:** W2.
**Purpose:** get first HarmonyOS evidence for every area in [`DISCOVERY.md`](DISCOVERY.md) on the dev machine and the emulator (plus short phone steps), so that W2 spends phone time only on measuring the leading candidates.

Everything under `spike/` is throwaway. At G1 it is tagged `h01-spike` and removed from the tree. `tool/journeygen/` survives: later stages reuse the dataset for regression fixtures.

## Kickoff obligations (orchestrator, foreground, before dispatch)

1. Cut `stage/01-foundation` from `main`.
2. Baseline with the verified commands from `AGENTS.md`; record commands and counts in `notes/<date>-W1-kickoff.md`.
3. **Confirm the budgets** in `DISCOVERY.md` with the owner (or record the owner's standing instruction) before any measurement.
4. **Seams (implement once, read-only to sessions):**
   - `spike/FORMAT.md`, the shared data formats:
     - **Journey file** `*.journeys.ndjson`: a header line per segment `{"segment":<int>,"kind":"local"|"transit","mode":"straight"|"great_circle"|"none","buffer_m":<m|null>,"label":"…"}`, then one observation per line `{"ts":<epoch ms UTC>,"tz_min":<offset>,"lat":<deg>,"lon":<deg>,"acc":<m|null>,"src":"gps"|"media","seg":<int>}`, sorted by `ts`. `seg` is ground truth.
     - **Result file** `*.result.json`: `{"spike","candidate","device":{"model","os","api","emulator":bool},"build_mode","workload","method","metrics":{…},"ts"}`. Results never contain coordinates, cell or tile indices of user data.
   - Four spike projects copied from the S01-01 template, each with its **own bundle name** (`<dev-prefix>.spike.geo`, `.store`, `.map`, `.probe`) and a placeholder local test, so parallel installs cannot collide: `spike/geo/`, `spike/store/`, `spike/map/`, `spike/probe/`.
   - `tool/journeygen/` skeleton: `journeygen.mjs` (Node, no npm dependencies), `README.md`, a placeholder test runnable with `node --test`.
5. **Device lease.** Name which sessions may use the emulator and when the phone is available (S01-13's phone steps are booked with the owner). Record in the kickoff note.
6. Confirm ownership (below), then dispatch all four with the `run-session` skill.

## File ownership

| Path | Owner |
| --- | --- |
| `spike/FORMAT.md`, the four spike project skeletons' `AppScope/`, root `build-profile.json5`, `hvigor/` | **kickoff (frozen for W1)** |
| `tool/journeygen/**`, `spike/geo/**` (except the frozen files) | S01-10 |
| `spike/store/**` (except the frozen files) | S01-11 |
| `spike/map/**` (except the frozen files) | S01-12 |
| `spike/probe/**` (except the frozen files) | S01-13 |
| `stages/01-foundation/notes/*-S01-1n-*.md`, `stages/01-foundation/evidence/S01-1n/**` | each session, its own |

A session that needs a module-level dependency adds it to its own spike's module `oh-package.json5` and lock file, which it owns; the licence must be open and recorded in the note.

**Merge order:** S01-10 → S01-11 → S01-13 → S01-12. The dataset and geometry come first because W2 consumes them; the render spike goes last because it is the largest.

---

### S01-10 — Journey generator + grid coverage candidate

**Wave:** W1 · **Branch:** `s01/10-journeys-grid` · **Size:** L · **Depends on:** W1 seams

**Goal.** Produce the reproducible dataset (exit 2) and an ArkTS implementation of coverage candidate B1 (Web Mercator grid) good enough to count units, measure cost and test the hard geographies. Report what G1(b) and G1(d) need.

**Owns:** `tool/journeygen/**`, `spike/geo/**`.
**Reads:** `spike/FORMAT.md`, `DISCOVERY.md` §B and §D, `docs/PRODUCT.md` (Acceptance principles).

**Deliverables.**
1. `node tool/journeygen/journeygen.mjs --seed 42 --years 5 --out <dir>` writes `dataset.journeys.ndjson` and `dataset.stats.json`. Scenarios per exit criterion 2. Sampling for local movement: a fix every 150–400 m (the charter's "every few hundred metres"). Pure integer/seeded PRNG; no platform-dependent maths in what is hashed, or record the risk.
2. `spike/geo` (ArkTS, a HAR plus a minimal entry page): lon/lat ↔ Mercator at any level with latitude clamping; a compact unit id; supercover rasterisation; great-circle densification; antimeridian splitting (never drawn the long way round); metric dilation computed per row at that row's latitude; per-row run-length encoding; per-(unit, segment) export for S01-11 in a binary format documented in the spike's README.
3. Local tests: equator, 60°, an antimeridian crossing, |lat| > 84°, a zero-length segment (a point reveals a disk), and "dilated line ⊇ line".
4. A counts table for the dataset at two or three candidate levels: unique units, (unit, segment) records, runs, generation time on the host; and a device timing of the 1,000 km × 500 m leg on a TaskPool worker (emulator, labelled).
5. A short desk note on B2 (H3/S2 via native code) and B3 (vector corridors): what would have to be true for either to beat B1.

**Acceptance.**
- [ ] Local tests green from the command line (command + count).
- [ ] Two generator runs with `--seed 42` produce identical SHA-256 values (recorded).
- [ ] Counts table and timings present, each labelled host or emulator.

**Owner steps:** none.
**Out of scope:** storage, rendering, low-zoom aggregation beyond counting parents.
**Traps.** Use integer arithmetic at fine levels to avoid drift at tile edges; great-circle interpolation near the poles; ArkTS rules (no `any`, no object-literal types, no destructuring) per the skill's `arkts-language.md`; TaskPool `@Concurrent` functions cannot capture outer state.

---

### S01-11 — Protected storage benchmark

**Wave:** W1 · **Branch:** `s01/11-protected-store` · **Size:** L · **Depends on:** W1 seams

**Goal.** Establish what ArkData RDB can do for this product under encryption, and measure the coverage layouts, so G1(c) can rule.

**Owns:** `spike/store/**`.
**Reads:** `spike/FORMAT.md`, `DISCOVERY.md` §C. Until S01-10 merges, generate stand-in corridors yourself in the documented format; W2 swaps in the real export.

**Deliverables.**
1. **Capability probe** (instrument test on the emulator): `encrypt: true` with the default key and with custom key parameters (record the API level each needs); the file's first 16 bytes are not `SQLite format 3\0`; opening with a wrong key fails; `WITHOUT ROWID` tables; `CREATE VIRTUAL TABLE … USING rtree` through `executeSql` (record success or the exact error); journal mode and synchronous settings available.
2. **Key protection options:** how the key would be generated and held (RDB-managed vs HUKS vs Asset Store Kit), with the docs pages read.
3. **Layouts:** L1 one row per (unit, segment); L2 per-segment runs plus a per-unit count table. For each, with encryption on: bulk insert of one 1,000 km × 500 m leg; delete that segment (with count maintenance for L2); "covered units in one viewport tile" at three levels; size on disk; the same once with encryption off to report overhead. Writes inside one transaction; work on a TaskPool worker.
4. A time-boxed (≤ 1 h) feasibility note on C2 (SQLCipher via NDK): toolchain, ABIs, licences. Build it only if C1 fails a functional check.
5. Results as `*.result.json` saved in the sandbox and pulled with `hdc file recv`; nothing user-like in logs.

**Acceptance.**
- [ ] The capability table is filled from runs on the emulator, each item with its observed result.
- [ ] Layout timings and sizes present, labelled emulator.
- [ ] The note recommends layouts and key protection for W2, with reasons.

**Owner steps:** none.
**Out of scope:** the production schema (S01-32).
**Traps.** RDB needs a Context, so it runs as an instrument test, not a local test. Warm the cache before timing reads. Don't put the key in preferences or plain files even in the spike.

---

### S01-12 — Map engine survey + fog render benchmark

**Wave:** W1 · **Branch:** `s01/12-map-fog` · **Size:** L · **Depends on:** W1 seams

**Goal.** Find a map engine and a fog strategy that can meet the pan and fog-update budgets, and a repeatable frame-time method on HarmonyOS.

**Owns:** `spike/map/**`.
**Reads:** `spike/FORMAT.md`, `DISCOVERY.md` §A, the charter's Compliance section (attribution).

**Deliverables.**
1. **Engine survey with evidence:** search ohpm and the OpenHarmony third-party ecosystem for open map renderers runnable on HarmonyOS NEXT; record what exists, licence, maintenance, API level and ABIs. Record Map Kit as rejected (invariant 1).
2. **A1 prototype:** an ArkWeb page with a pinned MapLibre GL JS build bundled in `rawfile` (licence recorded), a stock dark style from an open source, attribution visible. List every runtime host the spike contacts in the note.
3. **Fog strategies (a) and (b)** from `DISCOVERY.md` behind a toggle, fed from ArkTS through the Web bridge; optionally (c). Measure bridge/serialisation time separately from draw time. Use a synthetic clustered coverage set until S01-10's export lands.
4. **Scripted pan:** a fixed camera path at a mid zoom over a dense area and a long corridor, 20 s × 3.
5. **`spike/map/MEASURE.md`:** a frame-time method from documented tools, with exact commands verified on the emulator, and what each method actually measures (Web compositor vs ArkUI frames).
6. Optional, time-boxed: an A2 prototype if the survey finds a viable native renderer.

**Acceptance.**
- [ ] The spike runs on the emulator; the pan and the frame-time capture produce numbers (labelled emulator, no pass/fail).
- [ ] `MEASURE.md` lets someone else reproduce the numbers without reading the code.
- [ ] The note recommends the engine and strategy for W2, with reasons and risks.

**Owner steps:** none.
**Out of scope:** the production style fork (S01-33), the production fog controller (S01-41), the database.
**Traps.** Emulator GPUs are not representative. Large payloads through the bridge dominate; measure them. Keep attribution in every screenshot.

---

### S01-13 — Platform probes: located media, recording lifecycle, encryption areas

**Wave:** W1 · **Branch:** `s01/13-platform-probes` · **Size:** M · **Depends on:** W1 seams; S01-E3 for the phone steps

**Goal.** Observe, not assume, how HarmonyOS behaves for the three platform questions the foundation and Stages 02–03 depend on.

**Owns:** `spike/probe/**`.
**Reads:** `DISCOVERY.md` §C (encryption areas), §E, §F.

**Deliverables.**
1. **Located media:** pick photos and videos with `PhotoViewPicker`; read location and capture time with and without `MEDIA_LOCATION` (rationale shown first); record fields, formats and API levels; the identity to use for dedupe and whether it is stable across re-picks and restarts; whether picked URIs remain readable after a restart.
2. **Recording lifecycle:** foreground location updates at "a fix every few hundred metres" settings; a location-type continuous task; each accepted fix appended durably as it arrives (a small store in the chosen candidate area). On the phone: screen off for at least 20 minutes, screen lock, removal from Recents, force-stop and relaunch, each with its observed outcome. No `LOCATION_IN_BACKGROUND`.
3. **Encryption areas:** from the continuous task with the screen locked, try create/write/read in EL2, EL3 and EL5 (with and without `acquireAccess`) for a plain file and an RDB store; record results.
4. A findings list for the Stage 02 and Stage 03 kickoff threads.

**Acceptance.**
- [ ] A results table per probe, each row labelled emulator or phone, with the observed behaviour.
- [ ] Every permission requested shows a rationale first and is declared with `reason` and `usedScene`.
- [ ] No coordinate, note or media metadata appears in `hilog` output (command + grep result).

**Owner steps:** lock and unlock the phone, remove the app from Recents, carry the phone on a short walk with the screen off. The agent records what the owner reports as owner-observed.
**Out of scope:** import UI, recording UI, any production code.
**Traps.** A Recents clear skips `onDestroy`; never rely on it. Continuous tasks that do not match their declared type are suspended or killed (`continuous-task` page).
