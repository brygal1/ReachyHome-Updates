# Reachy Home 1.0.10

Fix backend update/repair (subprocess PATH) + opt-in auto-apply (#110)

* Auto-apply backend update when the app is newer (opt-in, robot-asleep only)

The macOS app self-updates via Sparkle, but the Python backend (robot brain + Mac
bridges) previously only updated on a manual banner click, so it could silently keep
running stale code (e.g. app 1.0.8 vs backend 1.0.6), breaking features like Claude Max
sign-in.

Add an opt-in coordinator that automatically brings the backend to the app's version
when it is safe, reusing the existing reachy-ops update flow (sleep -> deploy -> restart
-> verify -> rollback-on-failure). The manual banner stays as the fallback.

Safety gate (AutoBackendUpdate.shouldAutoApply, pure + unit-tested): opted in, an update
is available, reachy-ops present, nothing else running, e-stop clear, robot asleep (never
mid-conversation), and not a target that just failed (cooldown -> can never loop).

- AutoBackendUpdate.shouldAutoApply + AutoBackendUpdateCoordinator (@Observable): runs
  applyBackendUpdate() when safe, surfaces a visible non-modal progress/failure banner,
  refreshes version on success, cools down on failure.
- Settings toggle "Keep Reachy's software in sync automatically" (default on).
- ContentView: re-evaluate on life-state/version/preference changes; the progress banner
  stands in for the manual one while applying.

Verification: testAutoBackendUpdateGate covers the gate matrix; ReachyHome make build and
make test both succeed.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

* Fix reachy-ops subprocess PATH so update/repair can find their tools

Root cause of "Reachy couldn't check for an update" and the stuck "Mac services need
attention": MacServiceProcessRunner.run() never set process.environment, so a
Finder-launched app handed reachy-ops a minimal PATH (/usr/bin:/bin:/usr/sbin:/sbin).
The update and repair flows shell out to docker, uv, git, rsync, and ssh — which live in
/opt/homebrew/bin, /usr/local/bin, and ~/.local/bin — so the child died with
"[Errno 2] No such file or directory: 'docker'", surfaced to the household as an opaque
error. Repair uses the same runner, so it failed identically and never recovered the
services, which then blocked the update.

Add MacServiceEnvironment.augmented()/augmentedPATH (pure, testable): prepend the standard
tool directories to the inherited PATH (de-duplicated, order-preserving) and set
process.environment in run(). One fix mends the whole chain: repair works -> services
healthy -> update works -> the opt-in auto-apply coordinator can run.

Verified against the live deployed checkout: with a minimal PATH, `reachy-ops --json
update --dry-run` and `reachy-ops --json doctor` both fail on 'docker'; with the augmented
PATH both succeed (the update then correctly reports its real health guard). Unit test
testMacServiceEnvironmentAugmentsPathForToolLookups covers the augmentation.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

---------

Co-authored-by: Claude Opus 4.8 <noreply@anthropic.com>
