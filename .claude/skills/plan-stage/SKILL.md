---
name: plan-stage
description: Turn a FootNoted HarmonyOS KICKOFF stage (stages/NN-slug/README.md, threads only) into a detailed plan with waves, balanced parallel sessions, briefs, file ownership, seams and merge order, in the same shape as stages/01-foundation. Use when the previous stage has exited and the user says "plan stage N" or "plan the next stage".
---

# Plan a stage

Input: `stages/NN-slug/README.md` in state `KICKOFF`. Output: the same directory in the shape of `stages/01-foundation/` (copy its structure, not its content):

- `README.md`: header (state, branch, horizon outcome, backlog and wave links), **Objective**, numbered **Exit criteria** tagged `[host] [emu] [phone] [owner] [doc]`, the **F<NN> contract** (cross-module seams), **Waves at a glance**, **Session index** (ID, title, size, wave, depends on), **Risks**, **Current status**, **Blockers that still bind**.
- `W<n>-<slug>.md` per wave: kickoff obligations (baseline, owner rulings, **seams implemented once**), a **file ownership table**, the **merge order**, the device lease, then one brief per session in template A (`stages/TEMPLATE.md`).
- `VERIFICATION_BACKLOG.md`: headers only, plus `S<NN>-E#` rows for owner and environment needs.

## Procedure

1. **Re-ground.** Read `AGENTS.md`, `docs/PRODUCT.md` (the horizon outcome, invariants, compliance, acceptance principles), `docs/decisions.md`, the previous stage's exit table and **open backlog rows** (inherited risks), and the actual code tree. The kickoff threads were written before the previous stage ran: verify each against the code and the H-rows.
2. **Ground platform facts.** Use the harmonyos-app-lifecycle skill and the Huawei docs for every API, permission, minimum API level and tool behaviour a brief depends on. Cite the page. Never write a command or config field from memory.
3. **Settle open questions.** List every Open item and thread question this stage needs. Product questions go to the owner (AskUserQuestion, at most four per call, recommended option first) before writing briefs that depend on them. Log answers as H-rows. Engineering choices may be proposed in the plan and ruled at a kickoff or gate.
4. **Exit criteria first.** Derive them from the horizon outcome, the charter's acceptance principles and the compliance items. Each is checkable with a named method and an honest tag; phone and owner criteria are never disguised as host checks.
5. **Find the seams.** Shared files in a HarmonyOS project: `build-profile.json5` (modules), `oh-package.json5` + the lock file, `module.json5` (abilities, permissions), `resources/**/element/string.json`, the Navigation route map, store migrations, fixtures, frozen types. Each becomes a frozen contract clause or a kickoff seam, so parallel sessions never edit the same file.
6. **Cut sessions.** Each session:
   - owns a disjoint file set;
   - is S/M/L, where L ≈ one full agent session (roughly 1–4 h, ~1–2 k changed lines including tests); split anything bigger;
   - has 3–6 deliverables and an acceptance list where every item names a command or scenario and where it runs;
   - names its owner steps (Huawei ID, signing, physical phone handling).

   Aim for **3–4 parallel sessions per implementation wave** with balanced sizes, and put verification in its own final wave. Account for the shared emulator and phone in the device lease.
7. **Order waves** by dependency: a session needing a sibling's unmerged code goes in a later wave or a `b` sub-wave. Declare each wave's merge order.
8. **Write it**, set the stage row in `stages/README.md` to `ACTIVE`, and present the owner a summary: the waves table, owner involvement, and decisions still needed.
