# Stage 01 — Foundation (native discovery + foundation)

**Stage opened:** 2026-10-15 (planning, S01-00). **Branch:** `stage/01-foundation`, cut from `main` by the W1 kickoff.
**State:** `ACTIVE`. W0 planning is done; the bootstrap (S01-01) waits on the owner's toolchain (S01-E1, S01-E2).
**Horizon outcome:** 1 — "a dark world map with visible attribution, protected local history, reversible reveal, and correct recovery/rebuild behavior" ([PRODUCT.md](../../docs/PRODUCT.md#full-product-horizon)). It also carries Stage 00's discovery steps 2–7, including its proof flow.
**Discovery brief:** [`DISCOVERY.md`](DISCOVERY.md) (alternatives, measurements, acceptance per area).
**Backlog:** [`VERIFICATION_BACKLOG.md`](VERIFICATION_BACKLOG.md) (`S01-V###`, `S01-E#`).
**Waves:** [W0](W0-plan-and-bootstrap.md) · [W1](W1-discovery-spikes.md) · [W2](W2-measure-and-decide.md) · [W3](W3-building-blocks.md) · [W4](W4-pipeline-and-fog.md) · [W5](W5-integrate-and-verify.md)

## Objective

Pick the native architecture on HarmonyOS evidence rather than on the reference project's answers, then build the foundation every later outcome stands on: a DevEco-generated ArkTS/ArkUI app, a dark world map with attribution that is always visible, a protected local store that holds every kind of information the charter says must be retained, a geometry and coverage library with per-segment provenance, reveal and rebuild, and fog over the map.

At the end of the stage the proof flow from Stage 00 works on the owner's phone: **pick one located photo, reveal it alongside fixture journeys, keep it across a restart, delete its segment, and see the fog restored exactly without erasing an overlapping journey.** This proof is not the whole product; the full horizon stays in view.

## Exit criteria

Tags: **[host]** Hypium Local Test or host script on the Windows dev machine (`hvigorw test` is documented as unsupported on Linux) · **[emu]** run by an agent on the DevEco emulator; never decides a performance criterion · **[phone]** run by an agent over `hdc` on the owner's HarmonyOS phone · **[owner]** only the owner can do it · **[doc]** a written artifact.

**Discovery and decisions (gate G1)**

1. **[doc] Toolchain and targets recorded.** DevEco Studio, Command Line Tools/hvigor, ohpm, hdc, JDK and Node versions; `compatibleSdkVersion`, `targetSdkVersion` and the compile SDK recorded **separately**; emulator image; target phone model, HarmonyOS version and API; a temporary development bundle name, labelled as not a distribution identity. Build and test commands in `AGENTS.md` and `README.md` are ones that actually ran.
2. **[host][doc] Reproducible journeys.** A seeded generator produces about five years of synthetic journeys covering the charter's acceptance cases: a dense home city with repeated, overlapping commutes and walks; at least three drives of 1,000 km or more; at least ten flights including one across the antimeridian; a stay above 60° latitude and a near-pole leg; an overnight gap; two unrelated trips on adjacent days. Same seed → byte-identical output (hash recorded).
3. **[doc] Discovery brief complete.** Every area in `DISCOVERY.md` (map/fog, coverage representation, protected persistence, compute placement, located media, recording lifecycle) has its alternatives measured or rejected with labelled evidence and cited Huawei sources.
4. **[phone][doc] Responsiveness on the target.** A map-pan budget is written into `DISCOVERY.md` **before** measuring, derived from HarmonyOS experience standards, and ruled at G1. With the full dataset revealed, a scripted pan on the owner's phone is measured against it (build mode, method, OS/API recorded). Emulator numbers are context only.
5. **[doc] G1 ruled.** Written owner rulings, each citing its numbers, on (a) the map engine and tile/style source, (b) the coverage representation and resolution, (c) persistence, encryption, key protection, encryption area and coverage layout, (d) compute placement (ArkTS/TaskPool vs native), (e) the fog approach and zoom → level mapping. The **F01 contract** is frozen. `spike/` is tagged `h01-spike` and removed from the tree.

**Foundation**

6. **[host][emu][phone] App builds and runs.** A debug HAP builds from the command line with the recorded command, installs and launches on the emulator and the phone. Every user-visible string comes from resources (D-005 policy as carried by H-004); legal credit text lives in resources too and is never translated or paraphrased.
7. **[phone] Map with identity.** A dark, visually quiet world map from openly licensed tiles and style. Attribution is **always visible** on the map ("© OpenStreetMap contributors" linked to the ODbL copyright page, plus the selected tile source's credit from the charter's Compliance section) and repeated on an About/Licences screen alongside the open-source package licences.
8. **[host][phone] Protected local store.** Schema v1 gives a home to **every** item under "Information the product must retain", including those whose UI comes later. Raw observation records are insert-only. The store is encrypted at rest (a test proves the file is not readable as plaintext SQLite), the key is protected by the platform, and a missing or wrong key fails loudly. The store lives in an encryption area that stays writable during a screen-locked recording (G1(c), with evidence). A migration harness exists with a v1 baseline test.
9. **[host] Geometry and coverage library.** Straight and great-circle connections, antimeridian splitting, metre-based reveal at the real latitude, low-zoom aggregation and fog geometry, with tests at **the equator, 60°, across the antimeridian, near the poles**, and for a zero-length segment.
10. **[host][phone] Provenance and rebuild.** Adding, editing and deleting a segment returns all derived state **exactly** to its prior value. Deleting one of two overlapping journeys keeps the other's coverage. Dropping all derived data and rebuilding from raw observations plus edits reproduces an identical derived-state digest (F01.7). Switching connection mode preserves vias and corrections. A correction never alters the original observation.
11. **[phone] Fog correct at every zoom** from the world view to street level, including across the antimeridian, reading coverage only through the F01.4 reader, off the UI thread.
12. **[phone][owner] Proof flow.** After a rationale screen, the user picks one located photo. Its coordinates are read with `ohos.permission.MEDIA_LOCATION` granted and stored as captured; a photo whose location is absent or withheld is reported honestly, never silently placed. The photo and the fixture journeys reveal; the history survives an app restart and removal from Recents; deleting the photo's segment restores the fog exactly without erasing an overlapping journey. Automated in an instrument test where the platform allows, with screenshot pairs on the phone.
13. **[host][doc] Guards and truthful docs.** Guards fail on a non-open dependency licence, an undeclared or background-location permission, a runtime host missing from `docs/network-hosts.md`, and coordinates/notes/keys reaching `hilog`. The `Entitlements` service (every check true) and the first-use timestamp exist. `invariant-review` passes on the stage diff. `README.md`, `AGENTS.md`, `docs/decisions.md`, `docs/network-hosts.md` and this README's exit table are true.

## The F01 contract (frozen at G1 unless noted)

The cross-module seams W3–W5 build against. A frozen clause changes only through a new H-row.

| Clause | State | Contract |
| --- | --- | --- |
| F01.1 Raw observations | **charter-derived; frozen at the W3 kickoff** | GPS fixes, media items and notes are stored as captured: original coordinates, capture time and local-time context, accuracy/speed when supplied, source identity, links to sessions or media. Insert-only; no code path updates their coordinates. A correction is a separate record; the latest correction wins for display; deleting corrections restores the original. Media carry a platform asset reference used for dedupe (exact identity chosen from S01-13 evidence). |
| F01.2 Coverage unit and provenance | proposal until G1 (b)(c) | The coverage unit (grid cell, polygon or other) and its identity; the logical record is **(unit, segment)**; the physical layout; a read rule naming which tables fog and statistics may read. |
| F01.3 Low-zoom aggregation | proposal until G1 (b)(e) | Coarser levels derived from F01.2, rebuilt rather than hand-maintained; which levels exist. |
| F01.4 `CoverageReader` | proposal until G1 (e) | The only API the fog renderer may use: an async "covered units in this tile at this level" query plus a change notification for invalidation. Never runs on the UI thread. |
| F01.5 Display geometry | proposal; frozen at G1 | `SegmentGeometry { segmentId, mode, vertices, inferred }`, derived from member observations (corrections applied) plus ordered vias. `road` is unsupported in Stage 01 and falls back to straight with `inferred = true`. `none` draws no line; the points reveal on their own. |
| F01.6 Buffer | **product values decided (H-004)**; distance rule frozen at G1 (b) | Effective buffer = the segment's override ?? default(kind). Local default 100 m; overrides clamped to 25 m – 2 km; transit reveals its endpoints only, each with the local default. Distances are metric at the actual latitude, never a fixed angular radius. |
| F01.7 Derived-state digest | proposal; frozen at the W4 kickoff | A canonical digest over every derived table (geometry, coverage, aggregation, statistics) used by rebuild and delete-restores-baseline tests. |

## Waves at a glance

```
W0 ──▶ W1 (4 parallel) ──▶ W2 (integrate, then phone) ──▶ G1 ──▶ W3 (4 parallel) ──▶ W4 (3 parallel) ──▶ W5 (wire, then verify) ──▶ exit
plan +  discovery spikes     real dataset on the            owner   building blocks      pipeline + fog +      proof flow on the phone
bootstrap                    leading candidates             rules                        guards
```

| Wave | Purpose | Sessions | Owner involvement |
| --- | --- | --- | --- |
| [W0](W0-plan-and-bootstrap.md) | Plan; toolchain, targets and DevEco scaffold | **S01-00** planning (done) · **S01-01** bootstrap (foreground) | Install DevEco; Huawei ID; phone; provisional minimum API |
| [W1](W1-discovery-spikes.md) | Discovery spikes, first evidence | **S01-10** journeys + grid candidate · **S01-11** protected storage · **S01-12** map engine + fog render · **S01-13** platform probes (media, recording, encryption areas) | Short phone sessions for S01-13 (lock, Recents, a short walk) |
| [W2](W2-measure-and-decide.md) | The leading candidates on the real dataset; then gate **G1** | **S01-20** integrated spike (emulator) → **S01-21** phone measurement · *S01-22 conditional remediation* | Phone attached; **rule G1** |
| [W3](W3-building-blocks.md) | Foundation building blocks against frozen seams | **S01-30** app shell · **S01-31** geometry library · **S01-32** protected store · **S01-33** map view + style + attribution | Confirm module layout and UI-state H-rows at kickoff |
| [W4](W4-pipeline-and-fog.md) | Derived pipeline, fog, guards | **S01-40** reveal + provenance + rebuild · **S01-41** fog renderer · **S01-42** guards + CI | none |
| [W5](W5-integrate-and-verify.md) | Wire end to end; verify the exit | **S01-50** wiring + dev harness + located-photo path → **S01-51** exit verification + docs | Phone handling (restart, Recents, screenshots) |

**17 sessions** (plus one conditional) across six waves.

### Session index

| ID | Title | Size | Wave | Depends on |
| --- | --- | --- | --- | --- |
| S01-00 | Planning | L | W0 | — |
| S01-01 | Toolchain, targets, DevEco scaffold, agent config | M | W0 | S01-E1, S01-E2 |
| S01-10 | Journey generator + grid coverage candidate | L | W1 | W1 seams |
| S01-11 | Protected storage benchmark | L | W1 | W1 seams |
| S01-12 | Map engine survey + fog render benchmark | L | W1 | W1 seams |
| S01-13 | Platform probes: located media, recording lifecycle, encryption areas | M | W1 | W1 seams, S01-E3 for the phone steps |
| S01-20 | Integrated spike on the real dataset + device driver | M | W2 | W1 merged |
| S01-21 | Phone measurement | S | W2 | S01-20, S01-E3 |
| S01-22 | *(conditional)* performance remediation | M | W2 | a missed budget |
| S01-30 | App shell (navigation, theme, resources, About, entitlements, rationale) | M | W3 | G1 |
| S01-31 | Geometry and coverage library | L | W3 | G1 |
| S01-32 | Protected store, schema v1, migrations | L | W3 | G1 |
| S01-33 | Map view, dark style, attribution | M | W3 | G1 |
| S01-40 | Reveal pipeline, provenance, rebuild | L | W4 | W3 merged |
| S01-41 | Fog renderer | L | W4 | W3 merged |
| S01-42 | Guards + CI | M | W4 | W3 merged |
| S01-50 | End-to-end wiring, dev harness, located-photo path | M | W5 | W4 merged |
| S01-51 | Exit verification + docs | M | W5 | S01-50 |

## Risks carried into the stage

| Risk | Where it is tested | Fallback |
| --- | --- | --- |
| No open map renderer runs natively on HarmonyOS NEXT out of the box (the Huawei docs archive has no MapLibre/OSM renderer guidance; Map Kit is proprietary and excluded by invariant 1). | S01-12, G1(a) | A Web-based map in ArkWeb as an explicit architecture choice (H-row), or a native port/renderer with its cost stated. |
| The ArkTS ↔ Web bridge is too slow for fog geometry during pans. | S01-12, S01-20 | Pre-aggregated levels, fewer larger shapes, a custom GPU layer, or moving fog into the page from bundled data. |
| ArkData RDB lacks a spatial index (R\*Tree is not documented in the archive) or custom-key encryption at the chosen minimum API. | S01-11, G1(c) | Grid-key indexes in plain tables; SQLite/SQLCipher via NDK and NAPI with the key in the platform keystore. |
| ArkTS is too slow for rasterise/dilate/merge on long legs. | S01-10, S01-20, G1(d) | TaskPool parallelism, Sendable data, or C++ through NAPI. |
| Encryption area: EL4 data cannot be created while locked and EL2 is the documented default; a screen-locked recording must still write. | S01-13, G1(c) | Choose the area from probe evidence (EL2, or EL3/EL5 semantics) and record it. |
| Photos read without `MEDIA_LOCATION` are desensitized (EXIF location erased); videos are not ([docs](https://developer.huawei.com/consumer/cn/doc/doccenter-capabilities/medialibrary-asset-hidesensitive-faq)). | S01-13, S01-50 | Rationale + permission before reading; report items without location honestly. |
| Agents cannot log in to a Huawei ID; some emulators need a logged-in account and cannot start from the CLI. | S01-01 | The owner starts the emulator and performs signing; agents use `hdc` against what is running. |
| One emulator and one phone shared by parallel sessions. | every wave | Device lease rule and distinct spike bundle names (stages/README.md). |
| Local Tests do not run on Linux, so a Linux CI cannot run them. | S01-01, S01-42 | Host tests are a Windows-dev-machine gate until a CI shape is ruled (H-row). |
| Reference-project findings mislead. | all | They are hypotheses only; every decision cites HarmonyOS evidence. |

## Current status

- **2026-10-15. W0 / S01-00 planning done.** Read the charter, decisions, transfer record and the reference project's process; wrote this plan (README, discovery brief, W0–W5 briefs, backlog), the kickoff threads for Stages 02–06, the stage conventions and templates, and the four process skills under `.claude/skills/`. Logged H-005. No toolchain is installed on the dev machine yet (`hvigorw`, `ohpm` and `hdc` are not on PATH), so nothing was built, run or measured.
- **Next:** S01-01 bootstrap (foreground orchestrator + owner), then the W1 kickoff.

## Blockers that still bind

- `S01-E1` — DevEco Studio (and, if wanted, Command Line Tools) installed on the dev machine.
- `S01-E2` — the owner's Huawei ID for debug signing and emulator login.
- `S01-E3` — the owner's HarmonyOS phone (model, HarmonyOS version, API, RAM) with developer mode and USB debugging. Needed for S01-13's phone steps, S01-21 and W5; not for S01-01.
- `S01-E4` — a provisional minimum API for development. The launch market and final minimum stay open product questions.
