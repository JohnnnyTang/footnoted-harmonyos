# FootNoted — Product charter for HarmonyOS NEXT

This is the complete active product brief for this independent native client. Preserve the product's goal, provisions, story, and user promises. Technical design, evaluations, implementation, and delivery sequencing may be rediscovered (owner instruction, 2026-10-13).

**Decided** means an inherited product commitment or an explicit owner decision. **Proposed** means a candidate, not an approved commitment. **Open** remains unresolved. A technical alternative must preserve the product commitments; a product change needs an explicit recorded owner decision.

## Product summary

FootNoted is a native HarmonyOS NEXT app that records a person's life footprints on a dark, "misted" world map. Every place they have been lights up, so using it should feel like exploring the world and opening up new sights under their own feet.

**Two ways footprints get in:**

1. **Import mode** — the user picks photos and videos. GPS and timestamps are read from their metadata, the locations are ordered by time and connected into segments, and the area around them is revealed.
2. **Live recording mode** — the app reads phone GPS during a recording session and draws the route as it happens. Photos taken along the way appear as thumbnails at their location. Text notes can be attached to a location on their own or together with photos.

**What users see and do:**

- A dark base map where explored area lights up within a user-set buffer distance of their routes.
- Footprints grouped into **trips** — tight clusters of places joined by one or two long transit legs.
- A choice of how points are connected (straight line, great-circle arc, road-snapped, or not connected), and the ability to hand-edit any connection.
- Statistics on repeat visits and how much of the world they have explored.

**Name:** FootNoted (internal capital N). Do not style it with an asterisk; an unrelated financial publication uses "footnoted\*".

**Positioning:** a personal, private record of a life's journeys — not a fitness tracker and not a precision logger. A few hundred metres between samples is acceptable.

## Non-negotiable invariants

These hold in every milestone. Any design that breaks one is wrong, even if it is faster. All are **\[Decided\]**.

1. **Open source only.** Every library, map engine, tile source and data set must be open source or openly licensed. No proprietary SDKs, no paid map APIs, no paid location plugins.
2. **Raw points are the only source of truth.** GPS fixes and media coordinates are stored as captured. All route geometry, revealed cells and statistics are derived and must be rebuildable from raw data.
3. **Original anchors are never overwritten.** A user correction to a photo's location is a separate row. EXIF values stay intact.
4. **Connection mode is a view, not data.** Never store inferred geometry as if it were recorded. Changing a segment's mode must be instant and reversible.
5. **Reveal along segments, not just at points.** Sparse sampling must still reveal the full corridor between points.
6. **Coverage has provenance.** Every revealed cell records which segment revealed it, so editing or deleting a segment updates the map and stats correctly.
7. **Local-first.** All user data lives on the device, encrypted at rest. Nothing leaves the device unless the user explicitly turns on a feature that needs it.
8. **The user decides.** Automatic grouping, segmentation and connection choices are suggestions with sensible defaults; the user can always override them.
9. **History is never held hostage.** No future paywall, lapse or error may hide or lock a user's existing map, photos or history.
10. **No ads, no analytics SDKs that see location, no data sales.** Ever.

The invariants preserve their original wording above. References to cells, rows, or segments express logical provenance, immutability, and rebuildability; they do not mandate the old physical schema or z20 grid. Rediscovering implementation is not permission to weaken any invariant.

## Experience and visual identity

A person's ordinary commutes, walks, old photographs, holidays, and flights accumulate into one private story of where they have been. Import lets past memories become visible; recording adds journeys as they happen. The map must feel like exploration and discovery, with visited areas lighting up through a dark mist. Keep labels readable and the map visually quiet. Preserve the world-scale view and meaningful transitions between trips, places, routes, and individual memories.

Users choose the revealed width around routes. Preserve the existing product default: 100 m for local segments, adjustable from 25 m to 2 km. Transit currently reveals endpoints only, each with the local default; the later transit setting and final default remain a product question. Source D-018 supersedes D-007. These visible defaults can be revisited by the owner, not silently changed as an implementation shortcut.

## Information the product must retain

