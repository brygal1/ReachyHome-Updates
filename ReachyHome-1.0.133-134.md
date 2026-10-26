# Reachy Home 1.0.133

Bring up due threads and greet arriving members (ADR-0081 PR7) (#309)

* Bring up due threads and greet arriving members (ADR-0081 PR7)

The agenda is open threads: due, current threads about whoever is present
appear below the live-context marker as "Worth bringing up", audited per
item, and nightly reflection may now write dates, curiosity gaps, and
callbacks as threads. A new InitiativeGovernor is the one owner of when
REACHY may speak unprompted (quiet hours, occupancy, waking, active
conversation, per-member daily caps, cooldown); reminder delivery now asks
it, and an ArrivalGreeter uses it to open a conversation when a member
returns after an absence with something due.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Greet only true arrivals, into conversations REACHY opened

Review fixes for PR7: arrival is measured from when the member was last
present (any identified face track keeps them present), so turning away and
back is never a greeting; the greeter only speaks into a conversation it
opened itself; Clear All also empties the cached agenda; and the configured
quiet hours apply even when reminders are disabled.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Resolve dream speakers through the one member lookup

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Cover the arrival greeter's event loop

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* End a member's presence when their face track leaves view

Tracking reports PersonLeftView without an identity, so the greeter
resolves the departed track from its own track-to-member map.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Bring up a member's due threads when they are recognized mid-session

A member recognized only after the conversation started now gets their
own due threads with the identity update; household and REACHY threads
were already in the opening prompt.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Pick a member's own threads before the agenda limit, and date reflection

A mid-session identity update selects only the member's threads before
the agenda limit, so shared threads cannot crowd them out, and nightly
reflection is told the household date before scheduling agenda threads.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
