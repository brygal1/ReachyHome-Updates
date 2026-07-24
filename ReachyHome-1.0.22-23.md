# Reachy Home 1.0.22

Fix Apple release packaging and device attestation readback (#142)

* Fix iPhone device packaging rejecting its own archive

`reachy-ops ios install` failed on a developer Mac with "release archive
contains content outside the expected app", after a successful archive and a
valid signature. No device artifact was produced, so the household iPhone
install path could not be used at all.

`ditto --sequesterRsrc` hoists AppleDouble metadata into a top-level `__MACOSX`
entry (`__MACOSX/ReachyRemote.app/._CodeResources`, `._Info.plist`, and so on).
release_provenance.py requires every archive member to live under the .app, so
it rejected the archive the packager had just built. CI never hit this: a clean
runner has no metadata to sequester, which is why the identical flags pass there
and fail locally.

Dropping --sequesterRsrc alone would be worse — ditto then writes the same
AppleDouble files inside the bundle, passing the top-level check while polluting
the signed app. --norsrc --noextattr excludes them outright.

Verified: packaging exits 0 and emits the artifact; the archive's only top-level
entry is ReachyRemote.app with 0 AppleDouble members across 29 entries; and the
app extracted from that archive still passes `codesign --verify --deep --strict`
("valid on disk", "satisfies its Designated Requirement"), so removing the
metadata does not invalidate the signature.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

* Fix the macOS packager too, and the attestation readback destination

Follow-up to the iOS packaging fix in the previous commit, correcting two things
that commit got wrong or left behind.

The stated cause was wrong. __MACOSX is not a developer-Mac accident from stale
metadata: macOS stamps com.apple.provenance on essentially every file it writes,
and `xattr -cr` does not help because it comes straight back. So ditto always has
something to sequester and always emits the entry. CI escapes it only because a
GitHub runner does not stamp the attribute.

That means release_macos.sh has the identical bug, deterministically, on every
local run — including the supported --skip-notarization path. Its archive goes to
the same release_provenance.py check. Fixed with the same flags, and both scripts
are now pinned by a test so they cannot diverge again.

Separately, device attestation could never succeed. `devicectl device copy from`
opens --destination as a file, so passing the directory failed all 20 attempts
with "Cannot open destination file ...: Is a directory" and reported only the
generic "did not return a fresh release attestation". The readback now names the
file; the existing find() still locates it either way.

The stub in test_apple_release_scripts.py wrote *into* $2, matching the script's
wrong assumption, which is why the suite passed while every real phone readback
failed. It now writes to $2 as real devicectl does.

Verified: ruff, mypy strict (452 files), LOC gate clean; 3,067 passed. Reachy
Remote 1.0.21 build 22 installs and is confirmed present on both household
iPhones via `devicectl device info apps`.

Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>

---------

Co-authored-by: Claude Opus 5 <noreply@anthropic.com>
