# Reachy Home 1.0.131

Render the Family Book into the cached prompt and carry over recent sessions (#306)

* Put the Family Book in every brain's cached prefix (ADR-0081 PR4)

Memory renders a deterministic, salience-ranked Family Book (REACHY, each
member and pet, household, our story) that PersonalitySessionSource places
after the persona and before the live-context marker, so every brain caches
the household canon. Who is speaking and recently ended conversations move
below the marker; a mid-session identity change now sends only the speaker
line. Session assembly leaves main.py for dialogue/session_source.py, prompt
audit records the book version and rendered rows, and the history service
keeps a bounded, deletion-aware carry-over cache of recent sessions.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Keep carry-over robust and forgettable; use household time for the book

Review fixes for PR4: carry-over trims each recent session to its newest
turns within a per-session share instead of dropping it (one long session
no longer disables carry-over), every forget path clears the carry-over
cache, unregistered speakers render as "Someone", the Family Book's day and
episode months follow the household timezone, ended sessions are cached
after dreaming is signalled, and restart warm-up orders by end time.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Make every forget reach carry-over, durably, and name linked speakers

Review fixes for PR4: single and multi-row deletions also notify forget
listeners; forgetting records a durable carry-over cutoff in the
conversation database so a restart never re-warms a session that started
before it (including the session in which the forget happened); and
carry-over speaker names resolve through recognition-profile links.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Never let an in-flight carry-over read undo a forget

Recheck the forget cutoff after reading a transcript and before caching
it, and never move a newer in-process cutoff back during warm-up.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Carry recall usage into the next day's Family Book

Recalled rows update the cached book inputs in place, so the next daily
rebuild ranks by real use while today's book stays byte-stable for
prompt caching.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Fail a forget that cannot durably clear carry-over

Every forget listener still runs, but any failure now fails the forget
so the caller sees an error and can retry, instead of a success that a
restart would undo.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Retry an unsaved carry-over cutoff until it is durable

A retried delete can find nothing left to forget, so the conversation
service keeps an unsaved cutoff pending and saves it before the next
conversation starts, during warm-up, and on stop.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Carry over only what REACHY actually said

Apply each interruption correction to the assistant turn it amends
before taking the carry-over tail, so an unspoken remainder never
reaches the next conversation.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
