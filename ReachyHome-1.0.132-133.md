# Reachy Home 1.0.132

Dream from attributed evidence and reflect nightly (ADR-0081 PR5) (#307)

* Dream from attributed evidence and reflect nightly (ADR-0081 PR5)

Session dreams read per-turn speakers: a memory must cite a transcript turn
reliably said by a household member, so joint conversations are eligible and
unknown or guessed speakers alone teach nothing. The contract adds evidence
citations, quoted corrections for pinned facts, confirmations, social
threads with a follow-up day, thread closure, a session summary that replaces
the raw carry-over tail, and conversation signals. Invalid memories are
dropped individually instead of failing the run, and every member a dream
writes about is fenced against later forgets. A nightly reflection, owned by
the dream service and idempotent per household day, refreshes portraits,
merges duplicates, hardens episodes, and re-weighs importance; POST
/api/memory/reflect runs one on demand.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Harden dream attribution, corrections, reflection, and forgetting

Review fixes for PR5: the legacy speaker fallback applies only to sessions
recorded before per-turn attribution existed, so a face-only guess never
teaches memory; a pinned correction needs a substantive quote on word
boundaries that relates to the new memory; only enabled speakers support a
memory; reflection fences its writes from the time it read memory; session
summaries are never persisted outside the deletion-aware carry-over cache;
the Memory Brain response is parsed per item with every limit stated; runs
record how many memories were dropped; recovery re-queues only session
dreams; and the nightly loop skips busy nights, releases on preemption,
sleeps against the wall clock, and builds its prompt in linear time.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Resolve linked speakers and drop a bad memory on its own

A recognition id now counts as its linked member when a dream restricts
evidence to members, and a memory that fails policy is dropped without
failing the rest of the run. Covers the nightly reflection scheduler.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
