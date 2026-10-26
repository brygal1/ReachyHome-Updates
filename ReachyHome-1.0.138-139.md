# Reachy Home 1.0.138

Record every conversation end, and make ending a conversation going to sleep (#314)

* Record every conversation end, and make ending a conversation going to sleep

A dialogue session now announces ConversationEnded exactly once per
provider session, however it ended: sleep, a Reachy Home close, Body
grant loss, reconfiguration, reconnect, or a provider end. Normal
closes never published it before, so history, carry-over and memory
dreaming saw no conversation end on the local voice path.

History ends sessions a restart left open at their last turn, and the
dream backlog dreams them oldest first, skipping silent sessions.
Reachy Home's separate End Conversation is replaced by Sleep, and the
unused supervisor.sleep_after_idle_minutes setting is removed.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Count turns per provider session and let a stop own its end

A reconnect no longer carries earlier turns into the next provider
session's end, so an empty session is not dreamed or carried over, and
a provider end that arrives while a stop is in progress is left for the
stop to announce, so the gate never reacts to a close it caused.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
