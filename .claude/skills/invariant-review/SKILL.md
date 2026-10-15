---
name: invariant-review
description: Check a FootNoted HarmonyOS diff (a session branch, a stage PR, or uncommitted changes) against the ten non-negotiable invariants and the compliance rules in docs/PRODUCT.md. Use before finishing any session, before merging a session branch, before a stage PR, or when the user asks "does this break an invariant?".
---

# Invariant review

Get the diff, for example `git diff <base>...HEAD` (a session: base = the stage branch; a stage PR: base = `main`). Answer **each** row with `held`, `n/a`, or `BROKEN: <file:line, why>`. Never answer "looks fine" in aggregate.

| # | Invariant (docs/PRODUCT.md) | What to look for in the diff |
| --- | --- | --- |
| 1 | Open source only | New entries in any `oh-package.json5`/lock file, bundled JS/CSS, native libraries, styles, fonts, tile sources, datasets: is each openly licensed, with the licence recorded? Any Huawei proprietary kit used for maps, analytics or ads? Guards green once they exist. |
| 2 | Raw points are the only truth | Derived data (geometry, coverage, aggregation, stats, thumbnails) written by a path that cannot rebuild it from observations + edits? |
| 3 | Anchors never overwritten | Any update of coordinates or original metadata in observation/media records? Corrections must be new records. |
| 4 | Connection mode is a view | Inferred geometry stored as if recorded? A mode change must re-derive, keep vias, and be reversible. |
| 5 | Reveal along segments | Reveal computed from points only, without the connecting geometry? |
| 6 | Coverage has provenance | Coverage written without its segment? A bare counter instead of a (unit, segment) record? Delete paths that leave orphaned coverage or erase another journey's? |
| 7 | Local-first, encrypted | A new network call or host missing from `docs/network-hosts.md`? User data in an unencrypted store, plain file, preferences, or the wrong encryption area? Coordinates, notes, media metadata or keys reaching `hilog`, results or crash output? |
| 8 | The user decides | Automatic grouping/segmentation/connection that overwrites `is_user_edited` work, or has no override path? |
| 9 | History never held hostage | An `Entitlements` check or an error path that hides or locks **existing** data, or shows an empty map instead of an error? |
| 10 | No ads/analytics/data sales | Any SDK, kit or endpoint that can see location, media metadata or notes? Crash reporting without coordinate scrubbing? |

Also check, where the diff touches them:

- **Permissions:** every `user_grant` permission in `module.json5` has `reason` and `usedScene` naming the feature; a rationale screen precedes every request; denial and revocation are handled; **no `ohos.permission.LOCATION_IN_BACKGROUND`** outside passive mode; nothing requested for a feature that is not in use.
- **Attribution:** credits visible on every map surface; About/Licences updated for any new data source or package.
- **ODbL:** routed or matched geometry stays per user and per segment; nothing aggregates it into a shared road network.
- **Sharing/export:** location metadata stripped by default (including thumbnails); a warning before sharing anything that reveals location.
- **Monetization hooks:** UI asks `Entitlements`, never purchase state; nothing free is gated.
- **Strings:** user-visible text comes from resources; the name is "FootNoted" (capital N, no asterisk).
- **Secrets:** no signing material, profiles, passwords or tokens in the diff.

Output: the table filled in, then a one-line verdict (`PASS` / `FAIL: n items`). Any `BROKEN` or uncertain item becomes a backlog row in the session note. Never wave it through.
