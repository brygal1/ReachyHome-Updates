# Reachy Home 1.0.15

Add a "Test Reply" verification flow to the Claude Max card (#120)

Claude Max could sit stuck at "no spoken reply has been verified yet" because —
unlike ChatGPT — it had no button to trigger a verified spoken reply. Functional
readiness only flips to "ready" on a proven PlayableTurnCompleted (a real reply
through the room speaker) or a user-confirmed test reply, and that UI existed
only for ChatGPT.

- Generalize the ChatGPT-only testChatGPTReply/confirmChatGPTReply into
  engine-agnostic testEngineReply(providerLabel:)/confirmEngineReply(heard:),
  with thin ChatGPT and Claude Max wrappers. Same ReachyClient calls
  (startConversation, diagnoseAgentResponse, confirmAgentResponse) — no new
  endpoints; the heard:true confirm publishes the same user_confirmed proof.
- Add "Test Reply on Reachy" + "I heard it"/"I did not hear it" to the Claude
  Max card, reusing the shared testReplyFeedback / pendingAudibleDiagnosticID /
  isConfirmingAudibleResponse state and the existing gating/styling.

make build succeeds (strict concurrency + warnings-as-errors).

Co-authored-by: Claude Opus 4.8 <noreply@anthropic.com>
