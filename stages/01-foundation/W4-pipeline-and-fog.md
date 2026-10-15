# Stage 01 · W4 — Reveal pipeline, fog, guards

**Opens after:** W3 merged. **Parallel sessions:** S01-40, S01-41, S01-42 (one dispatch). **Closes into:** W5.
**Purpose:** compose the building blocks into the derived pipeline and the fog, and turn the invariants into failing checks.

## Kickoff obligations (orchestrator, foreground)

1. Rebase; baseline; record it.
2. **Seams (frozen for W4):**
   - **F01.4** `CoverageReader` interface in `app/geo/src/main/ets/CoverageReader.ets` (pure types, so the map module depends on `app/geo` only): the async tile/level query and the change notification. S01-40 implements it in `app/store`; S01-41 consumes it, using a fake until S01-40 merges.
   - **F01.7** the derived-state digest: a `DerivedStateDigest` interface (canonical ordering, hash algorithm) in `app/store`. S01-40 implements it.
   - `app/entry/src/main/ets/dev/fixtures/`: the three fixture journeys (a city walk overlapping a second walk, a 1,000 km drive, an antimeridian flight) as raw observations + segments, generated from `tool/journeygen` with a fixed seed. Read-only.
   - Any W4 dependency added up front, as in W3.
3. Dispatch.

## File ownership

| Path | Owner |
| --- | --- |
| `CoverageReader.ets`, the `DerivedStateDigest` interface, `dev/fixtures/**`, package and lock files | **kickoff (frozen)** |
| `app/store/src/main/ets/reveal/**`, `app/store/src/ohosTest/**/reveal/**` | S01-40 |
| `app/map/src/main/ets/fog/**`, its tests | S01-41 |
| `tool/guards/**`, `tool/test/**`, CI configuration (if any), `code-linter.json5` | S01-42 |

**Merge order:** S01-40 → S01-41 → S01-42.

---

### S01-40 — Reveal pipeline, provenance, rebuild

**Branch:** `s01/40-reveal` · **Size:** L

**Goal.** `RevealService` turns raw observations and edits into geometry, coverage with per-segment provenance, aggregation and statistics; adding, editing and deleting segments keeps everything exact; a full rebuild reproduces the same digest.

**Deliverables.**
1. `RevealService.revealSegment / unrevealSegment / rebuildSegment / rebuildAll`, on TaskPool, writing per F01.2's layout and maintaining F01.3 by rebuild; transit legs reveal endpoints only (F01.6).
2. `CoverageReader` per F01.4 and the read rule.
3. `DerivedStateDigest` per F01.7.
4. Mode switch re-derives geometry and coverage and keeps vias and corrections; `none` reveals points only.
5. Change notifications fire after each committed change so the fog can invalidate.

**Acceptance (instrument tests on the emulator; command + counts).**
- [ ] Add → delete returns the digest to the empty baseline exactly.
- [ ] Two overlapping journeys: delete one, the other's coverage is unchanged (digest of the survivor alone equals the digest after delete).
- [ ] Drop all derived data → `rebuildAll` → identical digest.
- [ ] Mode switch preserves vias and corrections; a correction leaves the original observation byte-identical.
- [ ] The 1,000 km leg meets its reveal budget on the phone, or a backlog row for S01-51.

**Owner steps:** none.

---

### S01-41 — Fog renderer

**Branch:** `s01/41-fog` · **Size:** L

**Goal.** A `FogController` that draws fog per G1(e) for the visible tiles at every zoom, reading only through `CoverageReader`, off the UI thread, with caching and invalidation.

**Deliverables.**
1. Tile/level selection per the ruled zoom → level mapping, with padding and a per-tile cache.
2. Fog geometry from `app/geo` helpers, delivered into the map's fog slot through `FootMapController`.
3. Invalidation on `CoverageReader` change notifications and on camera idle; no per-frame recomputation.
4. Correct fog across the antimeridian and at the world view.

**Acceptance.**
- [ ] Unit tests against a fake `CoverageReader`: the fog for probe tiles at several levels equals the expected geometry; an empty store gives full fog.
- [ ] After S01-40 merges (or on its branch at close): fog probe tiles for the fixtures have holes along each fixture and none elsewhere.
- [ ] Pan metrics on the phone against the budget (labelled), or a backlog row for S01-51.

**Owner steps:** none.

---

### S01-42 — Guards + CI

**Branch:** `s01/42-guards` · **Size:** M

**Goal.** Make the invariants fail loudly: dependency licences, permissions, runtime hosts, and private data in logs; settle where these checks run.

**Deliverables.**
1. `tool/guards/licences.mjs`: reads the ohpm lock file(s) and each package's declared licence; fails on anything not on an allowlist of open licences or on a denylist (proprietary map/analytics/ads SDKs, Huawei Map Kit and Analytics/Ads kits used by code).
2. `tool/guards/permissions.mjs`: parses every `module.json5`; fails on `ohos.permission.LOCATION_IN_BACKGROUND` or any permission not in an allowlist that grows only by stage brief; checks each `user_grant` has `reason` and `usedScene`.
3. `tool/guards/hosts.mjs`: collects URLs from styles, resources and code and fails on a host not in `docs/network-hosts.md`.
4. A log guard: a `codelinter` rule or a scripted check that fails on `hilog` calls formatting coordinate, note, media-metadata or key variables; and a runtime check in the instrument tests that grep the test run's `hilog` output for fixture coordinates.
5. Tests for each guard in `tool/test/` (`node --test`).
6. CI shape as an H-row proposal: what can run where (host tests are documented as unsupported on Linux; Command Line Tools access), and either a working pipeline or a recorded reason for local-only gating.

**Acceptance.**
- [ ] Each guard fails on a planted violation and passes on the tree (commands + outputs).
- [ ] The guard set runs in one command from the repo root.

**Owner steps:** approve the CI H-row.
