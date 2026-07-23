# Reachy Home 1.0.9

Auto-release on main + household iPhone install buttons (#108)

* Auto-release Sparkle builds on green main and add iPhone install buttons.

Cut the next Reachy Home release after CI succeeds on main, and let Reachy Home sideload Reachy Remote onto Bart and Holly’s phones from the latest v* tag without App Store distribution.

Co-authored-by: Cursor <cursoragent@cursor.com>

* Allow ChatGPT account unreadiness through the software update gate.

Signed-out was already exempted; not-ready and similar backend_ready=false account states still blocked deploys even when Mac bridge transport was healthy.

Co-authored-by: Cursor <cursoragent@cursor.com>

* Ignore editable egg-info and __pycache__ in post-activation robot readback.

After pip install -e and app startup those runtime paths are expected on the robot; treating them as sync drift made forced updates fail closed even when source and bytecode attestation matched.

Co-authored-by: Cursor <cursoragent@cursor.com>

* Harden update regressions for voice unreadiness and robot readback.

Add a voice-account gate matrix plus end-to-end update coverage so ChatGPT not-ready cannot block deploys again, and lock post-activation rsync excludes for egg-info/__pycache__ into ADR-0042 and the suite.

Co-authored-by: Cursor <cursoragent@cursor.com>

* Import Mapping from collections.abc for ruff UP035.

Co-authored-by: Cursor <cursoragent@cursor.com>

---------

Co-authored-by: Cursor <cursoragent@cursor.com>
