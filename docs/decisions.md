# HarmonyOS decision log

Append-only. Product changes require explicit owner decisions. Source D-### rows are historical; H-### rows govern this repo. Nothing here changes Android/iOS.

| ID | Decision | Authority |
| --- | --- | --- |
| H-001 | Independent native HarmonyOS NEXT app in a separate local repository. Flutter is reference material only. | Owner, 2026-10-13 |
| H-002 | Transfer the complete product goal, provisions, and story. Rediscover technical design, evaluations, and implementation. No automatic adoption of z20, spans, rollups, MapLibre, SQLCipher, Drift, benchmarks, or milestone sequencing. | Owner, 2026-10-13 |
| H-003 | Preserve all ten invariants, full feature scope, privacy/compliance, free launch, and no gating of history. PRODUCT.md is the active charter. | Handoff preserved under H-002 |
| H-004 | Retain MPL-2.0, English-only launch with i18n readiness, 100 m local reveal adjustable from 25 m to 2 km, and endpoints-only transit pending later transit design. | Source D-001, D-005, D-018 preserved under H-002 |
| H-005 | Organise work as stage → wave → session, with parallel sessions in disjoint file ownership, per `stages/README.md` and the skills in `.claude/skills/`. Only the process shape is taken from the reference project; none of its stage content, numbering, status, ownership, device passes or technical decisions. Stages follow the charter's horizon outcomes. Stage 00 closes as the transfer; native discovery and its proof flow become Stage 01 — Foundation, planned in full; Stages 02–06 hold threads only and are planned in detail after the previous stage exits. | Owner, 2026-10-15 |

## Proposed, not decided

ArkTS/ArkUI is the intended native direction. Choose the toolchain/language generation, UI state model, modules, dependencies, and rendering architecture during discovery. A Web-based map, if proposed, is an explicit architecture choice rather than an assumed meaning of native.

## Open technical choices

Minimum/target/compile SDK, phone matrix, tooling and CI, bundle identifier, maps and sources, geometry and precision, persistence and key protection, media access, recording lifecycle, performance budgets, test methodology, distribution, and optional services.

Use new evidence while preserving product outcomes. No shared runtime, binary schema, matching cell IDs, or identical implementation with Flutter is required. Reopen product commitments only through explicit owner decisions.
