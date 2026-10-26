# Reachy Home 1.0.136

Withdraw an early speaker match the full-window result rejects (#312)

* Withdraw an early speaker match the full-window result rejects

An accepted speaker prefix commits a provisional per-utterance match and the
retained identity. When the authoritative full-window result then rejected,
nothing withdrew it, so the transcript turn stayed attributed to the prefix's
person (and flowed into Family Book memory) and the retained speaker kept it.
Since #311 lets decisive prefixes accept at live scores, this happened in 2 of
the 6 sampled turns with an accepted prefix, one a near tie.

A final rejection for a turn whose prefix was applied now publishes
UtteranceSpeakerIdentityRevoked (reason "final_rejected") and rolls the
retained identity back to its pre-turn snapshot, through the same paths an
aborted turn already uses. A final not run (for example a capture boundary)
leaves the prefix alone.

The repeated active-turn reset becomes one helper so service.py stays within
its line ratchet, with tests for the robot-speech and capture-boundary paths
it covers.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Clear pending speaker state when the final withdraws a prefix

An accepted prefix followed by a missed later prefix sets the pending flag
again. The final_rejected path returned before the normal pending reset, so
a finished turn left Reachy Home showing "Listening for speaker..." until
the next speech boundary. Clear it in that path and cover the accepted,
missed, then rejected sequence. Qualify the ADR: the withdrawal corrects a
transcript turn only when it lands before the turn is persisted.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
