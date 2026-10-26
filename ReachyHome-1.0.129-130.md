# Reachy Home 1.0.129

Decide Family Book memory and remove Honcho (ADR-0081, PR1) (#303)

* Decide Family Book memory and plan its build (ADR-0081)

ADR-0081 replaces identity-gated four-store memory with a household-wide
Family Book in the cached prompt prefix, history-preserving evolving facts,
speaker-attributed mixed-session dreaming with nightly reflection, per-turn
FTS5 recall, a memory-owned agenda, and one supervisor initiative governor so
REACHY can start conversations. Honcho/QMD shared memory is removed.

Supersedes ADR-0008 and parts of ADR-0055/0056. The phased build plan lives in
docs/designs/family-book-memory-plan.md.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Remove Honcho/QMD shared memory (ADR-0081 PR1)

Local memory is the only memory system. Deletes the shared-memory service,
LAN bridge, search_shared_memory tool, settings and health routes, the
memory-bridge launch agent and its credentials, config, Reachy Home cards,
and the related tests and docs. The update flow's superseded-agent
reconciliation boots out the old launch agent on the next update; the Mac's
own Honcho/QMD installs are untouched.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Tighten ratchet baselines after removing Honcho; note post-deploy tool sync

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
