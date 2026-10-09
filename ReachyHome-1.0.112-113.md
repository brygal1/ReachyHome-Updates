# Reachy Home 1.0.112

Update dependencies, including Reachy Mini 1.11.0 (#285)

* Update dependencies, including Reachy Mini 1.11.0

Reachy Mini SDK and daemon advance to 1.11.0 after a file-by-file wheel diff:
the ADR-0058 audio seam files are byte-identical, WSClient is unchanged,
protocol changes are additive (IMU), and Pollen's new Rotation helper matches
SciPy within 6e-15 for every sequence the SDK uses. Deploy now requires the
1.11.0 daemon.

Also: OpenAI 3.26.1, Anthropic 1.12.1, ElevenLabs 2.71.0, LiveKit Python
1.1.20/1.2.1 and Server 1.13.7, Starlette override 1.7.0, Sparkle 2.10.0,
LiveKit Swift 2.17.0, uv 0.12.23 (matching Homebrew stable), setup-uv v10.2.0
with an explicit uv checksum, gitleaks 8.30.1, SwiftLint 0.65.1, SwiftFormat
0.63.1, newer dev tools, and the rest of the lock within the seven-day guard.
pip-audit reports no known vulnerabilities.

Pins bound to the signed speech and recognition runtimes, and Reachy Mini's own
ceilings (onnxruntime, huggingface-hub, OpenCV, MuJoCo, gstreamer, websockets),
are held and recorded in docs/dependency-research.md.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Replace the LiveKit container when an update moves its pinned image

reachy-ops update only re-registered the remote gateway's launchd job, so a
release that changes the pinned LiveKit Server image left the old container
running. Post-update verification then saw image drift, reported Mac services
unhealthy, and rolled the update back. The update now runs the existing
transactional remote gateway install when the configured container's image
drifts. Also removes services_install's always-true should_start flag.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Fold duplicate Mac readiness check into one owner

_update_services_ready duplicated _mac_bridges_healthy line for line; updates
and recovery now share the single check. The remote gateway launchd status
builds its record once. Keeps operations/cli.py under its size baseline after
rebasing onto the dependency update.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Replace the LiveKit container only after an update verifies

Replacing it before verification meant a later failure rolled back to code that
saw the new image as drift and ended in attended recovery. Verification now
accepts the image the container was already running, and the transactional
remote gateway install converges it to the pinned image only after the update is
verified and Reachy is asleep. The update result dictionaries share one base.
Also refreshes stale LiveKit and Starlette entries in the blueprint,
attributions, and known limitations (Codex review).

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Fail robot deploy when FastAPI rejects the Starlette override

The deploy's security-floor check now constructs a FastAPI app after forcing
Starlette 1.7.0, so a pip-selected FastAPI that is incompatible with the
override stops the deploy before the robot app is activated.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Move LiveKit to the pinned image after an update commits

The image cutover now runs only after the release is verified and Reachy is
asleep. A failed cutover or a gateway that does not re-verify becomes an
explicit update warning and the existing Repair diagnosis, never a source
rollback (the previous release would treat the new image as drift). The
update dry-run plan lists the post-commit cutover. Refusal results reuse
update_refused_result (Codex review).

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Poll the restarted gateway before warning about a LiveKit cutover

remote_gateway_install returns as soon as the gateway and relay restart, so a
single status read could report a Repair warning while the gateway was still
binding. Verification now polls for up to 45 s before warning (Codex review).

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Report an unfinished LiveKit cutover truthfully and still verify it

The installer deletes its rollback container before the gateway, relay, and
Serve steps that can still fail, so an install error no longer claims the
server stayed on its previous image. The resulting stack is always verified,
and both outcomes are reported as update warnings (Codex review).

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Record setup-uv v10.2.0 in the workflow supply-chain inventory

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Keep LiveKit readiness read errors out of update rollback

A status read that raises after the committed cutover now counts as not yet
verified and ends in the bounded Repair warning, instead of escaping into a
source rollback.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Journal a committed update before the LiveKit cutover

The fresh continuation records its verified, committed result before it
replaces the LiveKit container. If the continuation then dies or times out,
the coordinator returns that committed result with a Repair warning instead of
rolling the source back onto an image the old checkout would reject.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Defer a LiveKit cutover that could outlast the update coordinator

The coordinator that installs this release is the previous one and cannot read
the committed-result journal. The fresh continuation now records the
coordinator's deadline at launch and defers the LiveKit cutover to Repair when
less than its reserve remains, so a slow update never times out mid-cutover
and triggers a source rollback.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
