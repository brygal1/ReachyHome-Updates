# Reachy Home 1.0.117

Restore signed speech and recover conversation startup (#290)

* Fix SmartTurn CPU inference in the signed speech runtime

* Recover conversation startup and verify Body readiness truthfully

* Reap Codex readers when initialization exits early

* Preserve terminal turn failures while reconnecting Codex

* Bound blocked Codex request writes with the request deadline

* Bind turn notifications to their initialized Codex process

* Ignore late transport failures from replaced Codex processes

* Preserve unstamped simulator readiness without bypassing physical Brain checks

* Keep physical recovery fail-closed and bind conversations to process generations

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
