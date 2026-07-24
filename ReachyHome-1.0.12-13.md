# Reachy Home 1.0.12

Fix conversation engine switching: surface real failure reasons, stop silent revert to ElevenLabs (#112)

* Add voice-config permutation coverage across all 8 conversation backends

Parametrized credential-free tests for the full engine × reasoning ×
ollama-thinking × STT × TTS × service-mapping matrix:

- Layer 1 (test_backend_settings.py): round-trip every ConversationBackend,
  reject out-of-catalog local/pollen voices, ollama num_predict thinking
  boundary, HF/pollen endpoint scheme rules, and a 720-case matrix asserting
  only typed ValidationError/ValueError ever escape.
- Layer 3 (test_cli.py): voice_services_reconcile matches the engine→service
  matrix for all 8 providers (marker/plist/waits), idempotent no-churn once
  state matches, and _mac_service_diagnosis mismatch detection. Reverse
  provider-drift blind spot captured as an xfail (G2).
- Layer 3c (test_local_voice.py): reasoning-effort rejection across
  claude/openai/claude_max and STT engine-switch echo.

975 passed, 1 xfailed. No production code touched.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

* Surface real reason for conversation-start failures; stop silent revert on engine switch

Switching the conversation engine to a backend that could not become ready
silently rolled back to the previous engine (ElevenLabs) and reported a generic
"could not become ready to talk", leaving callers on a backend they never chose
with no idea why. Root cause: every provider-specific failure (bad key, bridge
5xx, signed-out account, cold STT/TTS) collapsed into a bare False.

- DialogueSession now captures the concrete provider readiness failure from the
  connect exception and from a fatal pre-start ProviderError, exposed via
  `readiness_failure` and cleared once SessionStarted confirms the session.
- ConversationGate records `last_open_failure` from the session on a not-ready
  open, exposes it as a property and in `lifecycle_status`, and clears it on a
  successful open.
- /start_conversation includes the concrete reason in its 409 detail.
- The backend-switch PUT keeps the user's selection persisted on failure (no
  silent revert to the previous engine) and reports the provider-specific
  reason, telling the user their choice was kept and to press Start once fixed.
  Removes the settings/session rollback machinery.

Tests updated to assert the new coherent contract (selection kept + real reason)
instead of the old silent-revert behavior, plus gate coverage for reason
capture, propagation, and clearing. Full unit+contract suite and strict mypy green.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

* Guard that every conversation backend builds its own provider

Parametrized coverage over all eight engines asserts _build_provider resolves
each selection to exactly that provider (built.name == provider), mirroring the
real deployment where config.voice.provider stays at the elevenlabs gate value
and the persisted backend setting selects the live engine. Documents that
openai/claude_max are reachable only via the backend setting, never the static
config literal.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

---------

Co-authored-by: Claude Opus 4.8 <noreply@anthropic.com>
