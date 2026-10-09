# Reachy Home 1.0.113

Provider-aware Mac services, What's in Use, setup guide, and ChatGPT scene vision (#286)

* docs: make codebase stewardship an explicit agent responsibility

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Require Mac services only for the features that use them

Scene vision gates the vision bridge and its dedicated Ollama; shared memory gates
the memory bridge. Status reports desired/required_by, diagnosis names stopped
services in household terms with their consequence, and Reachy Home ignores
services the household does not use. The Mac-service banner yields the primary
action when a full-pane recovery view is shown. ADR-0072.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Show what's in use and group brains by what leaves the house

The provider catalog now states each engine's data boundary (on this Mac, cloud
brain with local voice, or cloud conversation). Reachy Home groups the engine
menu by that tier, explains the boundary under the picker, and adds a What's in
Use card to Overview and Brain built from the saved selection. Overview's
services state and service symbols move to their own file, leaving the 750-line
cap.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Add a self-checking setup guide and a clearer toolbar

Set Up Reachy walks through connecting, Mac services, choosing where Reachy
hears, thinks, and speaks, and finishing the brain's setup. Each step reads live
state from its owner, opens once automatically when Reachy was never configured,
and stays available from the Dashboard menu.

The toolbar keeps state-bearing actions visible and moves Reconnect and the
hardware app into a labelled More menu. Emergency Stop is a labelled red button
with its availability explained.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Follow ChatGPT's relocated bundled Codex executable

The 2026-10-07 ChatGPT release moved its bundled Codex into
Contents/Resources/codex-cli/CodexCLI.app, so the Mac Brain's pinned path no
longer existed and the ChatGPT brain could not start Codex. The bundled build
(0.162.0-alpha.2) also exposes the GPT-6 model catalog.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Answer camera questions on isolated ChatGPT threads

The ChatGPT engine can now describe one JPEG frame on an ephemeral, tool-less
thread: Reachy's MCP server is disabled per thread, the frame is sent only as an
in-memory data URL, any action item discards the answer, and the thread is
released with thread/unsubscribe. Its notifications use a private bounded queue,
so vision never contends with a live voice turn. The model catalog now marks
image-capable models.

Verified live against the ChatGPT-bundled codex 0.162.0-alpha.2 with gpt-6-luna
at low effort: 2.4-3.2 s answers, concurrent turns, injected image text ignored.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Answer scene vision with the ChatGPT subscription (ADR-0073)

perception.scene_vision.provider selects ollama or chatgpt. The household
profile uses gpt-6-luna at low effort through new authenticated Mac Brain
routes, for explicit camera questions and presence- and change-gated ambient
observations. Ambient frames request the {summary, salient} schema at background
priority and yield to explicit questions as a deferral. A factory owns provider
construction and the frames policy, so cloud frames are gated by
allow_cloud_camera_frames.

The vision Ollama and vision bridge are now required only for Ollama scene
vision, and a vision health failure no longer repairs an unused launchd agent.
The privacy model and dependency research record the new boundary.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Show where camera frames go in Reachy Home

The vision settings API reports the scene-vision provider, where frames go, and
a one-line boundary, owned by vision/factory.py. Reachy Home's Scene Vision card
states that instead of claiming vision runs on this Mac, and What's in Use adds a
Sight row whose boundary joins the conversation footnote. Vision models move to
ReachyModels+Vision.swift.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Record the feature-gated services and ChatGPT vision boundaries

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Release ephemeral ChatGPT voice threads with thread/unsubscribe

Every Reachy voice thread is ephemeral, and the App Server rejects thread/delete
for those ('thread is not persisted'), which the cleanup path suppressed, so
released threads stayed loaded. thread/unsubscribe lets the server unload them;
verified live on the ChatGPT-bundled codex 0.162.0-alpha.2 (unloaded ~60 s
after release).

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Keep operations/cli.py at its size baseline after rebasing

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Keep Reachy Home working during the app-first update handoff

Sparkle installs the app before the backend updates, so for that window
reachy-ops and the robot API still return the previous release's payloads. The
new launchd 'desired' flag, catalog data boundary, and vision boundary fields
are optional in Reachy Home: a missing 'desired' means required (the previous
contract), engines without a boundary stay selectable, and the What's in Use
summary and Sight row are omitted rather than guessed. ChatGPT vision health
uses the engine's cached model catalog instead of a model/list call per poll.
The Sight row's 409-means-off rule is extracted and tested.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Address PR review: shared memory, bulk actions, vision effort, late starts, setup readiness

- Keep the shared-memory bridge always required: its enablement is a persisted
  robot setting the Mac cannot read, so static config must not remove it.
- services start/restart/stop 'all' expand to the agents this setup requires,
  like services install; named targets stay exact.
- ChatGPT vision offers only models that support the configured reasoning effort.
- Vision thread/turn starts that land after the deadline are reclaimed
  (unsubscribed or interrupted) by owned tasks instead of leaking.
- The setup guide requires the saved brain's functional health for every
  provider, not only ChatGPT.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Address PR review: setup guide reload, doctor secrets, stop all

- The setup guide reloads when the connection changes, so connecting from
  step 1 shows the brains without a manual refresh.
- Doctor lists the admin token plus the secrets of the agents this setup
  requires; ChatGPT vision no longer reports a missing vision token.
