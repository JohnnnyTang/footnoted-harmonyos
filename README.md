# FootNoted for HarmonyOS NEXT

A personal, private record of a life's journeys on a dark, misted world map. Places you have visited light up as you explore, import memories, and tell your own story.

This is an independent native HarmonyOS NEXT application repository. The Android/iOS Flutter project is a reference, not a codebase or runtime dependency to port. Product goals and promises are preserved in full; technical design and evaluations begin independently.

## Start here

1. [Product charter](docs/PRODUCT.md): vision, ten invariants, features, privacy, monetization, and long-term scope.
2. [Decision log](docs/decisions.md): owner decisions and open choices.
3. [Transfer record](docs/TRANSFER.md): source revision, hashes, and coverage mapping.
4. [Current work](stages/README.md): preparation and next discovery work.
5. [Agent instructions](AGENTS.md): contributor rules.

## Development

No app scaffold or build command exists yet. Select and record the native toolchain, minimum API, and target devices before generating a project from DevEco. Do not invent SDK pins, build files, dependencies, CI passes, or platform test results. ArkTS/ArkUI is the intended native direction; map, storage, geometry, and other technical decisions remain open.

This repository is self-contained for product planning. Archived documents preserve original wording; their relative links may refer to the original repository, as explained in [the archive index](docs/reference/README.md).

## License

The client retains FootNoted's [MPL-2.0 license](LICENSE). Hosted services may be paid later; the app launches free and existing history is never gated.
