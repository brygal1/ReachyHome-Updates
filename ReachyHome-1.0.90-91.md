# Reachy Home 1.0.90

Hard-cut maintainability Phase 1: conversation provider catalog (#251)

* Hard-cut maintainability Phase 1: conversation provider catalog

One frozen voice/catalog/ owns shipped engine IDs, labels, family, probes,
launchd units, and credential names. YAML, CLI, Memory Brain, and Reachy Home
project from it. openai_realtime and null_provider are deleted; conversation
disable is voice.conversation_enabled. Invalid providers fail closed.

Adversarial review closed fail-open holes before this PR: required probes
are always present (unmeasured is ready=false), persist gates ignore warm_up,
GET catalog is required, leftover Swift provider sets are gone, and applied
receipts must prove persistence.

* Seed the UI-test fixture with the shipped provider catalog

* Load the provider catalog before Memory Brain projects or reconciles it

* Close disabled conversation voice readiness gap

Preserve the Kokoro TTS readiness guard from the pre-rebase merge resolution. Treat a persisted Mac provider as configured metadata, but require and probe its bridge only while conversation is enabled. Cover every catalog-owned Mac bridge.

* Keep body archive fixtures free of generated files

Model the clean Git source archive consumed in production by excluding local virtualenv, build, egg-info, and Python cache artifacts from the copied fixture. This removes the xdist order dependency exposed by the rebased verification run.

* Record Phase 1 exact-head verification

* Close final provider catalog review gaps

Make explicit conversation disable authoritative in service health, remove the remaining duplicated Mac-provider membership checks, and invalidate stale Home catalog projections when authoritative readback is unavailable. Tighten the LOC baseline and cover the disabled and disconnected states.

* Keep local health on the local probe schema

* Keep Memory Brain local bridge required

* Probe Memory Brain alongside active conversation

* Require robot persistence proof for brain switches

* Preserve voice readiness on Memory probe timeout

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
