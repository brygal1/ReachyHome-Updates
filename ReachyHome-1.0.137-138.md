# Reachy Home 1.0.137

Play emotions while Reachy speaks instead of before (#313)

* Play emotions while Reachy speaks instead of before

play_emotion now returns as soon as the move is granted and skips the
recorded sound, so the reply is spoken over the motion rather than after
it (about 5 s saved on a typical emotion). The tool is declared
expects_response=false through a new ToolSpec flag, so the ElevenLabs
hosted agent keeps talking instead of pausing for the result. play_dance
still waits for the dance and its music.

When the model plays an emotion or dance, the automatic conversational
cue for that reply is skipped so Reachy does not react twice; a new user
turn clears the suppression.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Publish tool requests before the provider's next event

_spawn_tool used publish_nowait, which defers delivery to a new task. When a
provider hands over a tool call and its reply back-to-back (buffered ElevenLabs
messages), the awaited AgentUtterance reached subscribers first, so the
automatic expression service cued the reply it should have skipped and left the
suppression pending for a later one. Awaiting the publish keeps bus order equal
to provider order.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
