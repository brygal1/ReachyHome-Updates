# Reachy Home 1.0.127

Keep the selected Claude subscription brain warm between turns (#301)

* Keep the selected Claude subscription brain warm between turns

Every Claude-subscription turn used to run an account check, start a whole
Claude Code process for model discovery, then start a second cold process and
wait for MCP before sending anything; live context in --system-prompt also
changed the cached prefix every turn.

- live_context.py now owns the marker, split and per-turn phrasing for every
  brain; ChatGPT's duplicate marker is gone.
- New optional PreparedConversationChatEngine lifecycle: the bridge prepares
  warm resources for the selected brain on session configure and releases
  them on close; provider switches still release via reconcile_resources.
- The subscription engine caches its verified account and model catalog,
  launches with the stable persona prompt, and keeps one pre-started,
  tool-verified client in StandbySlot, probed before use and refilled after
  each turn. Nothing runs while another brain is selected.

ADR-0080 records the decision and the measured overhead it removes.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Close a probed standby when its turn is cancelled

A barge-in during the liveness probe left the taken client owned by neither
the standby slot nor the turn, keeping its Claude Code process alive until
the provider was switched. Close it before re-raising.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

* Log when a Claude subscription standby is replaced on demand

A standby that fails its liveness probe used to fall back to a cold launch
silently, hiding a latency regression. Log the exception type only.

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>

---------

Co-authored-by: Reachy Tests <reachy-tests@example.invalid>
Co-authored-by: Claude Opus 5.5 <noreply@anthropic.com>
