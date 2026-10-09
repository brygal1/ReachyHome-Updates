# Reachy Home 1.0.114

Make the Mac the single owner of Reachy's admin token (#287)

* Make the Mac the single owner of Reachy's admin token

The household admin token was a dotenv comment's text in three hand-maintained
copies; the 1.11.0 daemon update's python-dotenv read it as empty and the
robot app refused to start (ADR-0074).

- reachy_agent.api.admin_token owns what a usable token is; the robot refuses
  a missing, comment-like, or short token and doctor reports it.
- reachy-ops generates a real token in the Mac credential store when needed
  and pairs it to the robot on every deploy and on `robot repair`.
- Reachy Home reads the token from that store (HouseholdCredentials); the
  Keychain copy and the Settings field are gone. A token mismatch or a robot
  app that won't answer offers one action: Fix and Restart Reachy.
- launchd: bootstrap re-enables an agent a stop disabled, and the updater waits
  for launchd to finish unloading a retired helper before checking it.
- The robot-side credential holder script moves out of cli.py into its own
  module (cli.py -439 lines); DashboardSection gets its own file.
- Checklist and Hey Reachy review notes in docs/.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Point the admin-token error at Fix and Restart Reachy

The Settings token field is gone, so a 401 now tells the household to use the
repair action instead. Components.swift is reformatted to the project style.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Generate the admin token before deploy's preflight; keep repair out of simulation

- A deploy (and any update) generates a missing or unusable admin token before
  its credential preflight, so a fresh Mac never needs a separate repair first.
- reachy-ops robot repair --dry-run previews generation and pairing without
  requiring the token it would create.
- A simulator that is not listening keeps its own guidance instead of offering
  the physical robot repair.
- services install drops enable calls _bootstrap now makes before loading.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Show why a robot repair failed and offer it on every page

- A Fix and Restart Reachy that does not work leaves a plain-language reason
  and next step in the banner and the failure view.
- 'Reachy's software isn't running' is part of the one household status, so
  the repair banner appears on Reachy Remote and Integrations too.
- A reconnect clears the software-failure flag before validating settings.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Offer robot repair when a live connection's software stops answering

Recovery remembers its last error. When the retry cycle ends on a Control API
failure (SSH reaches Reachy but its software is silent), Reachy Home marks the
software as not answering and offers Fix and Restart Reachy instead of
retrying forever; network losses keep retrying. A later healthy probe clears it.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Read the credential store only when reachy-ops would accept all of it

HouseholdCredentials now parses the whole store with the same rules as
_parse_runtime_environment: a malformed line, duplicate key, invalid key, or
oversized entry means no token, so Reachy Home and the tooling can never
disagree about the household admin token.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Rotate the token only with its pairing; tighten what counts as a token

- A deploy no longer rotates the admin token before its preflights. The token
  is generated (if needed) and paired as one step just before the app starts,
  so a failed deploy can never leave the Mac and the robot disagreeing.
- A usable admin token is URL-safe ASCII only, matching what reachy-ops
  generates; anything else (including Unicode compare_digest rejects) is
  'not a generated secret'.
- Reachy Home requires the same credential-file metadata as reachy-ops: mode
  exactly 0600 and a single link.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Pick up a rotated admin token automatically; reject padded tokens

- When a Control API request is unauthorized and the credential store now
  holds a different token (an update or repair rotated it), ReachySession
  rebuilds its client instead of reporting a mismatch.
- A token with surrounding or inner whitespace is not a generated secret on
  either side; neither the robot policy nor Reachy Home trims it, so the
  value paired and the value sent can never differ.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Make Fix work against the previous backend; one token policy in Reachy Home

- On the first launch after a Sparkle update this Mac's backend can still be
  the previous release, whose reachy-ops has no 'robot repair'. Fix then
  restarts the robot app with the command that release has (ADR-0042
  app-first handoff) instead of failing on an unknown command.
- Reachy Home applies the generated-token policy (32+ URL-safe ASCII
  characters) before using a stored token, matching admin_token_problem.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Validate the credential directory; no restart fallback for an old backend

- HouseholdCredentials mirrors _validate_private_directory: the store's
  directory must be this user's, mode exactly 0700, with no symlinked path
  component, before any token is read.
- When this Mac's reachy-ops predates 'robot repair' (the first launch after
  a Sparkle update), Fix says plainly that the Mac is still finishing its
  software update instead of running a restart that cannot provision.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
