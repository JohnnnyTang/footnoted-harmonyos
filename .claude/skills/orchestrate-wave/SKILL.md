---
name: orchestrate-wave
description: Open, dispatch and close a FootNoted HarmonyOS wave (stage → wave → session). Use when the user says "start/kick off/open W<n>", "dispatch the wave", "merge/close W<n>", "rule G<n>", or "run the next wave" for the ACTIVE stage under stages/. Runs in the foreground as the orchestrator; sessions run as parallel background agents.
---

# Orchestrate a wave

You are the **orchestrator**. You own the stage branch, the seams, the merges, the gates and the stage status. Session agents own only the files their brief gives them.

Read first: `AGENTS.md`, `stages/README.md` (conventions), the active stage `README.md` (exit criteria, F-contract, current status), and the wave file `stages/NN-slug/W<n>-*.md`.

## A. Kickoff (foreground; nothing dispatches until every step is done)

1. **Branch.** In a stage's first wave, cut `stage/NN-slug` from an up-to-date `main`. Otherwise rebase the stage branch if `main` moved. Push only if the owner asked.
2. **Baseline, with commands recorded.** Run the verified command set from `AGENTS.md` (ohpm install, build, local tests, codelinter, guards) and keep counts, not adjectives. Note the toolchain versions; they must match the pinned H-row. Red baseline → fix or record it with a reason before dispatching.
3. **Owner rulings.** Resolve every owner ruling the wave file lists (AskUserQuestion) or record it as a blocker. Log decisions in `docs/decisions.md` as H-rows (append-only).
4. **Seams.** Implement every kickoff seam yourself: module registration, dependencies + lock file, frozen types, resource files, permissions, fixtures, skeletons. Commit them on the stage branch **before** dispatch. Seams are read-only to sessions.
5. **Ownership, order, device lease.** Re-check the ownership table against the actual tree. Confirm the merge order. Decide which sessions may use the emulator and the phone, and when; owner-only steps get a slot. A session needing a sibling's unmerged code goes in a `W<n>b` sub-wave.
6. **Kickoff note.** Write `stages/NN-slug/notes/<YYYY-MM-DD>-W<n>-kickoff.md` (template C). Commit it.

## B. Dispatch

Start **one background agent per session**, all in a single message, each with worktree isolation. Prompt shape:

```
You are session agent S<NN>-<wn> for FootNoted (HarmonyOS NEXT). Follow the `run-session` skill exactly.
Branch: s<NN>/<wn>-<slug>, created from stage/NN-slug @ <sha> (your worktree is on that commit; create the branch).
Your brief: stages/NN-slug/W<n>-<file>.md → section "S<NN>-<wn>".
Wave kickoff note: stages/NN-slug/notes/<date>-W<n>-kickoff.md.
Device lease: <which target, when, connect key>. Owner context: <rulings, phone details>.
Do not edit files outside your "Owns" list. Out of contract → stop and record it in your session note.
Finish with: your checks green, your session note committed on your branch, and a final message listing the branch, head SHA, acceptance items NOT met, and owner steps still pending.
```

Never fabricate or predict a session's result. Wait for each completion notification.

## C. Close (after every session reports)

1. **Review each branch** before merging: the diff stays inside its owned files; each acceptance item is evidenced by a command or scenario with where it ran; the invariant check is present; no owner-only result is claimed without the owner. Bounce gaps back to the agent (SendMessage) rather than fixing them silently.
2. **Merge in the declared order.** Rebase each branch onto the stage branch, `git merge --no-ff`, re-run that module's checks.
3. **Whole check set on the merged tree** (A.2). The sum of new tests per session should match the observed count delta.
4. **Backlog.** Append every `## Backlog rows` entry from the notes to `VERIFICATION_BACKLOG.md`, renumbered `S<NN>-V###`.
5. **Status.** Update the stage README's **Current status** (a line per session plus a wave line) and **Blockers that still bind**, and the stage row in `stages/README.md`. Status lives only there.
6. **Close note** `notes/<date>-W<n>-close.md`. Remove merged worktrees and session branches.
7. **Gates.** If the wave ends at a gate, write the decision note with options, evidence, recommendation and invariant check per question, ask the owner, log H-rows, freeze the F-clauses. Nothing in the next wave dispatches before the rulings are logged.

## Rules

- Do not do a session's work in the foreground "to save time". A missing seam is added as a seam (commit + note).
- Never claim a phone, owner or signing result nobody observed. Missing evidence → a backlog row.
- Records are append-only; correct with `**Correction of record (date):**`.