- services stop all stops every managed agent, including ones the setup no
  longer requires but that may still run.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Keep Reachy Home's toolbar actions operable by assistive technology

A toolbar Menu (and a split-button Menu's arrow segment) exposes no press
action or description, failing the launch accessibility audit. Reconnect and
Open Reachy Mini Control are plain labelled toolbar buttons again.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Probe the ChatGPT account before reporting scene vision healthy

Vision health now asks the Mac Brain's canonical ChatGPT health (credential
refresh and usage limits) before checking the cached model catalog, so a
signed-out or rate-limited account no longer reads as ready vision.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Recheck the setup guide's brain readiness after a switch

The guide now loads the API-key provider's key status and models through the
same store path as the Brain page before probing health, and its reload
identity includes the saved provider, so a switch made from the guide is
health-checked without a manual refresh.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Reload the What's in Use card after Reachy reconnects

The card's load task now keys on the connection identity as well as the
refresh token, so a card shown before the tunnel connects fills in once the
session becomes active.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Keep the setup guide from passing brains it cannot health-check

ElevenLabs and Hugging Face realtime run on the robot with no functional
probe, and the catalog health gate treats them as ready from configuration
alone. The guide now never checks step 4 for such a brain and tells the
household to confirm it by talking to Reachy instead.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Tell a broken scene vision apart from one that is turned off

The vision settings API now answers 409 only when scene vision is disabled in
config, and 503 when it is enabled but could not be constructed, so What's in
Use no longer shows Sight: Off for a configured-but-broken camera brain. The
setup guide records that it was offered only once it is actually shown.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Report vision turn faults as unavailable and name self-hosted realtime servers

- Only a model or effort that cannot run vision is a 422 now; timeouts,
  refused or malformed turns are 503, so the robot no longer treats a
  transient fault as an unusable model.
- What's in Use names a self-hosted Hugging Face realtime server by its host
  and counts it as staying home only for loopback, .local, or private
  addresses, instead of labelling it Hugging Face cloud.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Show a home icon for a conversation that stays on the home network

What's in Use took its icon from the catalog tier, so a self-hosted realtime
server on the LAN still showed a cloud. The icon now follows the resolved
rows: when every part stays home, the card shows a house.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Show a cloud icon whenever any part of the setup leaves the house

The What's in Use icon now considers every resolved row, including Sight, so a
local brain whose scene vision sends frames to ChatGPT no longer shows the
all-local icon.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Hold a vision turn's private route until cleanup settles

The vision turn released its private notification route before remote
cleanup ran and before a late turn/start reclaim settled, so late events could
fall through to the shared voice queue. The route is now released after remote
cleanup, or by the late reclaim task once it settles.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Skip a disabled vision bridge's endpoint in the Mac address check

The bridge address check validated the Ollama vision endpoint even when
local scene vision was off, so a stale endpoint could fail Mac services and
block Repair for a bridge the household no longer uses. The endpoint is now
checked only when the vision agents are required.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Fail ambient capture closed on stale presence; reclaim slow vision starts

- PresenceService owns the ambient capture rule: with presence automation on,
  the room must be occupied on a live Home Assistant link. Occupancy kept
  from a dropped stream no longer lets ambient frames go to ChatGPT.
- Vision thread/turn starts get a bounded five-minute reply window beyond the
  client's 30 s default, so a start accepted late is still reclaimed instead
  of leaking an active thread or turn.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Release a failed vision start's route and cap pending late reclaims

- A turn/start that fails outright no longer strands its private thread
  route: the route is handed to a reclaim task only when a late start is
  actually pending, and is otherwise released after remote cleanup.
- At most two late-start reclaims (each may hold a frame) can be pending; new
  vision turns fail fast as unavailable until they settle, instead of
  accumulating while the App Server is wedged.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Normalize malformed vision replies; scope the What's in Use boundary

- The ChatGPT vision provider reads every Mac Brain reply through one
  object parser, so invalid or non-object JSON is VisionUnavailable rather
  than an opaque ValueError/AttributeError.
- When a camera sentence is appended, the summary scopes both sentences
  ("Conversation: … Camera: …"), so "Nothing leaves the house" no longer
  reads as covering a camera that sends frames to OpenAI.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Count household tailnet and IPv6 unique-local hosts as staying home

A self-hosted realtime server on the household Tailscale tailnet
(100.64.0.0/10 or *.ts.net), or an fc00::/7 unique-local IPv6 address, stays
on household devices, so What's in Use no longer labels it as leaving the
house.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Scope the ChatGPT camera storage promise to Reachy

The boundary shown in Reachy Home said frames sent to OpenAI are "never
stored", which Reachy cannot guarantee once a frame leaves the house. It now
says Reachy never stores frames, matching ADR-0073 and the privacy model.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Re-read saved brain settings on every new connection

The setup guide and What's in Use skipped loading the saved settings once
any cache existed, so after a reconnect (for example across an update) they
could probe or describe a stale provider. The store now records the
connection its settings were read on, and both views reload when it differs.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Bind saved settings to their connection; reject vision turns without an id

- loadCore captures the connection before requesting and discards a reply
  that arrives after Reachy reconnected, so stale settings are never stamped
  as current.
- A turn/start reply without a turn id is an immediate vision error (with
  cleanup), not a wait until the deadline on a turn that cannot be
  interrupted.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
