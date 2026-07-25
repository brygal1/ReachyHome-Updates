# Reachy Home 1.0.24

Fix the one-turn reply lag on Claude Max (#144)

* Stop an abandoned Claude Max turn from being spoken on the next turn

The Agent SDK gives a session ONE message stream. A turn abandoned before its
terminal ResultMessage left that message, plus any buffered tail, queued; the
next turn's fresh receive_response() read it first, broke at the stale
ResultMessage, and left its own reply for the turn after. The offset was
self-sustaining and permanent — the one-turn reply lag heard in conversation.

turn_repair.py existed to prevent this but could not work as written:

* it re-entered the ABANDONED receiver. receive_response() is an async
  generator; cancelling its in-flight read closes it (later reads raise
  StopAsyncIteration) and re-entering one still running raises RuntimeError.
  Either way the drain consumed nothing while the residue stayed queued. It now
  drains a FRESH receiver over the same stream, which is where the residue is.
* it hung off `except asyncio.CancelledError`, so it never ran for the widest
  case: abandoning the generator parked at a yield finalizes it with
  GeneratorExit, which that clause does not catch, and no interrupt was sent at
  all. The repair now runs from the `finally`, which every non-terminal exit
  reaches — barge-in cancellation, GeneratorExit, the turn budget, SDK errors.
* a blanket `suppress(Exception)` hid all of it. A failed repair now logs.

Also close the turn deterministically (contextlib.aclosing at both engine call
sites and in the bridge consumer) so cleanup finishes inside the lock the turn
holds instead of at GC time, where it competed for the NEXT turn's stream; widen
ChatEngine.chat to AsyncGenerator so "closable" is a type-level guarantee; and
lower DRAIN_TIMEOUT_S to 2s, since the repair now sits on the barge-in path.

Why CI never caught it: every receiver double was a class with `async def
__anext__`, which is re-enterable and survives abandonment — the doubles
diverged from the SDK in exactly the load-bearing way, and
test_task_cancellation_interrupts_and_drains_the_abandoned_turn passed against
a repair that drained nothing. tests/.../fake_session.py replaces them with real
async generators over one shared queue, and the four new tests in
test_turn_ordering.py assert the SYMPTOM (the next turn speaks its own reply).
All four fail on the previous code.

uv run ruff format --check . && uv run ruff check .  -> clean
uv run mypy                                          -> 464 files, no issues
uv run pytest tests/unit tests/contract -q           -> 3144 passed, 1 xfailed
uv run pytest tests/simulation -q -m sim             -> 14 passed
python scripts/check_loc.py --check                  -> OK (engine.py 1293, was 1297)

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

* Record the turn-ordering root cause and the verified serialization constraint

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

* Drain and interrupt concurrently, or the repair deadlocks on a full SDK buffer

A judge panel flagged a hazard in the previous commit and it checks out against
the installed claude_agent_sdk 0.2.120:

* Query._read_messages routes `control_response` AND does
  `await self._message_send.send(message)` in the SAME loop (query.py:247+).
* That stream is created with max_buffer_size=100 (query.py:121), and the engine
  sets include_partial_messages, so there is roughly one message per token.
* interrupt() is a control request with a 60 s default timeout
  (_send_control_request, query.py:502).

So an abandoned reply fills the buffer, the read loop blocks on send(), and the
interrupt's control response can never be routed. The previous
interrupt-then-drain order therefore waited out the SDK's 60 s timeout inside the
`finally` that cancel_turn() awaits — Reachy silent for up to a minute, most
likely in exactly the GeneratorExit case the previous commit newly covered,
because a consumer that stopped reading is what fills the buffer.

Measured against the faithful double (capacity 8, 12-token reply abandoned after
one chunk):

    interrupt-then-drain   -> DEADLOCKED (>5.00s), residue=11
    concurrent             -> completed in 0.02s, residue=0

The repair now runs the drain and the interrupt concurrently under one
DRAIN_TIMEOUT_S bound, and cancels whichever half is left: the drain keeps the
read loop moving so the interrupt can be answered, and an interrupt that cannot
be answered no longer extends the barge-in path.
test_repair_does_not_stall_when_the_sdk_buffer_is_full asserts the whole repair
finishes in under a second.

Removes test_stream_turn_barge_in_cancels_reads_then_drains and
test_task_cancellation_interrupts_and_drains_the_abandoned_turn. Both asserted
the now-disproven invariant "await the in-flight read, then RE-ENTER the
receiver", using doubles that return the same receiver object from
receive_response() — which no real client does. Keeping them would have required
preserving the deadlocking order. Their coverage lives in test_turn_ordering.py
against real async generators.

uv run ruff format --check . && uv run ruff check .  -> clean
uv run mypy                                          -> 464 files, no issues
uv run pytest tests/unit tests/contract -q           -> 3143 passed, 1 xfailed
uv run pytest tests/simulation -q -m sim             -> 14 passed
python scripts/check_loc.py --check                  -> OK

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

* Audit the prompt as sent, and keep an interrupted exchange in memory

Two defects the turn-ordering audit surfaced in conversation_chat.py.

1. The prompt audit reassembled its message list AFTER the reply was appended to
   turn_messages, so every audited turn showed the model being fed its own
   answer, contradicting ADR-0046's "exact assembled message list". It now
   records the list actually sent, captured where it is built.

2. History was committed as the LAST statement of the turn, after all TTS, so a
   barge-in mid-reply returned CANCELLED first and dropped BOTH the user's
   message and the reply from `_history` — the only memory the
   ollama/claude/openai engines have. Now also committed on the cancelled path.

The audit's advice for (2) was to commit right after the agent_response send.
That is wrong, and a pre-existing test says so:
test_completed_speech_boundary_cancels_unseen_generation_before_history asserts a
reply superseded before ANY audio played must NOT be remembered — agent_response
is sent before a single sample is voiced. The rule is therefore narrower: commit
when audio was actually voiced, which keeps both invariants.

That guard is only a partial fix and the remaining gap is recorded as a STRICT
xfail rather than left implicit — test_barge_in_after_audio_started_keeps_the_
exchange_in_history reproduces it end to end. A barge-in mid-audio cancels the
turn task, so CancelledError is raised OUT of _finish_speech and the
samples-voiced check below it never runs. Closing it needs a voiced-frames signal
that survives the exception; a check after the await cannot see it.

uv run ruff format --check . && uv run ruff check .  -> clean
uv run mypy                                          -> 464 files, no issues
uv run pytest tests/unit tests/contract -q           -> 3143 passed, 2 xfailed
uv run pytest tests/simulation -q -m sim             -> 14 passed
python scripts/check_loc.py --check                  -> OK

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

* Keep main's LOC baseline for operations/cli.py

`check_loc.py --tighten` was run while the work sat on claude/one-button-update,
where operations/cli.py is 7378 lines, so it lowered that baseline to 7378. The
commits were then cherry-picked onto a branch off main, where the file is 7381 —
so the ratchet read a 3-line growth in a file this PR never touches, and CI's
file-size gate failed while the local check (run on the old branch) had passed.

Restores main's value. The only baseline change this PR now makes is the
claude_max/engine.py tightening it actually earned, 1297 -> 1293.

python scripts/check_loc.py --check  -> OK, 317 files scanned

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

* Address review: stop the repair escaping, short-circuit it, drop dead code

Both mandated reviewers (code-reviewer, silent-failure-hunter) found real defects
in the previous commits. Fixes, most severe first.

1. A failing repair escaped the turn and tore down the conversation.
   `await asyncio.shield(inner)` re-raises the inner task's exception to the
   awaiter, so a repair TimeoutError left `_stream_turn`'s finally and REPLACED the
   barge-in's CancelledError. `cancel_turn` suppresses only CancelledError, and the
   bridge's WebSocket dispatch handles neither TimeoutError nor OSError, so a
   barge-in against a slow CLI unwound the endpoint. The "never silent" warning was
   unreachable dead code on exactly the paths that mattered. repair_interrupted now
   never raises, logs at error with conversation id and elapsed time, and
   distinguishes "drained a terminal" from "gave up" from "stream ended".

2. A second cancellation at `await receiver.aclose()` skipped the repair entirely.
   suppress(Exception) does not cover CancelledError, so the finally aborted before
   the one call it exists to guarantee. The cleanup awaits are now inside their own
   try/finally, and a receiver that will not close logs — an un-finalized receiver
   stays a live consumer of the shared stream, and the repair then opens a second.

3. `terminated` was not set when asyncio.wait reported both futures done. The
   cancel branch discarded a completed read that may have held this turn's
   ResultMessage, so the repair interrupted a turn the CLI had already finished and
   burned the whole drain budget on an empty stream, inside session.lock, logging
   the wrong cause. Now inspected via _read_settled_the_turn.

4. The repair always waited for the interrupt's control-response round-trip.
   asyncio.gather waits for both halves, so the comment about a completed drain
   making the interrupt moot described behaviour the code could not produce. Now
   FIRST_COMPLETED, and a drain holding the terminal ends the repair.

5. A refused interrupt was swallowed unlogged, so the drain's subsequent timeout
   was reported as "repair failed" — naming the symptom and hiding the cause. Same
   pattern this branch blames for the original bug. Now logged.

6. The cleanup awaits in repair() sat OUTSIDE the drain timeout, so either half
   that would not die made the repair, the turn, and the session lock hang
   unboundedly. Bounded by CLEANUP_TIMEOUT_S, and a half that refuses logs.

7. Dropped the samples-voiced history commit. It was live but reachable only by a
   timing accident (SegmentSpeaker.finish swallowing the cancellation when its
   queue happened to be full), so whether Reachy remembered an interrupted exchange
   depended on TTS queue timing. Nondeterministic memory is worse than a documented
   limitation, and CLAUDE.md rules out placeholders declared done. G3 is now written
   up in docs/known-limitations.md with the two constraints that make the real fix
   narrower than it looks, still tracked by the strict xfail.

Also restores the "no orphaned reader task" assertion lost with the deleted tests,
and corrects the docs/progress.md evidence (wrong commit hash, test count, and
"four" where there are five).

Known follow-ups, recorded not fixed: cancelling the interrupt leaves two entries
in the SDK's pending_control_* dicts per timed-out repair; the Codex engine's turn
cleanup still hangs off `except asyncio.CancelledError` (app_server.py:1115), the
same defect this fixes for claude_max, now more reachable via aclosing.

uv run ruff format --check . && uv run ruff check .  -> clean
uv run mypy                                          -> 463 files, no issues
uv run pytest tests/unit tests/contract -q           -> 3142 passed, 2 xfailed
uv run pytest tests/simulation -q -m sim             -> 14 passed
python scripts/check_loc.py --check                  -> OK

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

* Split the model catalog out of the Claude Max engine

engine.py collided with the ratcheting file cap three times during this work, the
last time by +37 lines from the review fixes. Trimming comments to fit was the
wrong answer twice over: the comments are why the next reader will not reintroduce
these bugs, and a 1330-line file with a 750-line cap needs splitting, not shaving.

model_catalog.py takes the catalog axis: the curated default model list, the
per-family reasoning-effort curation, and the translation of the Agent SDK's server
info. None of it touched session state — both methods were already effectively
static — so they move as free functions. The engine keeps sessions, turns and the
CLI process.

engine.py 1330 -> 1242, and the baseline tightens to match.

uv run ruff format --check . && uv run ruff check .  -> clean
uv run mypy                                          -> 464 files, no issues
uv run pytest tests/unit tests/contract -q           -> 3142 passed, 2 xfailed
uv run pytest tests/simulation -q -m sim             -> 14 passed
python scripts/check_loc.py --check                  -> OK, 318 files

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

---------

Co-authored-by: Claude Opus 5 <noreply@anthropic.com>
