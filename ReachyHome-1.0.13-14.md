# Reachy Home 1.0.13

Fix Claude Max sign-in end-to-end (engine bricking, CLI PATH, authorization-code entry) (#113)

* Fix Claude Max sign-in bricking when another engine is active

Selecting any non-Claude-Max conversation engine ran the bridge reconcile,
which terminally close()d the claude_max engine. ClaudeAgentSdkChatEngine.close()
latches _closed=True with no reopen path, so every subsequent
/v1/claude-max/login (and /account, /models, /logout) raised
"Claude Max chat engine is closed" -> HTTP 503 until the bridge process
restarted. This also created a deadlock: selecting Claude Max requires an
already-connected account, but you could not sign in to become connected.

- Add ClaudeAgentSdkChatEngine.release(): frees conversation-scoped SDK sessions
  without setting _closed, leaving sign-in/account/models/health usable
  regardless of the active engine. The on-disk OAuth session and any in-flight
  login are untouched. close() stays the terminal teardown for app shutdown.
- Bridge reconcile releases (not closes) claude_max via a new _release_engine
  helper that prefers release() and falls back to close().
- probe_claude_max_voice_bridge no longer hard-503s the whole readiness probe
  when the (non-kokoro) TTS prepare call fails: _prepare_local_tts now degrades
  the tts sub-status instead of propagating the 503 that surfaced as an opaque
  "Local voice bridge HTTP 503".

Tests: engine release() keeps the engine usable while close() disables it;
bridge reconcile away from claude_max keeps POST /v1/claude-max/login at 200.
Full unit+contract suite and strict mypy green (2864 passed).

Note: the Codex/ChatGPT app-server engine has the same terminal-close-on-reconcile
hazard and is tracked as a follow-up (its subprocess lifecycle needs a dedicated
non-terminal release).

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

* Resolve the Claude CLI against standard tool dirs (launchd-minimal PATH)

The Claude Max engine resolved a bare "claude" via shutil.which on the inherited
PATH. A GUI/launchd-spawned local-voice bridge inherits a minimal PATH
(/usr/bin:/bin) that omits Homebrew (/opt/homebrew/bin) and per-user install
dirs, so account()/start_login() raised "Claude CLI is not installed on this
Mac" even though Claude Code was installed — blocking Claude Max sign-in. Same
class as the app subprocess PATH bug fixed in PR #110.

- _resolve_executable falls back to a PATH augmented with the standard tool dirs
  (plus ~/.local/bin and ~/.claude/local) when the inherited PATH hides the CLI.
- _cli_env gives the CLI child that augmented PATH so it can find its own
  helpers regardless of the launchd PATH.

Tests cover the augmented-PATH fallback and the child env PATH.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

* Accept the Claude Max authorization code (finish --no-browser sign-in)

The `claude auth login --no-browser` flow prints a URL, the user authorizes in a
browser and receives an authorization code, and the CLI then reads that code
from stdin to finish the token exchange. The engine spawned the login with
stdin=DEVNULL, so the code could never be delivered — the login hung forever and
sign-in could not complete from the app.

- Spawn the login process with a real stdin pipe.
- Add ClaudeAgentSdkChatEngine.submit_login_code(login_id, code): writes the code
  to the waiting process, waits for it to finish, and returns the resulting
  account (or a typed auth error if the code is rejected / no login is pending).
- Bridge: POST /v1/claude-max/login/submit (LoginCodeSubmissionRequest).
- Brain proxy: POST /api/local-voice/claude-max/login/submit.

Tests: engine delivers the trimmed code to stdin and reports the account, rejects
a bad code (auth error), errors with no active login and on an empty code; bridge
route forwards login_id+code to the engine. Full suite + strict mypy green
(2871 passed). The macOS auth-code input field is a follow-up UI change.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

* Add Claude Max authorization-code field to the app

The Claude Max sign-in uses `claude auth login --no-browser`: it returns a URL,
the user authorizes and receives a code, and the code must be submitted to
finish. The app previously only opened the browser and polled, so sign-in could
never complete. Add the in-app code entry that calls the new
/api/local-voice/claude-max/login/submit endpoint.

- ReachyClient.submitClaudeMaxLoginCode(loginID:code:).
- SubscriptionStatusState: new .awaitingCode operation with pendingLoginID/
  pendingAuthURL; signIn() now enters .awaitingCode after opening the browser
  (the account poll stays as a tolerant backstop); submitCode() is the primary
  completion path; cancel() calls the server cancel endpoint. A narrow
  SubscriptionLoginCodeClient protocol keeps ChatGPT's browser-redirect flow
  untouched.
- VoiceSection: an awaiting-code card with the copyable/reopenable auth link, a
  code TextField, and Submit/Cancel — reusing existing styling and feedback.

make build succeeds with strict concurrency + warnings-as-errors.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>

---------

Co-authored-by: Claude Opus 4.8 <noreply@anthropic.com>
