# Stage 01 · W0 — Planning and bootstrap

**Sessions:** S01-00 (planning, done 2026-10-15) and S01-01 (bootstrap, foreground orchestrator + owner). **Branch:** `main` (the W1 kickoff cuts the stage branch).

W0 is recorded as the stage's first wave so the record stays complete. Both sessions run in the foreground; nothing is dispatched in parallel.

## S01-00 — Planning (done)

**Date:** 2026-10-15 · **Author:** orchestrator.

**Read:** `docs/PRODUCT.md`, `docs/decisions.md`, `docs/TRANSFER.md`, Stage 00, the archived handoff's milestone list, and the reference project's process (stage → wave → session conventions, templates, skills). Its content, status, numbering, device passes and technical decisions were not imported (H-002, `AGENTS.md`). Grounded HarmonyOS facts in the harmonyos-app-lifecycle skill and the local Huawei docs archive (pages cited where used).

**Produced:**
- `stages/README.md` (conventions, stage index, threads not yet opened) and `stages/TEMPLATE.md`.
- This stage: `README.md`, `DISCOVERY.md`, `W0`–`W5` wave files with 15 briefs (+1 conditional), `VERIFICATION_BACKLOG.md`.
- Kickoff threads for Stages 02–06.
- `.claude/skills/`: `plan-stage`, `orchestrate-wave`, `run-session`, `invariant-review`, adapted to this repo.
- `AGENTS.md` / `CLAUDE.md` pointers to the conventions and skills; H-005 in `docs/decisions.md`; a dated status line in Stage 00.

**Observed environment (2026-10-15, Windows dev machine):** `hvigorw`, `ohpm` and `hdc` are not on PATH and no DevEco installation was found; Node is available. No build, test, device or measurement claim is made.

## S01-01 — Toolchain, targets, DevEco scaffold, agent config

**Wave:** W0 · **Branch:** `main` (small, reviewable commits) · **Size:** M · **Depends on:** S01-E1, S01-E2 (S01-E3 optional)

**Goal.** A verified native toolchain and a DevEco-generated app that builds, tests and launches from recorded commands, with the version facts recorded separately and nothing invented.

**Owns:** the generated project directory (proposed `app/`, ruled in this session), `.gitignore`, `.gitattributes`, `.claude/settings.json`, `tool/hooks/**`, `README.md` (Development section), `AGENTS.md` (verify section only), `docs/decisions.md` (new H-rows), this stage's `notes/*-S01-01-*.md`, `evidence/S01-01/**`.

**Deliverables.**
1. **Inventory with commands.** After the owner installs DevEco Studio: record the DevEco version, hvigor, ohpm, hdc, JDK and bundled Node versions using their own version commands, and the installed SDK/API levels. Record whether Command Line Tools are installed separately.
2. **Owner rulings → H-rows.** Proposed: H-006 toolchain pin (no upgrades mid-wave); H-007 `compatibleSdkVersion` (provisional, development only; the launch minimum stays an open product question), `targetSdkVersion` and the compile SDK, each separately; H-008 a temporary development bundle name, explicitly not a distribution identity; H-009 device types (phone only for Stage 01 unless the owner says otherwise); H-010 project location (`app/`) and the verification gate (host tests on the Windows dev machine until S01-42 rules a CI shape).
3. **Scaffold from the installed template.** Generate an ArkTS, Stage-model "Empty Ability" (or the current equivalent) project with DevEco itself. Record the template name and version. Do not hand-write hvigor, model or SDK versions.
4. **Baseline, with commands and outputs:** `ohpm install`; a debug `assembleHap` build with `--no-daemon`; the template's local test with `hvigorw test` (or the documented equivalent if the template lacks one); `codelinter`; install and launch on the emulator through `hdc` (the owner starts the emulator if it requires login). Write the verified set into `AGENTS.md` ("Verify before claiming") and `README.md`.
5. **Hygiene.** Confirm `.gitignore` covers `oh_modules/`, `.hvigor/`, `build/`, `local.properties` and all signing material; `git status --ignored` shows no signing files tracked. Auto-signing material stays in the owner's profile (`~/.ohos/config`), never in the repo.
6. **Agent configuration.** `.claude/settings.json`: an allowlist for read-only and test commands, force-push denied, and a SessionStart hook (`tool/hooks/session_start.mjs`, Node, no dependencies) that prints the active stage row and its Current status "Next" line.

**Acceptance.**
- [ ] Every version and SDK value comes from a command output or DevEco's About screen, quoted in the note.
- [ ] The debug HAP builds from the command line and launches on the emulator (command + output).
- [ ] The template's local test runs from the command line on the dev machine (command + result count), or the note records exactly why it cannot.
- [ ] No signing material, password or profile is tracked (command + output).
- [ ] The SessionStart hook prints the Stage 01 row (command + output).

**Owner steps:** install DevEco Studio (S01-E1); sign in with the Huawei ID and enable debug auto-signing (S01-E2); start the emulator if it needs a logged-in account; rule H-007's provisional minimum API (S01-E4). Optionally plug in the phone (S01-E3) and confirm `hdc list targets` sees it.

**Out of scope.** Any product code, dependency, permission or map work.

**Traps.** Run hvigorw from the project root only. Never place the project under a dotted path. Keep three SDK versions distinct. Do not "fix" a template build by changing SDK or tool versions; record the error and ask. Don't trust remembered commands: use the skill's cheat sheet and the installed tools' `--help`.

## Handover to the W1 kickoff

- The verified command set is in `AGENTS.md`; the W1 kickoff uses it for its baseline.
- The spike skeletons copy the generated template so their versions match the pinned toolchain.
