# Reachy Home 1.0.21

Stop Reachy speaking tool-call markup and forgetting the conversation (#141)

Two production bugs, diagnosed from the Claude Code CLI transcripts and the
bridge logs (78 sessions in one afternoon, 42 of them single-turn).

Barge-in destroyed the conversation. `_run_agent_turn` discarded any turn that
did not commit, and an interrupted turn never commits. That was written for the
ChatGPT engine, which is re-sent the bridge's full transcript each turn and
rebuilds its thread; Claude Max is sent only the latest user message and keeps
all history CLI-side, so the same call erased everything. Hence "I don't have
record of an earlier conversation here" one turn after a ten-minute chat.

Turn outcome is now a tri-state, `CancelledError` is classified as an
interruption rather than a failure, and the new `TurnLifecycleChatEngine`
protocol lets the engine that owns the conversation repair itself instead of
the bridge picking a repair whose cost depends on an ownership model only the
engine knows. Engines that do not implement it keep the existing behaviour.

That exposed a latent bug: `cancel_turn()` cancels the turn task with no await
after setting the event, so `CancelledError` escapes `asyncio.wait` and the
in-loop interrupt branch never ran. Destroying the session had been masking it.
`_stream_turn` now runs a shielded, bounded interrupt+drain on cancellation and
cancels its pending receiver read, so the abandoned turn cannot replay onto the
next one.

A fresh session's first turn ran with no tools. `connect()` returns before the
CLI has handshaked the Reachy MCP server, so the first turn raced `tools/list`
and won. Given a prompt naming tools it could not see, the model serialized the
call into prose and TTS read the raw `<function_calls>` block aloud. Session
open now blocks until the server is connected with a non-empty tool list, and a
reply carrying tool-call markup fails the turn instead of being voiced.

Co-authored-by: Claude Opus 5 <noreply@anthropic.com>
