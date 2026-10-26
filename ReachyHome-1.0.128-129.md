# Reachy Home 1.0.128

Keep a speaker turn abortable after an early prefix miss (#302)

An early (2 s) speaker-ID attempt shares its turn's abort signal. When that
prefix found no match, _release_inference forgot the signal and the turn's
rollback snapshot while the turn was still active. Every later early attempt
and the final full-utterance check were then refused with "no abort signal is
armed", retried on every frame, and the turn was left unattributed. Live robot
logs showed 436 such errors across 7 turns in 12 minutes.

Early requests now leave release to the endpoint request (or to the existing
abort/supersede/discard paths), so a missed prefix no longer disarms mute
cancellation or discards the turn's longer, more reliable check.

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
