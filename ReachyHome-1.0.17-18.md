# Reachy Home 1.0.17

Add an inline Save on the engine card + consistent reasoning row (#137)

The engine card showed "Changes not saved" with no reachable Save — the only
"Save and Use" lived nine cards down inside "Try the Voice", so switching the
conversation engine looked like it did nothing (reported on Claude Max). Add a
"Save & Use This Engine" button right under the engine picker for the Mac-backed
engines (local/chatgpt/claude/openai/claude_max), calling the same
saveMacBackedSettings that restarts the active conversation with the new brain's
speech settings. ElevenLabs/HF/Pollen keep their own cards' save.

Also standardize the reasoning affordance: the Claude (API) and OpenAI cards now
show a "Reasoning · Model default" row when a model exposes no effort control,
matching ChatGPT/Claude Max, instead of silently hiding it (Phase A2).

make build + make test green (33 tests).

Co-authored-by: Claude Opus 4.8 <noreply@anthropic.com>
