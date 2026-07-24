# Reachy Home 1.0.20

Split the local voice bridge into cohesive sub-750-line modules (#140)

bridge.py had grown to 2753 lines and was grandfathered far above the
750-line cap, forcing a baseline bump on every voice fix. Split it by
responsibility with no behaviour change:

- bridge_wire.py       wire models, settings, exceptions, capacity gate
- conversation_base.py session state, transport, provider resolution
- conversation_chat.py tool-calling chat loop, TTS speak path, audit
- conversation.py      session_config, client-VAD input, turn lifecycle
- bridge_lifecycle.py  engine close/release, finalize, prewarm
- bridge_routes.py     HTTP route registration (_BridgeContext)
- bridge_app.py        create_local_voice_app + WebSocket route
- bridge.py            executable entry point + public facade

_Conversation is decomposed via a linear base->chat->concrete inheritance
chain so call direction stays base->derived and mypy --strict resolves
every attribute. app_from_environment and the engine constructors stay in
bridge.py so `python -m ...bridge` and the create_local_voice_app
monkeypatch tests keep working. Every module is now under the cap, so
bridge.py drops from .loc-baseline.json.

Carries the bridge behaviour from #139 (narration-only reply suppression,
higher STT/TTS timeouts, STT/TTS registry timeout knobs) into the new
modules unchanged.

Also includes two small independent fixes: isolating the remote-gateway
trust store in the shell-credential test so it no longer depends on real
repo state, and tightening the stale VoiceSection.swift LOC baseline that
#138 left at 3906 for a now-406-line file.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
