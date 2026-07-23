# Reachy Home 1.0.11

Fix update rollback + verify: tolerate generated artifacts, wait for slow bridges (#111)

Two bugs in `reachy-ops update`, surfaced by a real deploy whose forward step succeeded
but whose verify then rollback both failed, leaving a version skew.

1. Rollback aborted on a generated build artifact. The strict robot checksum readback
   (rsync -rlzcp --delete --dry-run --itemize-changes) had no cache excludes, so on the
   rollback path — when the robot already carries the forward deploy's freshly generated
   src/reachy_agent.egg-info from `pip install -e` — it false-failed on a
   "*deleting ...egg-info" diff and aborted the rollback. Default
   _robot_rsync_commands(allow_runtime_caches=) to True so every readback proof ignores
   regenerated .pyc/__pycache__/egg-info. The write sync still --deletes them, so stale
   bytecode can never survive; only the proof ignores them. Verified locally: without the
   excludes rsync reports the exact deletion, with them it is clean.

2. Verify false-failed a healthy deploy because the 45 s window was too short for a
   just-restarted bridge to stabilize — the local-voice bridge loads its TTS models on
   startup and was mid-restart when verify polled. Raise UPDATE_VERIFY_TIMEOUT_S to 180 s,
   well within the outer continuation budget.

Update test_robot_checksum_readback... to assert the readback (not the write sync)
excludes runtime artifacts.

Co-authored-by: Claude Opus 4.8 <noreply@anthropic.com>
