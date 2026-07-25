# Reachy Home 1.0.23

Cut spoken-turn latency across every conversation provider (#143)

Spoken conversations felt awkward, starting from the Claude Max path. Researched
first (docs/latency-research.md), then fixed; the two largest findings were
cross-provider rather than Claude-specific.

Two root causes. TTS waited for the *entire* completion before speaking, so
first-audio was TTFT plus the whole generation. And spoken turns ran at an agentic
reasoning tier: Claude Max curated Opus to "medium" effort, which crossed the
engine's own `!= "low"` branch and enabled extended thinking on every reply, while
an unset effort let the CLI pick `xhigh`. The two Claude engines had also drifted
into opposite rules for the same setting.

Shipped on by default:

- voice/thinking_policy.py — one home for the effort/thinking rules, shared by both
  Claude engines. Always state an effort, defaulting to the voice floor. Keep
  thinking adaptive rather than disabled: disabling is a trap on current Opus
  models, which then intermittently write a tool call into user-facing text instead
  of emitting a tool-use block — the exact failure voice/tool_markup.py exists to
  catch. Opus no longer defaults to "medium"; Haiku is sent neither control; the
  Messages-API engine leaves token headroom for the thinking pass; the OpenAI
  picker no longer preselects "medium".
- Claude prompt caching with a frozen prefix — cache_control on the persona block,
  with per-turn live context split out (it would otherwise sit in front of the
  cached prefix and miss every turn) and carried as a trailing user turn. Usage is
  now reported at all, including the cache split, so hits are verifiable.
- dialogue/latency_stats.py — per-stage medians and p95 per provider, on
  GET /api/conversation as `stage_latency`. Only completed turns count; a sample
  spanning more than one config_fingerprint is flagged as mixed_config.
- Recognizer warmed at session config instead of on the user's first utterance.
- A runaway reply truncates instead of failing a turn already part-spoken.

Built but shipped OFF: sentence-level streaming synthesis
(`tts_streaming_enabled=false`). Review established that a reply whose first
sentence is under the release floor, or which is a single sentence, gains nothing —
so on a terse persona the win may be small, and this is the change here with the
largest risk footprint. Off is a genuine bypass (no synthesis task; one utterance
after the stream closes), which makes it both the escape hatch and the measurement
baseline: a baseline taken with streaming already on has nothing to compare against.

Review found and fixed a critical defect before merge: synthesizing inside the
provider's stream iterator charged TTS time to the engine's own deadline, and when
it fired inside the suspended generator a bare CancelledError surfaced, was
classified as a barge-in, and the turn hung with no error and one sentence spoken.
Synthesis now runs on an owned task behind a bounded queue. Also fixed: skip_turn
could speak, markup detection was a silent no-op when the round also had a real tool
call, spoken preambles never reached the transcript, meets_targets compared total
turn time against a first-audio target, the token-headroom fix was a no-op in the
shipping config, xhigh/max could still be requested, and truncation lost the whole
reply on single-chunk engines.

Deliberately not done, recorded in docs/latency-research.md §5: the Claude Max
output ceiling (no such option exists in claude-agent-sdk), the 500ms VAD hangover
(Silero's own default; its cost is a constant while its benefit is uninstrumented,
and a truncated turn is worse than 500ms), chat-engine warming (needs a real
inference, so it would spend quota and could dispatch tools), and fast mode plus the
cross-provider harness, both pending measurement.

Not yet measured on hardware — that is the next step, and the telemetry above exists
to make it possible.
