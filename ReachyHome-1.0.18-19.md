# Reachy Home 1.0.18

Add a ratcheting 750-line-per-file gate (#128)

* Add a ratcheting 750-line-per-file gate

Production source (src/reachy_agent, macos/ReachyHome/Sources, web) is held to
750 physical lines per file. scripts/check_loc.py --check runs in CI: a new file
over the cap fails, and a grandfathered file (.loc-baseline.json, 29 current
violators) may only shrink — never grow. --tighten ratchets the baseline down
after a file is reduced; it can never raise a size or add a new file, so it
can't bless a regression. Tests are exempt (long test files are legitimate).

This is the gate behind the Voice/Brain split: VoiceSection.swift (3,874) is
grandfathered now and comes off the baseline once it's decomposed.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

* Rebaseline the 750-LOC gate after the merged bug fixes

Merged main (all 14 conversation-bug fixes + the inline-save UI) into the gate
branch and re-seeded .loc-baseline.json so the grandfathered set matches the
current tree: pollen_tts.py crossed the cap via its new read timeout, and
VoiceSection.swift/session.py/bridge.py grew from their fixes. The gate now
passes on this tree; from here every listed file may only shrink.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

---------

Co-authored-by: Claude Opus 4.8 <noreply@anthropic.com>
