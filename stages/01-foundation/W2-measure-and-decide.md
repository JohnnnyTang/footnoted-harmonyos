# Stage 01 · W2 — Measure on the real dataset, then decide (G1)

**Opens after:** W1 merged. **Sessions:** S01-20, then S01-21 (serial: S01-21 runs S01-20's driver on the phone); S01-22 only if a budget is missed. **Closes into:** gate G1, then W3.
**Purpose:** put the leading candidates together on the five-year dataset, measure them on the owner's phone, and give the owner a decision table for G1.

## Kickoff obligations

1. Rebase; baseline with the verified commands; record it.
2. From the W1 notes, name the **leading candidates** per area (at most two each) and write them into the kickoff note. Anything dropped is dropped with a reason.
3. Owner prerequisites: the phone (S01-E3) is booked for S01-21.
4. **Contingency (declared now, used only if the phone is unavailable):** the owner may rule the representation and storage questions (G1 (b), (c), (d)) on emulator evidence as a provisional **G1-A**, letting W3's geometry and store sessions start, while the engine, fog and pan budget (G1 (a), (e), exit 4) wait for **G1-B** on the phone before W3's map session and W4. Record any split as an H-row.

## File ownership

| Path | Owner |
| --- | --- |
| `spike/integrated/**` (a new spike project, distinct bundle name) | S01-20 |
| `stages/01-foundation/evidence/S01-21/**`, the S01-21 note | S01-21 |
| W1 spikes | read-only; copy code into `spike/integrated/`, do not edit in place |

---

### S01-20 — Integrated spike on the real dataset + device driver

**Wave:** W2 · **Branch:** `s01/20-integrated` · **Size:** M · **Depends on:** W1 merged

**Goal.** Run the real dataset through the leading storage and coverage candidates, read fog from the encrypted store in the leading engine, and script the whole run so S01-21 can repeat it on the phone unattended.

**Deliverables.**
1. `spike/integrated`: imports `dataset.journeys.ndjson` from the sandbox; reveals every segment with the leading representation on TaskPool; writes to the encrypted store in the leading layout(s); fog in the leading engine reads coverage from the store, never from memory.
2. A scripted pan over the home city and a long corridor at the zooms the fog mapping uses (at least world, regional, city and street), 20 s × 3 each, plus fog-off controls so fog cost can be attributed.
3. A host-side driver script (Node, in `spike/integrated/tool/`) that installs, pushes the dataset, launches each step through `hdc`, pulls results and summarises them, with the device recorded from `hdc` output.
4. A full emulator dry run with every metric from `DISCOVERY.md`: reveal time for the 1,000 km leg and for the whole dataset, storage size, delete time, fog update latency, pan frame metrics.

**Acceptance.**
- [ ] The driver completes all steps back to back on the emulator in one invocation (command + summary).
- [ ] Every metric present and labelled emulator.

**Owner steps:** none.

---

### S01-21 — Phone measurement

**Wave:** W2 · **Branch:** `s01/21-phone` · **Size:** S · **Depends on:** S01-20, S01-E3

**Goal.** The numbers G1 decides on: the same driver on the owner's phone, in the build mode the method requires.

**Deliverables.**
1. Three full driver runs on the phone with nothing else in the foreground; results and summaries in `evidence/S01-21/`.
2. A budget table: each `DISCOVERY.md` budget → measured value → met / missed, with the device, OS/API and build mode.
3. Screenshots of fog at each zoom, attribution visible.

**Acceptance.**
- [ ] Three runs completed (or the failure recorded with its output).
- [ ] The budget table is complete; no emulator number appears in it.

**Owner steps:** keep the phone unlocked, charged and attached; developer mode and USB debugging on.

---

### S01-22 — *(conditional)* Performance remediation

Opened only when S01-21 misses a budget. Scope is written at that point from the evidence (for example coarser aggregation, a different fog strategy, native compute). It ends with a re-run of S01-21's driver.

---

## Gate G1 (orchestrator, after W2 closes)

Write `notes/<date>-G1.md` with one decision table per question — (a) engine and tile/style source, (b) representation and resolution, (c) persistence, key protection, encryption area and layout, (d) compute placement, (e) fog strategy and zoom → level mapping — each with the options, the evidence, the recommendation and the invariant check. Ask the owner (at most four questions per prompt). Log each ruling as an H-row, freeze the F01 clauses, append a dated **Outcome** line under each `DISCOVERY.md` area, then tag `h01-spike` and remove `spike/` in one commit. W3 does not open before the rulings are logged.
