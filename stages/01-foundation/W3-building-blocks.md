# Stage 01 · W3 — Building blocks

**Opens after:** G1 is ruled and F01.2–F01.6 are frozen (F01.1 freezes at this kickoff). **Parallel sessions:** S01-30, S01-31, S01-32, S01-33 (one dispatch). **Closes into:** W4.
**Purpose:** build four independent foundations (app shell, geometry library, protected store, map view) against frozen seams, so W4 only composes them.

Briefs name G1 outcomes as "per G1(x)". The kickoff rewrites any brief whose wording a ruling changed, and records the change in its note.

## Kickoff obligations (orchestrator, foreground)

1. Rebase; baseline with the verified commands; confirm `spike/` is gone and tagged.
2. **Owner rulings (H-rows):**
   - **Module layout** (proposed): `app/entry` (HAP: UI shell and features), `app/geo` (HAR: pure ArkTS geometry and coverage, no `@kit` imports so it tests on the host), `app/store` (HAR: ArkData store and repositories), `app/map` (HAR: the map engine wrapper, style assets and attribution overlay). HAR rather than HSP because there is a single HAP.
   - **UI state and navigation** (proposed): state management V2 (`@ComponentV2`, `@Local`, `@Param`, `@ObservedV2`/`@Trace`) and `Navigation` with a route map, the docs' recommendation for new apps. One generation only.
   - Freeze **F01.1** as written in the stage README.
