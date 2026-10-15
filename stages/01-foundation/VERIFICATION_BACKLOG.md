# Stage 01 — Verification backlog

Rows are appended by the orchestrator at merge, from each session note's `## Backlog rows`. Sessions never edit this file. Close a row by striking it through and adding a line of evidence (a date, a command or link, the result).

## Environment / owner-action rows (`S01-E#`)

| ID | Row | Needed by | State |
| --- | --- | --- | --- |
| S01-E1 | **DevEco Studio** installed on the Windows dev machine (and Command Line Tools if wanted for CLI builds). Agents record the versions; they do not install toolchains silently. | S01-01 | open |
| S01-E2 | The owner's **Huawei ID** signed in to DevEco for debug auto-signing and for emulators that require a logged-in account. Signing material stays in the owner's profile, never in Git. | S01-01, every device run | open |
| S01-E3 | The owner's **HarmonyOS phone**: model, HarmonyOS version, API level, RAM; developer mode and USB debugging on; visible in `hdc list targets`. | S01-13 phone steps, S01-21, W5 | open |
| S01-E4 | A **provisional minimum API** for development (`compatibleSdkVersion`). The launch market and final minimum remain open product questions. | S01-01 | open |

## Verification rows (`S01-V###`)

| ID | Row | Raised by | Blocks | State |
| --- | --- | --- | --- | --- |
