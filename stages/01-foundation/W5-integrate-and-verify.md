# Stage 01 · W5 — Integrate and verify (the Stage 01 exit)

**Opens after:** W4 merged. **Sessions:** S01-50, then S01-51 (serial: S01-51 verifies what S01-50 wires). **Closes into:** stage exit.
**Purpose:** wire the real caller path end to end, add the located-photo proof flow, then verify every exit criterion on the phone and leave the docs true. Verification is a wave of its own.

## Kickoff obligations

1. Rebase; baseline including the guard set; record it.
2. Owner prerequisites: the phone attached (S01-E3) and a slot for the physical steps (restart, Recents, screenshots).
3. **Permission seam:** add `ohos.permission.MEDIA_LOCATION` (user grant, with `reason` and `usedScene` naming the feature) to `module.json5` and the permissions allowlist, with the rationale strings in resources. Record it in the kickoff note.

## File ownership

| Path | Owner |
| --- | --- |
| `app/entry/src/main/ets/app/Bootstrap.ets`, `app/entry/src/main/ets/dev/**` (except fixtures), `app/entry/src/main/ets/features/memory/**`, `app/entry/src/ohosTest/**/e2e/**` | S01-50 |
| `README.md`, `AGENTS.md`, `docs/**` except `PRODUCT.md`, this stage's README status and exit table, `evidence/S01-51/**` | S01-51 |

`docs/PRODUCT.md` changes only by explicit owner decision.

---

### S01-50 — End-to-end wiring, dev harness, located-photo path

**Branch:** `s01/50-e2e` · **Size:** M

**Goal.** The app boots into the real stack — protected store → `AppMetaStore` (first use recorded) → `RevealService` → `CoverageReader` → `FogController` on the home map — and the proof flow runs through the same path a user's would.

**Deliverables.**
1. `Bootstrap`: open the store, run migrations, build services, record first use; a visible error state on failure, never a silent empty map (invariant 9).
2. Dev page (debug only): insert fixtures, reveal, delete one or all, rebuild, show counts and the derived-state digest, jump the camera to world/regional/city/street presets over each fixture.
3. **Located-photo path** (dev-harness entry, not the Stage 02 import UI): rationale → `MEDIA_LOCATION` → `PhotoViewPicker` (one item) → read location and capture time → insert media + observation + a one-point segment → reveal. A photo without readable location is reported, offered nothing else, and not placed.
4. An instrument test of the exit scenario through the real providers: baseline digest and probe fog → insert fixtures + reveal → holes along each fixture, none across the antimeridian's far side → delete the overlapping walk → the other walk's coverage intact → delete all → digest and probe fog equal baseline.

**Acceptance.**
- [ ] The e2e instrument test passes on the emulator (command + output).
- [ ] Cold start twice on the emulator: first use written once; the store reopens with its key.
- [ ] The guard set is green.

**Owner steps:** none (the phone run is S01-51's).

---

### S01-51 — Exit verification + docs

**Branch:** `s01/51-exit` · **Size:** M

**Goal.** Walk every exit criterion (1–13) with evidence on the phone, and leave the docs true. Unmet criteria are recorded as MISSED with backlog rows, never softened.

**Deliverables.**
1. On the phone: the e2e instrument test; the proof flow by hand with a real located photo (owner picks it); restart and Recents removal with history retained; screenshot pairs (before reveal / revealed / after delete) at three zooms; pan metrics on the production fog path with the full dataset against the G1 budget.
2. An exit table in this stage's README: each criterion → MET / MET IN PART / MISSED, with evidence links.
3. Docs: `README.md` (what exists, how to build and test), `AGENTS.md` repo map and verify commands, `docs/decisions.md` complete, `docs/network-hosts.md` true; close or carry every `S01-V` row.
4. `invariant-review` on `main...stage/01-foundation`, recorded in the note.
5. A draft of the stage PR description (`stage/01-foundation → main`).

**Acceptance.**
- [ ] Every criterion has a verdict and evidence; every non-MET criterion has a backlog row.
- [ ] No owner-only result is claimed unless the owner reported it.

**Owner steps:** keep the phone attached; pick the real photo; perform the restart and Recents steps if they cannot be scripted.

---

## Stage exit (orchestrator)

1. The owner reviews the exit table.
2. With the owner's go-ahead, open the PR `stage/01-foundation → main` and merge after review. Nothing is pushed or published without that instruction.
3. The stage row becomes `EXITED` or `EXITED WITH CAVEATS`.
4. **Plan Stage 02 in detail** from [`../02-import/README.md`](../02-import/README.md) with the `plan-stage` skill.
