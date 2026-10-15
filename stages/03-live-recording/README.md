# Stage 03 — Live recording · KICKOFF

**State:** `KICKOFF`. Threads only; planned after Stage 02 exits.
**Horizon outcome:** 3 — "explicit start/pause/stop, durable history, route photos and notes. Validate a two-hour walk with the screen off on a physical target phone; new engineering thresholds and the device matrix need their own evaluation."
**Entry conditions:** Stage 02 exited (routes, segmentation, markers exist); physical HarmonyOS phones available for long walks.

## Objective

The user starts, pauses and stops a recording; the route draws as it happens; fixes keep arriving with the screen off; nothing accepted is lost to process death; photos and notes attach along the way.

## Threads

- **Location stack.** Location Kit updates tuned to "a fix every few hundred metres" (dense tracking is not a goal); `APPROXIMATELY_LOCATION` + `LOCATION`, each behind a rationale naming recording. Never `LOCATION_IN_BACKGROUND` (passive mode only).
- **Staying alive.** A location-type continuous task (`KEEP_BACKGROUND_RUNNING`) started from the visible UIAbility, stopped when the recording stops or pauses; the system suspends or kills work that does not match its declared type. Record what actually happens on screen lock, Recents removal, power saving and reboot; promise nothing the OS does not deliver.
- **Persistent indication.** A clear, platform-appropriate indication while recording (the continuous-task notification; evaluate a Live View if it is open to this app category).
- **Durability.** Each accepted fix written to the store as it arrives, from a context that can write while locked (encryption area ruled at G1); sessions resume or close cleanly on relaunch; nothing depends on `onDestroy`.
- **Live reveal.** Incremental reveal per new fix without blocking the UI thread; batched fog invalidation.
- **Photos along the route.** In-app capture (evaluate the system camera picker, which may avoid the camera permission) or library photos matched to the session by timestamp, as thumbnail markers.
- **Notes.** Text at the current location, alone or with photos (`notes`, note–media links already in schema v1).
- **Energy.** Background CPU and battery measured against HarmonyOS standards (background CPU < 2% is an official rule for idle background; a recording is not idle, so the budget is set at planning).

## Open questions to settle at planning

- Sampling parameters and whether to auto-pause when still.
- Notes UX: on the map only, or also a timeline.
- Which phones form the device matrix for the two-hour walk.

## Likely shape

- **W1:** location service + continuous task + durable writes ‖ session state machine + UI ‖ notes data + UI.
- **W2:** photos along the route ‖ live reveal performance ‖ indication and permission flows.
- **W3 (verification, device-heavy):** the two-hour walk matrix — screen off, locked, removed from Recents, power saving, reboot — owner-run with agent-collected logs.

## Risks

Platform background management; battery drain; distribution review of the continuous-task type; locked-screen storage access.
