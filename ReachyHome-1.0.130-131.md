# Reachy Home 1.0.130

Stamp the attributed speaker on every user transcript turn (ADR-0081, PR2) (#304)

* Stamp the attributed speaker on every user transcript turn (ADR-0081 PR2)

DialogueSession already decides who spoke each traced utterance; it now
keeps the final decision per trace (late matches and revocations win) and
stamps speaker plus speaker_basis on UserUtterance, falling back to the
fused session identity for untraced providers. Conversation migration 0003
persists both columns, and Reachy Home labels transcript turns by person.
Pending-speaker bookkeeping moves to identity_context so session.py shrinks.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Attribute only correlated voice evidence as utterance-level speaker

Review fixes for PR2: an unmatched voice no longer inherits the retained
speaker as utterance evidence (it is session-basis at most, or unattributed),
late voice results fold through the same conflict and provisional rules,
revocations after speech end clear the attribution, and untraced phone turns
are never attributed to the person in the room. Per-utterance speaker state
moves into UtteranceSpeaker, shrinking dialogue/session.py.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