- Original GPS fixes, photo/video coordinates and metadata, capture timestamps and available local-time context, accuracy/speed when supplied, source identity, and links to recording sessions or media.
- References to the user's original library assets, capture time, original metadata, and thumbnails. Asset access loss must be handled explicitly; it must not silently erase the user's history.
- Text notes, their time and location, and optional links to one or more media items.
- Recording sessions, user location corrections, ordered via-points, connection modes, per-segment reveal/buffer choices, and whether a segment was user-edited.
- Trips → places → segments, deliberately shallow. A segment may be outside a trip, such as a daily commute. Tags can represent day trips within holidays; no nested sub-trip hierarchy is required.
- Trips can be created empty in advance and segments reassigned between trips. Return legs can be paired with outbound legs.
- Geometry, coverage, and statistics remain derived and rebuildable from original observations plus user edits. Choose physical storage and schema timing independently, but do not lose future editing or provenance capabilities.

## Feature specs

### Import mode

- Multi-select photos and videos from the library; optionally scan a date range.
- Read capture time and coordinates; create one `media` row and one `points` row (source `media`) per item.
- Items without location are listed separately; the user can pin them manually or skip them.
- Re-importing the same asset must not duplicate it (dedupe on platform asset ID).
- After import, run segmentation and trip suggestion, then reveal.

### Live recording mode

- Explicit start / pause / stop of a recording session, with a clear, persistent platform-appropriate indication while recording.
- Target sampling: one fix every few hundred metres or so. **\[Decided\]** Dense tracking is not a goal.
- Photos taken during a session appear as thumbnail markers at their location (in-app camera, or library photos matched by timestamp to the session).
- Text notes can be added at the current location, alone or with photos.

### Passive mode (later)

- Optional always-on background logging, off by default. Coarse and low-power; the HarmonyOS mechanism and permission model must be evaluated independently.

### Connection modes

| Mode | How it draws | Runs | Default when |
| --- | --- | --- | --- |
| Straight line | Polyline through points and vias | On device | Default for everything not detected as a flight |
| Great-circle arc | Arcs between successive points and vias | On device | Implied speed above \~150 km/h (likely a flight) |
| Road-snapped | Road route with vias as constraints | Server, opt-in | Never automatic; user chooses it |
| Not connected | No line; points revealed on their own | — | Gap too large to infer anything |

Mode is chosen **per segment**, not globally. Inferred segments must look different from recorded ones (for example dashed or lower opacity). For sparse photo points, road mode means routing between points, not HMM map matching — matching assumes points seconds apart.

### Segmentation and trips

- **Segment breaks** **\[Proposed\]**: start a new segment when the time gap exceeds a threshold (start at \~6 h) or implied speed is implausible for the previous mode. User can split and merge.
- **Places:** stay-point detection — points within a radius spanning more than a few hours form a place.
- **Trips:** a cluster of places in a time window, joined by transit legs. Usually one outbound leg, sometimes a return leg.
- **Segment kinds:** `local` (inside a place, full buffer) and `transit` (between places).
- **Transit defaults** **\[Proposed\]**: reveal little or nothing along a transit leg (a flight shows nothing below it); endpoints still reveal. User can turn transit reveal on.
- **Return legs:** when a return leg nearly duplicates its outbound leg, link them with `return_of` and draw one line with a two-way marker.
- **Ask, don't assume:** offer a suggestion such as "Looks like a trip to Kyoto, 6–14 March. Group these 47 photos?" Store as `suggested` until confirmed.
- **Map behaviour:** zoom-to-fit a trip; collapse a trip to one pin at low zoom and expand into places as the user zooms in; cluster repeated trips to the same city.

### Route editing

- Add, move and delete vias; drag an anchor to correct it (writes `anchor_corrections`).
- Recompute geometry and coverage on drag-end, limited to the affected segment's bounding box. Never per frame.
- Per-segment undo stack. Fat-fingered vias on phones are the common case.
- "Reset to automatic" per segment clears vias and corrections for it and drops `is_user_edited`.
- Batch regeneration skips segments with `is_user_edited = 1`.
- Live road-snapping while dragging is opt-in for the segment being edited only.

