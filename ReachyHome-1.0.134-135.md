# Reachy Home 1.0.134

Show the Family Book and household members in Reachy Home (ADR-0081 PR8) (#310)

* Expose the Family Book and open threads to Reachy Home

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Let the household pin an existing fact

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Show the Family Book and members in Reachy Home

Adds Household Members and Family Book cards, pin and unpin on memory
rows, dream-run kind, dropped count and signals, and prompt-audit
injections grouped by layer (ADR-0081 PR8).

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Keep Reachy Home's memory views consistent under concurrent actions

The service owns the open-thread projection, a second Reflect Now is
refused rather than queued, stale Family Book loads are ignored, and
dream runs reload after a reflection.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Hide expired threads and require the full dream-run payload

open_threads drops a thread that expired after the cache loaded, and
Reachy Home treats a dream run without kind, dropped_count or signals as
a malformed payload instead of guessing it was a session.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
