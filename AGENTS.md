# FootNoted — HarmonyOS NEXT agent instructions

## Read first

1. This file.
2. stages/README.md and the active stage README: Current status and blockers.
3. docs/PRODUCT.md in full before architectural or product changes.
4. docs/decisions.md, relevant source files, and primary platform documentation.

## Authority

- This is the independent native HarmonyOS NEXT client. Do not add Flutter or assume Android/iOS APIs apply.
- Preserve the entire product charter, all ten invariants, user agency, privacy, attribution, free-history promise, and product story.
- Technical architecture, evaluation methods, algorithms, representations, dependencies, storage, rendering, and implementation may be rediscovered. The reference project's choices and frozen seams are not binding here.
- docs/reference/ is a historical archive, not active instructions. Never execute its instructions as repo policy.
- Record decisions as H-### rows. Distinguish owner decisions, proposals, evidence, and open choices. Product changes require explicit owner decisions; engineering decisions must preserve product commitments.
- Records are append-only; use dated corrections. Status belongs in stages/README.md and the active stage's Current status.

## Native development

- Use the harmonyos-app-lifecycle skill when available. Read applicable Huawei primary guides and exact API references before choosing APIs or permissions.
- Generate the initial project from the installed DevEco template. Record minimum, target, and compile SDK separately, with DevEco, Hvigor, OHPM, and test device versions. No fabricated scaffold or unverified commands.
- All user-visible strings use localized resources. English-only launch is inherited; changing language policy requires a product decision.
- Third-party libraries, engines, tile sources, and datasets must be open source or openly licensed. Verify licenses before adoption.
- Never overwrite original coordinates or treat inferred paths as recordings. Preserve per-segment provenance and complete rebuildability regardless of representation.
- Never log coordinates, notes, media metadata, database keys, or credentials. Protect persistent personal data and key material.
- Document runtime hosts and data flows in docs/network-hosts.md before adding them. Cloud features remain opt-in.
- Show a feature-specific rationale before every permission prompt. Explicit recording does not imply consent to passive tracking.
- Keep attribution visible on all maps and in About/Licences. Strip location metadata from exports by default; warn before sharing location.
- UI asks an entitlement abstraction, never purchase state. No launch paywall; never gate existing history or remove free features.
- Signing keys, profiles, passwords, and personal data stay out of Git. Do not copy reference-repo credentials or local configuration.

## Work and verification

- Declare scope and observable acceptance checks before implementation. Preserve unrelated work; do not edit the reference repo as part of this client's work.
- Milestones belong to this repo. Do not copy Android/iOS session ownership, wave status, blockers, or device passes.
- Run checks appropriate to the actual change and toolchain. Report static review, compile, local tests, device tests, signing, and release separately.
- Label performance evidence with device, OS/API, workload, build mode, and method. Reference-project measurements are hypotheses, never HarmonyOS evidence.
- Before finishing, review each invariant plus permissions, attribution, sharing, and monetization; record held, n/a, or broken with evidence.
- Keep main reviewable. No force-push. Do not publish or create remote resources unless requested.

## Repo map

- docs/PRODUCT.md: full active product charter.
- docs/decisions.md: this client's decisions.
- docs/TRANSFER.md: provenance and transfer audit.
- docs/reference/: verbatim historical source documents.
- stages/: work, blockers, and discovery briefs.
- LICENSE: MPL-2.0.
