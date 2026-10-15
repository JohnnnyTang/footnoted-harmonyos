# Stage 04 — Trips and places · KICKOFF

**State:** `KICKOFF`. Threads only; planned after the previous stage exits. It depends functionally on Stage 02 only (imported history is enough to group), so the owner may order it before Stage 03 if phone availability blocks recording.
**Horizon outcome:** 4 — "suggestions with confirmation, local/transit behavior, return-leg pairing, and map clustering."

## Objective

Turn a flat history into **trips → places → segments**, as suggestions the user confirms ("Looks like a trip to Kyoto, 6–14 March. Group these 47 photos?"), with local and transit segments revealed differently and trips collapsing to a pin at low zoom.

## Threads

- **Stay points → places:** observations within a radius over more than a few hours.
- **Trip suggestions:** clusters of places in a time window joined by transit legs; stored as `suggested` until confirmed; origin auto/user. Batch regeneration never touches `is_user_edited` work (invariant 8).
- **Local vs transit:** per-kind buffers; transit reveals endpoints only today (H-004). The final transit default and the user switch are an open product question to put to the owner.
- **Return legs:** pair near-duplicate outbound/return legs (`return_of`); one line with a two-way marker.
- **Trip UI:** list, confirm, rename, merge, split, create empty trips in advance, reassign segments, tags for day trips (no nested sub-trips).
- **Map behaviour:** zoom-to-fit; a trip collapses to one pin at low zoom and expands into places; repeated trips to the same city cluster. Clustering is implemented on the engine ruled at G1.
- **Place names — open and constrained.** Platform reverse geocoding may send coordinates to a service; that conflicts with local-first (invariant 7) unless opt-in, and any provider must be openly licensed (invariant 1). Candidates: an offline open boundary/places dataset, or an opt-in self-hosted geocoder. Likely its own spike or gate.

## Open questions to settle at planning

- Transit reveal default and switch.
- Place-naming source.
- "Same city" clustering thresholds.

## Likely shape

- **W1:** stay-point + trip suggestion algorithms with fixtures (adjacent unrelated trips, overnight gaps) ‖ trip/place repositories ‖ place-naming spike.
- **W2:** trip UI + confirmation prompts ‖ low-zoom clustering ‖ return-leg pairing.
- **W3:** verification on the owner's real imported history.