### Statistics

- Total explored area, accounting for latitude and avoiding duplicate area from overlapping journeys. The representation and calculation are to be evaluated.
- Repeat visits per cell and per place. The exact definition of a "visit" is \[Open\].
- Counts: trips, places, countries, cities.
- Coverage percentage of a city or country needs openly licensed administrative boundaries; the dataset is to be selected.

## Permissions and continuity

- Show a rationale screen before every permission prompt, naming the specific feature it enables. Request access when that feature is used.
- Explicit recording and optional passive tracking are separate modes. Do not ask for always-on tracking merely to import media or record an explicit session. Determine HarmonyOS's actual permission requirements rather than copying Android/iOS permission names.
- Durably preserve accepted observations as they arrive. Process death must not turn a recorded journey into lost history; do not depend on a final lifecycle callback to save it.
- Handle permission denial/revocation, limited media access, unavailable assets, interrupted recording, screen lock, and relaunch honestly. Do not promise continued execution after the OS terminates the process.
- Select a permanent HarmonyOS bundle identifier before distribution. The old temporary identifier and claims about Android/iOS store rejection are not HarmonyOS safeguards.
- Preserve English-only launch and internationalization readiness (D-005); use localized resources from the first UI. A Chinese launch language or changed market scope requires a product decision.

## Compliance requirements

These came out of a legal research pass and translate directly into code. They are requirements, not legal advice; the owner handles the non-code items (privacy policy, DPIA, trademark).

### Map attribution \[Decided\]

- Always visible on the map: "© OpenStreetMap contributors" linking to the ODbL copyright page.
- With OpenFreeMap tiles: "OpenFreeMap © OpenMapTiles Data from OpenStreetMap".
- With Protomaps tiles: "© OpenStreetMap" (Protomaps credit optional).
- Repeat all credits on an About / Licences screen, alongside open-source package licences.

### OpenStreetMap licence (ODbL) \[Decided\]

- Keep road-matched or routed geometry **per user and per segment**.
- Never aggregate OSM-derived geometry from many segments or users into a shared road-network database. That could make it a Derivative Database subject to share-alike.
- Raw GPS traces carry no OSM obligation; only routed geometry derives from OSM.

### Privacy \[Decided\]

- All user data stays on device and is encrypted at rest; keys are protected by the platform. The database and encryption implementation are to be evaluated.
- Any future cloud backup or sync is opt-in and end-to-end encrypted.
- No third-party SDK may receive location, media metadata or notes. Crash reporting, if added, must scrub coordinates.
- Permission rationale and purpose strings must name the specific feature, for example: "FootNoted uses your location during a recording to reveal the places you explore on your personal map."
- Passive mode remains unavailable until its separate consent, platform permissions, and applicable HarmonyOS distribution requirements have been verified. Android store procedures are historical references only.
- HarmonyOS store disclosures and the privacy policy must match actual behavior; update them whenever data flows change. Verify current requirements for the intended market before submission.
- Provide in-app deletion: delete a trip, a segment, a media item, or everything.

### Sharing and export \[Decided\]

- Strip GPS and EXIF from any image or video leaving the app by default, including embedded thumbnails. Keeping location is an explicit opt-in per share.
- Warn before sharing anything that reveals location (a route, a map screenshot, a poster).

## Monetization readiness

The app launches fully free with no paywall. **\[Decided\]** Build two small hooks now so paid tiers can be added later without a retrofit; do not build purchase flows yet.

### Build now

- **Entitlements layer:** a single `Entitlements` service the UI asks ("is cloud sync available?"). At launch every check returns true. Features never test purchase state directly.
- **First-use date:** record the first install / first launch timestamp locally, so early users can be grandfathered when monetization arrives.

### Planned tiers \[Proposed\]

