---
name: run-session
description: Execute one FootNoted HarmonyOS session brief (S<NN>-<wn>) as a session agent inside its own worktree — stay inside owned files, meet the acceptance list with evidence, write the session note. Use when dispatched as a session agent, or when the user says "run session S01-xx" / "do S01-xx".
---

# Run a session

You are a **session agent**. You own exactly the files under **Owns** in your brief, and nothing else.

## 1. Orient (in order; stop when you have enough)

1. `AGENTS.md`: rules, repo map, the verified command set.
2. Your brief in `stages/NN-slug/W<n>-*.md`. Read the whole wave file once: sibling briefs tell you what *not* to build.
3. The wave kickoff note in `stages/NN-slug/notes/`: baseline, seams, ownership changes, device lease, owner rulings.
4. The stage README: the exit criteria you feed and the F-contract.
5. The `docs/PRODUCT.md` sections your brief names.
6. The harmonyos-app-lifecycle skill's references for your task (ArkTS rules, state/lifecycle, build, testing), and the exact Huawei API pages for anything you call.

## 2. Set up

- Create branch `s<NN>/<wn>-<slug>` at the commit you were given. Run `ohpm install` in the project(s) you touch; use `--no-daemon` for every hvigorw call.
- Run the baseline for your module(s) and note the counts.

## 3. Work

- **Stay inside "Owns".** A file you don't own, a seam change, an undeclared dependency or permission, or a schema migration not in your brief: **stop that line of work** and record it under *Out-of-contract findings*. Do not work around it.
- **Read before you use an API.** Exact `@kit.*` import, signature, error codes, permissions, and the `since` API level vs the project's `compatibleSdkVersion`. Don't translate Android, Web or TypeScript examples by resemblance. Licences of anything you add must be open (invariant 1).
- **ArkTS and UI.** Self-review against the arkts-no-* rules before compiling. Follow the project's state-management generation. Keep blocking I/O and heavy compute off the UI thread (TaskPool). Never rely on `onDestroy`/`onBackground` to save data.
- **Devices.** Use only the target your lease names (`hdc list targets`; `-t <key>` when several are attached). Never uninstall or clear another session's bundle. Label every number emulator or phone, with OS/API and build mode. Owner-only steps (Huawei ID, signing, physical handling) are recorded as owner-observed only when the owner reports them.
- **Privacy.** No coordinates, notes, media metadata, keys or credentials in `hilog`, results or commits. Signing material never enters the repo.
- Tests first where practical. Commit in small steps: `S<NN>-<wn>: <what>`.

## 4. Finish

1. Your module checks green with the verified commands (build, local tests, instrument tests where the brief requires, codelinter, guards once they exist). Report the highest verification rung reached.
2. Run the `invariant-review` skill on your diff.
3. Write `stages/NN-slug/notes/<YYYY-MM-DD>-S<NN>-<wn>-<slug>.md` from template B, including the acceptance list with ✅/❌ and evidence, docs consulted, measurements with their labels, backlog rows (`S<NN>-V<wn>a`, `…b`; the orchestrator renumbers) and a handover. Lead the summary with anything that did **not** land.
4. Commit the note. Final message to the orchestrator: branch, head SHA, acceptance items not met, out-of-contract findings, owner steps pending.

Do **not** edit `VERIFICATION_BACKLOG.md`, the stage README status, `stages/README.md` or `docs/decisions.md`. Those belong to the orchestrator.
