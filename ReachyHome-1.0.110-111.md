# Reachy Home 1.0.110

Fix Mac service recovery and adopt Xcode 27 (#282)

* Stop treating a powered-off Reachy as a Mac-service failure.

Reachy Home stacked Repair Mac Services on top of Reachy is unavailable
because remote gateway health mixed the robot media canary into Mac-stack
readiness. Mac services now stay ready when launchd, Docker Desktop LiveKit,
Serve, and the loopback gateway process are healthy. Repair reclaims stray
Reachy-labeled LiveKit containers on other Docker engines and refuses
unlabeled occupants of the loopback media ports.

* Bump locked anyio to 4.15.1 for pip-audit.

Transitive anyio 4.14.1 failed local quality and pre-push with CVE-2026-63374,
CVE-2026-64847, and CVE-2026-63349. Lock the patched 4.15.1 release.

* Harden Mac service repair and adopt Xcode 27

* Create supervised child groups atomically on macOS 27

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