3. **Seams (implement once; frozen for W3):**
   - The four modules registered in `app/build-profile.json5`, each with its `oh-package.json5`; **every W3 dependency added up front** with the lock file committed, licences checked by hand and listed in the kickoff note. Sessions never edit these files; a missing dependency is out of contract.
   - `app/geo/src/main/ets/Types.ets` (frozen): `LatLng`, the coverage-unit identity per F01.2, `TileId(z, x, y)`, `ConnectionMode`, `SegmentKind`, the H-004 buffer constants (`DEFAULT_LOCAL_BUFFER_M = 100`, `MIN_BUFFER_M = 25`, `MAX_BUFFER_M = 2000`).
   - `app/store/src/main/ets/AppMetaStore.ets` (frozen): an interface with `read(key)` and `write(key, value)`. S01-30 uses an in-memory implementation; S01-50 wires the store's.
   - `app/entry/src/main/resources/base/element/string.json`, **owned by S01-30**, pre-filled with the map credit strings (legal text, never translated or paraphrased; the charter's Compliance wording) and the credit URLs in a frozen `app/map/src/main/ets/MapCredits.ets`. Other sessions list the strings they need in their note; the orchestrator adds them at merge.
   - `app/entry/src/main/module.json5`, **owned by S01-30**: only `ohos.permission.INTERNET` in W3 (for tiles).
   - A placeholder `FootMapView` component in `app/map` with its final constructor parameters; S01-33 replaces the body.
4. Dispatch all four.

## File ownership

| Path | Owner |
| --- | --- |
| `app/build-profile.json5`, all `oh-package.json5` files, the lock file, `Types.ets`, `AppMetaStore.ets`, `MapCredits.ets` | **kickoff (frozen)** |
| `app/entry/**` (except frozen files), `app/AppScope/**` | S01-30 |
| `app/geo/**` (except `Types.ets`) | S01-31 |
| `app/store/**` (except `AppMetaStore.ets`) | S01-32 |
| `app/map/**` (except `MapCredits.ets`), `docs/network-hosts.md` | S01-33 |

**Merge order:** S01-31 → S01-32 → S01-30 → S01-33.

---

### S01-30 — App shell

**Branch:** `s01/30-app-shell` · **Size:** M

**Goal.** Turn the template into the FootNoted shell: navigation, a dark theme that matches the map, resources for every string, About/Licences, `Entitlements`, first-use recording and a reusable permission flow. No location or media permission yet.

**Deliverables.**
1. `EntryAbility` + a `Navigation` root with routes: home (embedding `FootMapView`), About/Licences, and a dev page reachable only in debug builds (S01-50 fills it).
2. A dark theme; the app label **FootNoted** (capital N, never an asterisk) in `AppScope` and the entry module.
3. Every user-visible string from `string.json`; `en_US` as the shipped locale and the base resources ready for later languages (English-only launch, H-004).
4. About/Licences: version, map credits with links, and a licences list including the map data (ODbL), the tile schema/source and style licences, and each open-source package (text taken from the upstream files, never paraphrased).
5. `Entitlements` with `isAvailable(feature)` returning true for everything; a `Feature` enum listing the future gated items (cloud backup/sync, road routing, offline packs, detailed stats, heatmaps, advanced export, posters). Nothing tests purchase state.
6. `FirstUseRecorder`: writes `first_use_at` (epoch ms UTC) through `AppMetaStore` if absent, never overwrites.
7. `PermissionFlow`: a rationale page naming the feature (title, purpose, Continue / Not now) → `checkAccessToken` → `requestPermissionsFromUser` → on denial, an explanation and `requestPermissionOnSetting` when the user asks. Ready for Stage 02/03; unit-tested with a fake access-token layer.
8. Lifecycle hygiene: nothing saved in `onDestroy`/`onBackground`; listeners registered and released symmetrically.

**Acceptance.**
- [ ] Local tests for `Entitlements`, `FirstUseRecorder` (idempotent across two "launches") and `PermissionFlow` (granted, denied, denied-then-settings) green (command + count).
- [ ] A UI test on the emulator: About shows both credit lines, each opens its link; the dev page is absent in a release build.
- [ ] A grep finds no user-visible string literals in `.ets` UI code (command recorded).
- [ ] Debug HAP builds and launches.

**Owner steps:** none. **Out of scope:** the store, map internals, real permission declarations.

---

### S01-31 — Geometry and coverage library

**Branch:** `s01/31-geo` · **Size:** L

**Goal.** `app/geo` becomes the complete, tested implementation of F01.2's unit maths, F01.3, F01.5's geometry and F01.6, hardened from the S01-10 spike per G1(b)(d).

**Deliverables.**
1. Projection and unit maths; supercover rasterisation; great-circle densification; antimeridian splitting; metric dilation per F01.6 (with the near-pole refinement S01-10 noted).
2. `SegmentGeometry` derivation (F01.5) from observations, corrections (latest wins) and ordered vias, for `straight`, `great_circle` and `none`; `road` falls back to straight with `inferred = true`.
3. Low-zoom aggregation per F01.3 (from a sorted unit list; add/remove deltas for the pipeline).
4. Fog geometry helpers per G1(e) (for example rectangle merge within a tile at a level) with exact-union tests.
5. Area of a unit at its latitude (for Stage 05); set operations on sorted unit lists.
6. If G1(d) chose native compute: the NAPI module and its build, owned here.

**Acceptance.**
- [ ] Local tests green, including the equator, 60°, the antimeridian, near the poles, a zero-length segment, aggregation round-trips (sum of children = count) and fog-geometry exactness (union == input, no overlaps).
- [ ] Line coverage of `app/geo` reported from the local-test coverage report.
- [ ] Phone or emulator timing of the 1,000 km × 500 m leg vs its budget (labelled).

**Owner steps:** none. **Out of scope:** storage, UI.

---

### S01-32 — Protected store, schema v1, migrations

**Branch:** `s01/32-store` · **Size:** L

**Goal.** `app/store` ships schema v1 for **every** information item the charter retains, encrypted per G1(c), with platform-protected keys and a migration harness every later stage extends.

**Deliverables.**
1. Schema v1:
   - **Source (insert-only):** observations (GPS and media points), media (platform asset reference, unique for dedupe; original metadata as captured; kind; capture time and local-time context; thumbnail reference; imported-at), notes, note–media links, recording sessions.
   - **Edits:** location corrections (latest wins), ordered vias, per-segment mode and buffer choices, `is_user_edited`.
   - **Structure:** trips (including empty ones, status `suggested`/confirmed, origin auto/user), places, segments (kind, mode, buffer override, return-of link), segment membership, tags and their links.
   - **Derived (rebuildable):** segment geometry, coverage per F01.2, aggregation per F01.3, per-unit statistics.
   - **Meta:** `app_meta`.
   - Deletion rules written down: deleting a segment removes its derived rows and memberships and **never** observations or media.
2. Opening: the store is created in the encryption area ruled at G1(c), with encryption on and the key per G1(c); it refuses to open unencrypted and fails loudly on a wrong key.
3. `AppMetaStore` over `app_meta`.
4. Migrations: a version table and an upgrade path pattern, with a v1 baseline test and a documented recipe for v2 in `app/store/README.md`.
5. Thin repositories for W4/W5: insert/get observations, media (dedupe on the asset reference), segments with members and vias, corrections (insert-only; no update path for observations).

**Acceptance.**
- [ ] Instrument tests green on the emulator: the file is not plaintext SQLite (first 16 bytes), the wrong key fails, a correction leaves the observation byte-identical, the cascade rules hold, re-inserting a media item with the same asset reference does not duplicate, the v1 baseline test.
- [ ] The key never appears in logs (grep of `hilog` output during the tests).
- [ ] A locked-screen write test on the phone, or a backlog row naming it for S01-51.

**Owner steps:** none (the phone check can move to S01-51). **Out of scope:** reveal logic (S01-40).

---

### S01-33 — Map view, dark style, attribution

**Branch:** `s01/33-map` · **Size:** M

**Goal.** The product's identity on the engine ruled at G1(a): a quiet dark style, a `FootMapView` with a small controller API the fog and routes use, and attribution that is always visible.

**Deliverables.**
1. A dark style forked from an openly licensed source (upstream URL, revision/date and licence recorded): POI clutter removed, place and road labels legible, and a **layer order contract** — base → fog → routes/markers → labels — with the insertion slots documented in `app/map/README.md`.
2. `FootMapView` + `FootMapController`: style loading, fog and route insertion hooks at the contracted slots, camera helpers (fly to, fit bounds, current zoom, visible bounds), world-copy behaviour decided and recorded.
3. An always-visible attribution overlay (not hidden behind an info button): the OSM credit linked to `https://www.openstreetmap.org/copyright` and the tile-source credit per the charter, from `MapCredits.ets` and resources.
4. `docs/network-hosts.md`: every runtime host (tiles, style, glyphs, sprites), with purpose, operator, data sent (including viewport/tile coordinates), consent, licence/terms evidence and offline/error behaviour.

**Acceptance.**
- [ ] A style test: valid JSON, the fog slot sits below the first label layer, and every URL host is listed in `docs/network-hosts.md`.
- [ ] A UI test: both credit lines present and unclipped on a small phone viewport.
- [ ] Emulator and phone screenshots of the dark map in `evidence/S01-33/`.

**Owner steps:** none. **Out of scope:** fog logic (S01-41).
