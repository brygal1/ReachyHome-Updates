# Reachy Home 1.0.49

Fix non-realtime voice latency and barge-in; move CI to self-hosted runners (#203)

* Make sentence-streamed TTS canonical for every pipeline brain

The bridge's sentence-release machinery (StreamingSplitter/SegmentSpeaker)
was gated to provider_name == "chatgpt", so local/Ollama, claude, openai,
and claude_max waited for the entire LLM completion before synthesizing
any audio — the dominant perceived-latency term for non-realtime brains
(docs/voice-pipeline-audit-2026-08-07.md).

- tts_streaming_enabled (renamed from chatgpt_tts_streaming_enabled)
  applies to all five Mac-bridge engines; off stays the whole-utterance
  escape hatch and measurement baseline.
- Legacy key migrates on load and on API input; serialization projects it
  alongside the new key so pre-rename consoles keep decoding readbacks.
- Reachy Home reads the renamed key as an optional field, tolerating a
  brain on either side of the rename.
- MacBridgeBackend / MAC_BRIDGE_BACKENDS in voice/backend_settings.py now
  own pipeline-backend membership; main.py, operations/cli.py, API routes,
  and the latency fingerprint project from it.
- Lock upgrades clear pre-existing pip-audit CVEs (aiohttp, cryptography,
  h2; h2 needed an exclude-newer-package override for its 2026-08-03 fix).

Verified: make quality green (LOC ratchet tightened), make test-coverage
3,764 passed / 2 expected xfails / 97% changed-line coverage, simulation
lane 14 passed with the end-to-end test re-pinned to the streamed shape,
Reachy Home make test green with new rename round-trip cases.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Stream ChatGPT speech deltas into sentence-level TTS

The Codex App Server engine buffered the entire schema-bound reply and
yielded it only after turn/completed, so sentence streaming could never
overlap ChatGPT generation — the turn was silent for the whole completion
(docs/voice-pipeline-audit-2026-08-07.md).

- voice/chatgpt/speech_output.py owns the speech-output contract: the
  turn/start schemas and PartialSpeechExtractor, an escape-safe
  incremental decoder for the speech value in item/agentMessage/delta
  fragments.
- _run_turn yields each newly decoded span mid-turn, then reconciles with
  the authoritative item: unvoiced tail only, and a divergent stream
  fails the turn instead of voicing anything twice.
- Diagnostic turns remain whole-turn (canary verifies before voicing).
- Complexity kept under the ratchet by extracting _authoritative_speech;
  the deadline-release test now pins the authoritative-only path
  explicitly since mid-stream yields are bounded by the live turn.

Verified: make quality green, make test-coverage 3,775 passed (new
fragment/tail/divergence and extractor tests), simulation lane 14 passed.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Barge-in fast path: robot VAD monitors bridge sessions

Bridge sessions muted the on-robot VAD for their entire lifetime, so
barge-in needed a Mac round trip through a second VAD plus a third
receipt-timed confirmation, then had to win a race against the bridge's
own end-of-speech or be silently discarded as echo
(docs/voice-pipeline-audit-2026-08-07.md, confirmed findings #4-#7).

- VadService keeps running under provider ownership and publishes on the
  monitor source (audio.vad.monitor); ownership now moves boundaries
  between streams instead of stopping inference. Stream-labelled aborts
  keep correlation intact across ownership changes.
- DialogueSession filters monitor boundaries out of STT turn admission;
  the bridge's endpointing stays authoritative for what reaches STT.
- BargeInDetector seeds its window from the event's back-dated onset
  (no stacked double-confirm), re-arms across sentence-streaming
  playback gaps, and accepts voice.local.vad.reset aborts so an
  upstream-invalidated boundary cannot leave a phantom timer.
- The playback-overlap discard is unchanged by design: with a local fast
  path it returns to being an echo gate rather than a lost race.

Verified: make quality green (session.py shrank below baseline),
make test-coverage 3,778 passed with rewritten VAD ownership contract
and three new detector cases, simulation lane 14 passed.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Fix G3 barge-in memory loss and the swallowed tail cancellation

A barge-in mid-reply erased both sides of the exchange from bridge
history on the transcript-rebuilding engines (ollama/claude/openai):
cancellation unwound before the history commit, and the voiced-sample
count died with the cancelled task. Voiced samples are now folded into
turn state as each chunk is sent to the robot, and the CancelledError
path commits the partly-heard exchange — guarded on audio actually sent,
so an unheard superseded reply still leaves no fabricated memory. The
strict xfail pinning G3 now passes; known-limitations updated.

SegmentSpeaker.finish also no longer suppresses CancelledError around
the tail enqueue, which could swallow a barge-in exactly when synthesis
lagged generation — a path every pipeline brain now exercises with
sentence streaming canonical.

Deferred with rationale (new known-limitation G5): per-request STT/TTS
worker aborts. Verified already-covered: robot-side flush on cancel —
stale-turn audio is dropped on arrival by the dialogue session.

Verified: make quality green, make test-coverage 3,779 passed
(1 remaining expected xfail: G2), simulation lane 14 passed.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Make speech-stack prewarm canonical; drop the blocking session warm

Every non-ChatGPT bridge session gated session_started on a blocking
silence transcribe (up to the full cold model load), while the prewarm
coordinator — the ChatGPT path's answer to the same problem — only
accepted the exact chatgpt/parakeet/qwen3 combination
(docs/voice-pipeline-audit-2026-08-07.md).

- The coordinator warms whatever speech stack the session selects, for
  any pipeline brain; provider identity left the selection key, so brain
  switches over the same stack stay warm. WhisperKit warms only the TTS
  side (it is an external service with its own lifecycle).
- The robot-side trigger fires for any Mac-bridge brain on verified wake
  or session start, mapping local -> ollama on the wire and carrying the
  selected engines and voice.
- The inline _warm_recognizer and its provider conditional are deleted;
  a cold model loads on demand at the first turn, never at session start.

Deferred with written rationale: optimistic STT during input_accepted,
and binary frames for outbound TTS audio.

Verified: make quality green, make test-coverage 3,779 passed,
simulation lane 14 passed.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Abort abandoned requests in the STT/TTS workers (G5)

Barge-in cancellation stopped the Mac-side wait but the workers ran
abandoned inference to completion, head-of-line blocking the next
request — including transcription of the very utterance the user
interrupted with (docs/voice-pipeline-audit-2026-08-07.md finding #8).

The protocol is strictly request-then-respond, so inbound readiness
after a complete request (abort byte or client EOF) is unambiguous
abandonment — no wire change needed:

- TTS worker: checks before each generated chunk; synthesis stops
  within one ~32 ms chunk and the upstream generator is finalized via
  contextlib.closing rather than garbage collection.
- STT worker: checks after reading the full request; a request whose
  client vanished in the accept backlog spends no forward pass.
- Lifecycle guarantees (SIGTERM cleanup, cold-start ceiling, model
  residency) unchanged; known-limitations G5 narrowed to the residual
  mid-inference Parakeet window.

Verified: make quality green, make test-coverage 3,781 passed with two
new socketpair contract tests, simulation lane 14 passed.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Fix pre-PR review P1/P2 findings: monitor-VAD isolation, console draft, surrogates

- Speaker identity and live speaker fusion skip monitor-source VAD
  boundaries: with the barge-in monitor active for whole bridge sessions,
  a monitor start could steal the identity turn correlation from the
  authoritative bridge boundary and misattribute the utterance's speaker.
  Complexity paid down by extracting the speech handlers and deduplicating
  the active-speech reset; both behaviours pinned by new tests.
- Reachy Home buildDraftSettings round-trips ttsStreamingEnabled; the
  omission kept engineIsDirty permanently true and silently reset an
  explicit false on the next save. Pinned by a round-trip XCTest.
- PartialSpeechExtractor holds back a lone trailing high surrogate so a
  delta boundary splitting an emoji's two \uXXXX escapes can never emit
  text that fails UTF-8 encoding downstream. Pinned for both split points.

Verified: make quality green, make test 3,785 passed, simulation lane 14
passed, Reachy Home make test green.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Align docs with the shipped rollout (pre-PR review P2/P3 findings)

latency-research.md (summary, §5.1 prose, §5.7 status),
natural-conversation-latency-experiment.md, voice-tuning.md,
voice-pipeline-audit-2026-08-07.md (audit-base framing, LANDED vs
DEFERRED status, unverified-findings wording, G3 pin status),
known-limitations.md (G-number registry note; tts_streaming_enabled
rename upgrade-order and rollback note), progress.md (G2 reference),
and the bridge_wire.py threshold comment now match shipped behaviour.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Stop pre-push gates from inheriting Git's hook environment

Git exports GIT_DIR/GIT_INDEX_FILE/GIT_WORK_TREE to every hook, and
prepush.py handed that environment straight to its gates. Those variables
outrank a subprocess's own -C and cwd, so unit tests that build throwaway
repositories in tmp_path and shell out to real Git retargeted the actual
repository: `git add .` staged deletion of every tracked file and
`git init` rewrote the real config, flipping core.bare to true and
breaking every subsequent Git command in the checkout.

- prepush.py scrubs GIT_* from the gate environment, keeping only
  credential plumbing (GIT_ASKPASS, GIT_SSH*, GIT_TERMINAL_PROMPT).
- The throwaway-repository test in test_cli.py builds an explicit
  Git-free environment so no caller's environment can retarget it.
- Both pinned by tests, including a regression test naming the failure.

Affected checkouts were repaired in place; no commits or tracked content
were lost.

Verified: make quality green, prepush and operations CLI suites pass.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Run CI on the household's self-hosted macOS runners

The repository is private, so every GitHub-hosted minute is billed while
self-hosted execution is free. CI, the Apple client lane, and the auto
release orchestration now run on Reachy-M4-1..3
(runs-on: [self-hosted, macOS, ARM64, reachy]).

- .github/actionlint.yaml declares the custom labels; without it the
  workflow gate rejects every self-hosted runs-on.
- The Linux media apt-get steps are guarded with runner.os == 'Linux'
  rather than deleted: macOS gets GStreamer from the project's wheels,
  and a Linux lane can be restored without rewriting the steps.
- apple-client pins DEVELOPER_DIR to the host's Xcode path; its existing
  version assertion still fails closed if that toolchain changes.
- Release macOS deliberately stays hosted, annotated in the workflow and
  docs/ci.md: its signing step replaces the user's keychain search list,
  which is harmless on an ephemeral image and disruptive on a Mac in use.

docs/ci.md documents the runner topology, the label requirement, and that
the runner host is also the live brain host, so CI contends with voice.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Run every lane on one self-hosted runner, releases included

The first three-runner attempt failed in two structural ways that don't
exist on GitHub's one-fresh-VM-per-job model:

- Both Apple lanes ran concurrently against the same desktop GUI session.
  One won; the other died with "Early unexpected exit ... crashed with
  signal kill before establishing connection" and Gatekeeper put a
  "damaged and can't be opened" dialog on the developer's screen.
- make test-coverage starved under three concurrent jobs plus the live
  brain and tripped a 120s pytest-timeout on a contract test that passes
  in seconds on an unloaded machine.

One runner removes both without a concurrency group, extra labels, or a
workflow split. CI is slower end to end, which is the right trade here.

Release macOS also moves off GitHub, with its keychain handling made safe
for a Mac someone actually uses: the signing keychain is now prepended to
the user's search list instead of replacing it, and an always() step
restores the prior list and deletes the temporary keychain so a failed or
cancelled release cannot leave the login keychain out of the search path.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Ad-hoc sign the macOS UI test host; finalize abandoned TTS streams

Two independent defects, both surfaced by finally running CI.

Apple Silicon refuses to exec an unsigned binary, and the reachy-home lane
built with CODE_SIGNING_ALLOWED=NO — so the generated
ReachyHomeUITests-Runner.app was SIGKILLed at launch ("Test crashed with
signal kill before establishing connection", plus a Gatekeeper "damaged"
dialog). The lane had never run in CI because Actions was disabled on the
repository, so this was latent rather than a migration regression. It now
uses Xcode's local ad-hoc identity, exactly as the passing reachy-remote
lane already does; verified by running .github/scripts/apple-ci.sh locally.

G5's "aborts within one chunk" guarantee did not hold for the real barge-in
caller: _speak_response consumed the TTS stream with a bare
`async for ... return`, abandoning the generator mid-suspension so its
finally — the socket close the worker detects — waited for async-generator
GC. The worker kept synthesizing into a connection nobody read, which is the
stall G5 exists to close. Both consumers now use contextlib.aclosing (the
pattern this file already used for its LLM stream), and
TextToSpeechEngine.synthesize is typed AsyncGenerator so the protocol admits
aclose(). Pinned by a test that fails without the fix.

Also: the release keychain backup/restore reads one array element per line
instead of word-splitting, so a keychain path containing a space cannot
shatter into bogus paths (bash 3.2-safe).

Verified: make quality green, make test-coverage 3,787 passed, simulation
lane 14 passed, apple-ci.sh reachy-home passes locally.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Keep release signing unambiguous; record the xcodegen version

Making the keychain search list additive (previous commit) protected the
developer's login keychain but introduced a worse risk: ExportOptions.plist
selects the identity by name ("Developer ID Application", team N329UMS8HK),
so a searchable login keychain holding a second copy of that certificate can
make the export ambiguous or sign with the wrong one. A release must not be
a coin flip.

The search list is narrowed to the temporary keychain again — exactly the
prior signing behaviour — and safety now comes from the always() restore
step instead. docs/ci.md records the residual (a hard kill between the two
steps) and the one-line recovery.

brew install xcodegen is a silent no-op on a persistent runner, so it now
installs only when missing and records `xcodegen --version` in the release
log rather than implying each release pinned it.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>

* Run monitor VAD only inside the barge-in window

Profiling the CM4 after deploying this branch showed the barge-in fast path
regressed CPU: keeping the robot VAD alive under provider ownership meant
continuous inference for the whole bridge session, where the old code
short-circuited every frame. Barge-in is the only consumer of monitor
boundaries and only matters during playback, so that work was wasted.

Inference is now gated on (not provider_owned) or barge_in_window. The
window opens on SpeechOutputStarted and closes 1s after SpeechOutputEnded —
past playback's 300ms idle timeout, because sentence-streamed replies
flicker that signal at synthesis gaps and closing on the edge would disarm
the monitor mid-reply, recreating the dropped-barge-in defect this branch
fixes. Both transitions reset hysteresis so no stale state leaks.

No capability cost: the monitor is armed for all of Reachy's speech plus a
second of margin. Pinned by two tests.

Verified: make quality green, 3,788 tests, simulation lane 14 passed.

* Give motion sends one dedicated FIFO worker

Actuator writes run at the scheduler's 50 Hz tick and measured ~86% of the
default executor's call volume, queueing behind camera and DoA work on the
CM4 (that shared pool carried ~39% of the agent's CPU). They now use one
dedicated worker in robot/motion_sender.py, via loop.run_in_executor rather
than asyncio.to_thread — to_thread copies contextvars on every call, pure
overhead 50 times a second for a send that reads none.

Deliberately still awaited to completion and strictly ordered: the
scheduler counts send failures, escalates backoff, and treats a returned
send as delivered, and its last-sent bookkeeping assumes FIFO. A cheaper
fire-and-forget handoff would silently break all of that.

Extracted to its own module because reachy_sdk.py is grandfathered and may
only shrink (1280 -> 1275, baseline re-tightened).

Verified: make quality green, 3,790 tests, simulation lane 14 passed.

* Resolve the Codex pre-merge review

Bugs: BargeInDetector.start() reset only the motion guard, so a restart
during overlapping speech re-armed from a stale onset and could interrupt a
new turn with no user speech (Codex reproduced it). The G3 partly-heard
commit credited queued-but-unspoken text, letting an interrupted turn claim
words the user never heard — text is now credited only when its synthesis
emits PCM. Session start still blocked on TTS prepare() for non-ChatGPT
brains via registry.resolve, undoing part of the warm-skip.

Silent failures: a PlaybackRejectedError killed barge-in for the rest of the
session; both interrupt steps are now independent. A non-cancel failure
after audio started dropped the audible exchange from history. Prewarm
failures hid their cause behind a ChatGPT-specific message.

Simplification: AGENTS.md forbids compatibility shims here, so the
tts_streaming_enabled rename is hard-cut — legacy dict, pre-validator,
dual-write serializer, AliasChoices, optional Swift field and migration
tests removed. The robot's persisted document was rewritten to the new key
as part of rollout.

Safety: the vad-barge-in-window-close task is now awaited with error
handling, per the task-lifecycle rule.

Verified: make quality green (session.py shrank via real de-duplication),
3,788 tests, simulation 14, Reachy Home tests green.

* Let personality own reply length on ChatGPT turns

The ChatGPT output schema described the speech field as "non-empty
concise text ... For unclear or inaudible input, ask the speaker to
repeat it". That is response policy, not structure, and it rode on
every turn — so it silently outranked the personality sliders
(chattiness 100 / verbosity 77 still produced clipped two-word
replies), and it instructed the model to answer a poor transcript with
a spoken "say that again" instead of leaving that decision to the
pipeline's own empty-transcript guard.

The schema now describes structure only. The test asserted against a
duplicated copy of the literal; it now asserts against the canonical
constant.

* Stop dropping speech that dips below the VAD threshold

BridgeInputVad required min_speech_ms of *unbroken* above-threshold
audio to confirm an utterance, resetting the accumulated evidence on
the first sub-threshold chunk. Connected speech dips below any fixed
threshold constantly — at plosive closures, between words, and
throughout quieter or more distant delivery — so the run restarted
before it could reach the bar and the utterance was dropped with no
turn, no error, and no log line. Loud close speech worked; everything
else did not, which is why Reachy "could barely hear" half the time.

Feeding one chunk per probability through the real detector, at
threshold 0.5 / min_speech_ms 200 (seven 32 ms chunks):

  steady 0.95 .................. detected      detected
  flickering 0.4-0.7 ........... DROPPED  ->   detected
  two up, one down ............. DROPPED  ->   detected
  dips every fifth chunk ....... DROPPED  ->   detected
  silence / click / sparse
    noise / two short blips .... rejected      rejected

Onset now spends a bounded dip budget instead of discarding the
evidence, and the confirmation window — which sizes pre-roll — widens
to match so the start of the utterance is still captured. Production
pre-roll is 1000 ms and already dominates, so retention is unchanged
there.

Separately, endpointing silence goes 400 ms -> 700 ms. 400 ms is
shorter than an ordinary mid-sentence pause, so one spoken thought was
being cut into two turns and answered twice ("Okay, my man." / "It's
been a little while."). This costs 300 ms on every reply.

* Apply the same onset tolerance to the robot-side VAD

VadService had the identical defect just fixed in BridgeInputVad: any
sub-threshold frame reset the accumulated onset evidence, so speech had
to clear min_speech_ms in one unbroken run. Its own docstring called
that "hysteresis", but a hard reset is the opposite of hysteresis.

This detector arms barge-in rather than endpointing utterances, and the
all_on_robot profile asks for 600 ms of speech before an interruption
counts — 600 ms with no dip is a stretch ordinary speech does not
produce, so interruptions went unregistered. Onset now spends the same
bounded dip budget; sustained silence still retires the candidate, and
intermittent noise still never arms.

The budget is deliberately a separate constant from the bridge's: this
detector sees 20 ms capture frames and gates interruption, the other
sees 32 ms Silero chunks and cuts turns, and the two may need different
tolerances.

Note this does not on its own restore barge-in. The motion guard is
held permanently active by the robot's own speech:presence gesture,
which suppresses interruption regardless of what the VAD reports; that
is tracked separately.

* Remove the second copy of ChatGPT reply-length policy

Codex caught that the constraints stripped from SPEECH_OUTPUT_SCHEMA in
da5dcf93 were duplicated verbatim in _DEVELOPER_INSTRUCTIONS: "respond
conversationally and briefly", "a concise, speakable final answer", and
"if an utterance is unclear, inaudible, or minimal, ask the speaker to
repeat it". Both ride every turn, so the first fix was half a fix —
chattiness 100 / verbosity 77 was still being overruled, and the model
was still being told to answer a poor transcript with "say that again".

What stays is structural and safety: end every turn with a speakable
answer, never finish silently or return empty, and use only the Reachy
MCP tools. Handling a poor transcript stays with the pipeline's own
empty-transcript guard.

Also records this phase's evidence in docs/progress.md, including the
one Codex finding accepted in writing rather than fixed: SessionConfigWire
still defaults the three VAD fields, so an omitted payload selects 400 ms
against the canonical 700 ms. Requiring them breaks ~60 bridge tests that
send an empty session_config, and every real client sends the configured
values, so it is tracked separately rather than blocking this deploy.

* Stop the robot's own speaking gesture from suppressing barge-in

BargeInDetector suppresses interruption while the chassis is
mechanically noisy, since the mic hears servo noise as speech. It
counted every MotionStarted/MotionCompleted with no filter — but
idle:breathing runs whenever the robot is still and speech:presence runs
for the entire duration of every reply, so the guard was pinned on
during exactly the window barge-in exists to serve. UserInterrupted is
the only promoter of a speech boundary deferred during playback, so the
overlapping utterance was returned as playback_overlap and discarded
with no STT and no error.

MotionActivity now carries acoustically_quiet, propagated on both motion
events, with the two continuous ambient activities setting it. The
motion package owns the classification rather than the audio package
matching on activity names. The exemption applies on both edges so the
count cannot unbalance — there is a test for that specifically.

Confirmed the regression test earns its place by reverting the fix and
watching it fail.

* Make the barge-in subscription lossless

Codex: the detector kept the default DROP_OLDEST policy. Its motion
guard is a counter reconstructed from paired MotionStarted/
MotionCompleted events, so a dropped completion strands the count above
zero and suppresses barge-in for the rest of the session — silently, and
by exactly the mechanism the previous commit just fixed. A dropped start
weakens self-interruption protection instead.

BLOCK is the bus's sanctioned policy for a consumer that cannot miss
events, and this is one: correctness here depends on the pairing, not on
the latest value. Publishers reach the bus through publish_nowait, which
hands off to a task, so back-pressure lands there rather than on the
motion scheduler's own loop. This is the first BLOCK subscriber in the
codebase; the detector's handlers are all field updates and task
cancellations, so it drains without stalling a publisher.

---------

Co-authored-by: Quality Test <quality@example.invalid>
Co-authored-by: Claude Fable 5 <noreply@anthropic.com>
Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