| Tier | Includes | Why it sits there |
| --- | --- | --- |
| Free, forever | Recording, import, the fog map, trips, straight-line and great-circle connections, editing, basic stats | Retention comes from accumulated data, which takes months to build |
| Lifetime unlock (one-time) | Depth features: year-in-review, detailed stats, coverage percentages, heatmaps, advanced export | Runs on device, so near-zero ongoing cost |
| Subscription | Cloud backup and sync, road-snapped routing, offline map packs | Each has a real recurring server or bandwidth cost |
| One-off purchase | Printed posters of the explored map (print-on-demand) | Appeals to users who will never subscribe |

### Rules for later

- Paid features arrive as new additions; nothing free is ever taken away.
- A lapsed subscriber keeps their full map, photos and history; only the hosted services stop.
- The client is open source under MPL-2.0; hosted services may be paid later (source D-001).

## Full product horizon

These are preserved outcomes, not inherited engineering milestones or a release-date promise.

1. Foundation: a dark world map with visible attribution, protected local history, reversible reveal, and correct recovery/rebuild behavior.
2. Import: a real camera roll produces a sensible personal map; photos and videos are supported, duplicates avoided, missing metadata handled, flights shown as arcs, and inferred paths distinguished.
3. Live recording: explicit start/pause/stop, durable history, route photos and notes. Validate a two-hour walk with the screen off on a physical target phone; new engineering thresholds and the device matrix need their own evaluation.
4. Trips and places: suggestions with confirmation, local/transit behavior, return-leg pairing, and map clustering.
5. Statistics: explored area, repeat visits, counts, and administrative-area coverage.
6. Editing: vias, anchor corrections, undo, reset to automatic, and responsive recomputation after edits.
7. Passive mode: optional, off by default, coarse and low power, separately consented and validated.
8. Road routing: opt-in, honors vias, preserves raw history and ODbL boundaries. Server/provider choices are open.
9. Later: soft-edged fog, offline map packs, opt-in end-to-end encrypted backup/sync, paid additions, year-in-review, heatmaps, advanced export, and printed posters.

## Acceptance principles

- Rebuild all derived information from raw observations and edits and reproduce the same map and statistics.
- Adding, editing, and deleting a segment restores correct coverage and counts, including overlaps; deleting one journey never erases another's contribution.
- Mode changes preserve vias and corrections; reset has explicit and tested semantics. Editing never overwrites original coordinates.
- Exercise equator/high-latitude, antimeridian, near-pole, flight, long-drive, overnight-gap, and adjacent-unrelated-trip cases.
- Preserve data through upgrades and interrupted operations; define migration tests for the storage design actually selected.
- Evaluate recording on physical HarmonyOS phones, including screen off, app removal from Recents, power saving, relaunch, and reboot. Record actual behavior; do not assume recording survives each transition.
- Import real photos and videos from different devices, including stripped or absent location metadata.
- Establish HarmonyOS-specific responsiveness, energy, storage, and longevity evaluations. Old benchmark numbers are not new-platform evidence.
- A feature is complete only when its declared acceptance checks pass, no invariant is broken, attribution and rationale flows are present, and documentation reflects its behavior.

## Open product questions

- Launch market, HarmonyOS phone coverage, minimum OS/API, final bundle identifier, and store identity.
- Whether English-only launch should change for the intended audience; inherited default remains English until decided otherwise.
- Time-gap and speed thresholds for segmentation; the ~6 h and ~150 km/h examples above remain candidates for evaluation.
- Place-name/reverse-geocoding source under open-license and privacy constraints; no bulk public-service use may be presumed permitted.
- What counts as a repeat visit: segment, day, trip, or another definition?
- Final transit reveal default and the user-facing transit switch; endpoints-only remains the carried default.
- Future pricing, services, backup/sync design, and offline distribution. Proposed tiers are not implemented purchase commitments.

## Authority and source

Read [decisions.md](decisions.md) for this repo's decisions and [TRANSFER.md](TRANSFER.md) for complete source mapping. The [original handoff](reference/HANDOFF.original.md) and [original decision log](reference/decisions.original.md) are retained verbatim for completeness. Their Flutter stack, frozen seams, test results, dates, and milestones do not govern this implementation. No runnable client or cross-platform compatibility promise is implied by this transfer.
