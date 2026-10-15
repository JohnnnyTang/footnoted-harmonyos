# HarmonyOS work

Independent milestones are planned from this client's charter and evidence. Android/iOS stage numbers, status, device gates, and schedules are not imported.

Work is organised in three layers (H-005):

- **Stage.** One outcome from the [full product horizon](../docs/PRODUCT.md#full-product-horizon), with numbered exit criteria. Lives in `stages/NN-slug/`.
- **Wave.** Sessions that can run **in parallel** because their file ownership does not overlap. Waves run strictly in order; a wave opens only after the previous one has merged and its checks are green.
- **Session.** One agent's balanced unit of work: one brief, one worktree, one branch, one session note. Sized for a single agent session (roughly 1–4 hours of agent work), touching one module or feature slice.

## Stage index

Status lives in exactly two places: this table and the stage README's **Current status**.

| # | Stage | Horizon outcome | State | What still binds |
| --- | --- | --- | --- | --- |
| 00 | [Product transfer and discovery](00-discovery/README.md) | — | `EXITED WITH CAVEATS` (2026-10-15): transfer accepted; native discovery and its proof flow moved to Stage 01 | — |
| 01 | [Foundation](01-foundation/README.md) | 1 | `ACTIVE`: W0 planning done (S01-00, 2026-10-15); **S01-01 bootstrap next** | DevEco toolchain (S01-E1), Huawei ID for signing/emulator (S01-E2), target phone (S01-E3), provisional minimum API (S01-E4) |
| 02 | [Import](02-import/README.md) | 2 | `KICKOFF`: threads only | Stage 01 exit |
| 03 | [Live recording](03-live-recording/README.md) | 3 | `KICKOFF`: threads only | Stage 02 exit; physical phones |
| 04 | [Trips and places](04-trips-places/README.md) | 4 | `KICKOFF`: threads only | Stage 02 exit (Stage 03 not required) |
| 05 | [Statistics](05-statistics/README.md) | 5 | `KICKOFF`: threads only | Stage 04 exit |
| 06 | [Route editing](06-route-editing/README.md) | 6 | `KICKOFF`: threads only | Stage 02 exit |
| — | Passive mode | 7 | not opened | separate consent; HarmonyOS background-location rules verified |
| — | Road routing (opt-in) | 8 | not opened | server/provider choice; ODbL boundaries |
| — | Later | 9 | not opened | soft fog, offline packs, E2E backup/sync, paid additions, year-in-review, heatmaps, export, posters |
| — | Release readiness | (cross-cutting) | not opened | launch market, permanent bundle name, AGC identity, store disclosures |

**States:** `KICKOFF` (threads only, not yet planned into waves) · `ACTIVE` (planned and in progress) · `MERGED, NOT EXITED` (merged to `main`, some exit criteria open) · `EXITED WITH CAVEATS` (every criterion met or carried as a named backlog row) · `EXITED`.

A `KICKOFF` stage is planned into waves and sessions **only after the previous stage exits**, with the `plan-stage` skill. Plans written earlier go stale; the threads in each kickoff file are starting points, not commitments.

## Threads not yet opened

- **Passive mode (horizon 7).** Optional, off by default, coarse and low power. `ohos.permission.LOCATION_IN_BACKGROUND` is a separate user grant; consent, distribution rules and the actual HarmonyOS background behaviour must be verified before any build declares it. Never declared by Stages 01–06.
- **Road routing (horizon 8).** Opt-in server routing that honours vias; per-user, per-segment geometry only (ODbL); a new host in `docs/network-hosts.md`; entitlement-gated as a hosted service. Provider open.
- **Later (horizon 9).** Soft-edged fog, offline map packs, opt-in end-to-end encrypted backup/sync, paid additions through `Entitlements`, year-in-review, heatmaps, advanced export with location stripped by default, printed posters.
- **Release readiness (cross-cutting).** Launch market and minimum API (open product questions), permanent bundle name, AppGallery Connect app identity, release signing kept out of Git, privacy policy and store disclosures that match actual data flows. Distribution actions need explicit owner authorization.

## Conventions

### IDs

- **Session `S<stage>-<wave><n>`**: `S01-10` is Stage 01, Wave 1, session 0. Wave 0 (planning/bootstrap) is `S01-0x`. A suffix letter (`S01-31b`) marks a split decided at kickoff.
- **Backlog rows `S<stage>-V###`** live in the stage's `VERIFICATION_BACKLOG.md`. **Environment/owner-action rows `S<stage>-E#`.**
- **Gates `G<n>`**: owner rulings inside a stage before work continues (Stage 01 G1: engine, representation, storage and fog decisions from the spikes).
- **Decisions `H-###`** in [`docs/decisions.md`](../docs/decisions.md). Source `D-###` rows are history only.

### Branches and worktrees

- `main` stays reviewable. One stage branch, `stage/NN-slug`, is cut from `main` by the stage's first wave kickoff.
- Each session works in its own worktree on branch `sNN/<wave><n>-<slug>` (for example `s01/10-journeys-grid`), cut from the stage branch at wave open.
- The orchestrator merges sessions into the stage branch **in the declared merge order**, rebasing each first and merging with `--no-ff`. A stage reaches `main` through one PR at exit, or at an agreed checkpoint.
- Nothing is pushed or published unless the owner asks.

### How a wave runs

1. **Kickoff (foreground orchestrator, `orchestrate-wave` skill).** Rebase the stage branch. Re-run the verified checks and record the baseline **with the commands**. Implement every cross-session **seam** once (shared types, resource files, module registration, dependency and lock files, fixtures), read-only to sessions. Confirm ownership, the merge order and the device lease. Resolve or escalate owner questions. Write `notes/<date>-W<n>-kickoff.md`.
2. **Dispatch.** One background agent per session, each following the `run-session` skill against its brief. A session that needs a sibling's unmerged code goes in a later sub-wave (`W3b`).
3. **Session.** The agent stays inside its owned files, writes tests, runs its module checks, and writes a session note (`stages/NN-slug/notes/<date>-S<id>-<slug>.md`, skeleton in [`TEMPLATE.md`](TEMPLATE.md)). Backlog rows go in the note, never straight into `VERIFICATION_BACKLOG.md`.
4. **Close (orchestrator).** Merge in order. Re-run the **whole** check set on the merged tree. Append backlog rows. Update the stage README **Current status** and the row above. Write `notes/<date>-W<n>-close.md`.

### Shared device and toolchain rules

- **One build tree per worktree.** Each worktree runs its own `ohpm install`; builds and tests use `--no-daemon` so parallel sessions do not share an hvigor daemon.
- **Device lease.** The emulator and the owner's phone are shared. Address a target explicitly (`hdc -t <connect-key>` when more than one is attached), never uninstall or clear another session's bundle, and record the device, OS/API and build mode with every result. Spike apps carry distinct bundle names so parallel installs cannot collide.
- **Owner-only steps.** Huawei ID login, signing material, AppGallery Connect, and physical handling (walking, locking, rebooting, removing from Recents) are owner steps named in the brief. An agent never reports such a result unless the owner supplied it.

### Rules every session follows

- **Out of contract → stop and raise.** A session that needs a file it does not own, a frozen seam change, an undeclared dependency, permission or schema migration stops that line of work and records the need in its note.
- **Never claim what you did not observe.** Static review, compile, host tests, emulator tests, phone tests, signing and release are separate rungs; report the highest one reached. An unrun check is a backlog row, not a pass.
- **Records are append-only.** Correct the past with a dated `**Correction of record (YYYY-MM-DD):**` line.
- **Invariants first.** Every session checks its diff against the ten invariants in [`docs/PRODUCT.md`](../docs/PRODUCT.md#non-negotiable-invariants) (`invariant-review` skill) before it finishes.
