# Templates

Copy the skeleton you need. Keep the headings so notes stay greppable.

---

## A. Session brief (written at planning time, lives in the wave file)

```markdown
### S<NN>-<wn> — <title>

**Wave:** W<n> · **Branch:** `s<NN>/<wn>-<slug>` · **Size:** S | M | L · **Depends on:** <seams / merged sessions / G-rulings>

**Goal.** One paragraph: the outcome, not the steps.

**Owns (may create/edit):**
- `path/…`

**Reads (must not edit):**
- `path/…`

**Deliverables.**
1. …

**Acceptance (all must hold, each with the command or scenario that proves it, and where it ran: host / emulator / phone).**
- [ ] …

**Owner steps** (Huawei ID login, signing, physical handling of the phone, …): … or "none".

**Out of scope.** …

**Notes / traps.** …
```

---

## B. Session note (written by the session agent at the end)

File: `stages/NN-slug/notes/YYYY-MM-DD-S<NN>-<wn>-<slug>.md`

```markdown
# YYYY-MM-DD — S<NN>-<wn> <title>

**Stage:** NN · **Wave/session:** W<n> / S<NN>-<wn> · **Branch:** `…` @ `<sha>` · **Author:** <session agent | orchestrator>

## Summary
What landed, in three to six sentences. Lead with anything that did **not** land and why.

## Changes
| File | What |
| --- | --- |

## Verification
Every claim with the command that proves it, where it ran, and its result. Paste counts, not adjectives.
Highest rung reached: static review | compile | host tests | emulator | phone | signed release.

~~~
<command>
<trimmed output>
~~~

## Docs consulted
Huawei pages actually read (slug or URL), and the API level each fact applies to.

## Invariant check
The ten invariants, one line each: `held` / `n/a` / `RISK → backlog row`. Then permissions, attribution, sharing, monetization.

## Measurements (if any)
Numbers with device model, HarmonyOS version/API, build mode, workload and method.

## Out-of-contract findings
Anything that needed a file, permission, dependency or decision outside the brief. Raised, not worked around.

## Backlog rows
| ID | Row | Owner | Blocks |
| --- | --- | --- | --- |

## Handover
What the next session or the wave close must know.
```

---

## C. Wave kickoff / close note

File: `stages/NN-slug/notes/YYYY-MM-DD-W<n>-kickoff.md` (or `-close.md`)

```markdown
# YYYY-MM-DD — W<n> kickoff|close

## Baseline (commands recorded)
## Seams implemented (kickoff) / merges in order (close)
## Ownership and device lease confirmed / changed
## Owner rulings obtained
## Dispatch list (kickoff) / checks on merged tree (close)
## Backlog rows appended (close)
```
