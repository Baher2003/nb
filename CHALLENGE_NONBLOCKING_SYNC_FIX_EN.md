# Challenge Responsiveness and Non-Blocking Synchronization Fix

## Task ID
CHALLENGE-UX-NONBLOCKING-SYNC-01

## Goal
Make every challenge experience smooth and uninterrupted for children, both:
- Player-versus-player (two real students)
- Student-versus-AI robot

The current UI sometimes displays **"جاري المزامنة"** ("Syncing") and appears to pause or block the child from answering. This breaks concentration and can cause the child to lose focus or leave the challenge. Diagnose and fix the underlying synchronization/state-management problem rather than merely hiding the message.

## Required First Steps
1. Read `/home/z/my-project/worklog.md` first and append a work record when finished.
2. Read `PLATFORM_KNOWLEDGE_BASE.md` and follow its architecture, exact API contracts, security rules, and Never-Do list.
3. Inspect the current PVP and robot challenge implementations before changing code. Identify the exact state transition that causes the syncing message and whether it incorrectly disables the answer UI.
4. Inspect existing logs and reproduce the issue where possible. Do not assume the frontend message is the root cause.

## Project Constraints That Must Be Preserved
- The application uses the single client SPA catch-all route. Do not create `src/app/page.tsx`.
- Follow the existing challenge/robot views and the existing `src/hooks/use-arena-socket.ts`, `src/lib/api-client.ts`, and arena API route patterns.
- The Socket.IO gateway uses port `3003` for Socket.IO only and port `3004` for HTTP operations. Do not add HTTP handlers to port `3003`.
- Cross-port calls must follow the existing `XTransformPort` rule. Do not introduce raw localhost cross-port calls.
- Keep server authority: never send correct answers or correctness information to clients during live play.
- Preserve idempotency, sequential-answer validation, monotonic progress, match expiry, wager/points transactions, and server-side result finalization.
- For the AI robot, the final verdict must remain tied to the server-stored `robotPlanJson`; do not make the frontend authoritative.
- Keep Arabic UI copy and RTL layout. Code identifiers and comments should remain English.
- Do not introduce a second question generator or bypass the shared rules engine.

## Required Behavior

### A. Never Block Answering for a Background Sync
- A transient socket disconnect, REST polling delay, progress refresh, acknowledgement delay, or background reconciliation must not unnecessarily disable the current question's answer input.
- The child should be able to read and answer the current question immediately whenever the server considers that question answerable.
- Do not place a full-screen loading overlay over an active question merely because progress is being synchronized.
- Do not display a blocking "جاري المزامنة" state over the question or keypad during ordinary background synchronization.
- If a small status indicator is useful, keep it subtle and non-blocking, outside the answer area, and do not shift the layout when it appears.
- Do not permit duplicate submissions: while the answer for the current question is genuinely being submitted, use a per-question submission guard. Keep the question UI responsive and avoid freezing unrelated controls.

### B. Make Submission Reliable and Idempotent
- Trace the complete answer flow from keypad/button interaction through the API/socket acknowledgement and server persistence.
- Ensure each answer has a stable match/session identifier and question/progress identifier, and that retries cannot award points or advance progress twice.
- If an acknowledgement is delayed or lost, reconcile against authoritative server progress before deciding whether to retry. Do not blindly resubmit an answer in a way that can create duplicate effects.
- Distinguish a request that is still pending from a request that failed. Use bounded waiting and a recoverable state instead of leaving the child stuck indefinitely.
- Prevent stale responses from an earlier question, earlier match, or previous connection from overwriting the current state.
- Keep progress monotonic: late or out-of-order updates must never move the player backward or restore an already answered question.

### C. Robust PVP Reconnection and Fallback
- Treat Socket.IO as the fast path, not the only path. Preserve the existing REST polling fallback.
- On disconnect/reconnect, reconnecting, timeout, or missed socket event, reconcile with the authoritative match state through the existing supported API.
- Do not restart the match or reset the current question merely because the connection changes.
- If a socket event is missed, recover current question index, submitted/answered state, player progress, and match status from the server without blocking the child longer than necessary.
- Avoid excessive polling or duplicate requests. Reuse existing intervals and lifecycle cleanup where possible.
- Ensure a late event from a prior match cannot affect a newly opened match.

### D. Robust AI Robot Responsiveness
- The robot must continue to feel responsive even if robot progress polling or background updates are delayed.
- Never make the child's answer input wait for a robot-progress refresh.
- Keep the robot's planned response timing and accuracy governed by the server's `robotPlanJson` and existing AI rules.
- Preserve the current robot answer batching strategy and server snapshot anchoring. Do not make client-side predictions authoritative.
- If the client temporarily loses synchronization, continue presenting the child's current answer flow safely and reconcile the robot/progress display in the background.
- Do not let a stale robot snapshot erase a newer child answer or regress the displayed progress.

### E. Clear Recovery for Genuine Failures
- If the server definitively rejects an answer, show a short, child-friendly Arabic message and allow recovery according to the existing match rules.
- If the network is unavailable for a prolonged period and an answer cannot be confirmed, show a clear non-blocking connection notice and provide a safe retry/reconciliation path.
- Do not silently discard a child's answer.
- Do not claim an answer was accepted until the server confirms it or authoritative state proves it was accepted.
- Do not show an endless spinner, indefinite "جاري المزامنة", or a disabled keypad with no recovery action.
- Preserve surrender, expiry, finalization, and match-ended handling.

## Investigation Checklist
Inspect the actual implementation and document the root cause before editing:
- Every place that renders the Arabic "جاري المزامنة" text or equivalent syncing/loading state.
- Conditions that disable the numeric keypad, answer field, submit button, or entire challenge view.
- Socket connection and acknowledgement handlers.
- REST polling and fallback handlers.
- State updates triggered by question changes, answer submission, reconnect, and component remount.
- Race conditions between socket events, polling responses, and answer acknowledgements.
- Request cancellation, timeout handling, and cleanup when leaving or reopening a match.
- Any broad `isLoading`/`isSyncing` state that incorrectly couples background refresh with answer-input availability.

Search the repository for the exact UI text and all related state flags before changing the behavior.

## Files and Areas to Inspect
Use the current repository structure and touch only files justified by the root-cause investigation. Likely areas include:
- `src/components/views/arena/` — PVP match and robot challenge views
- `src/hooks/use-arena-socket.ts`
- `src/lib/api-client.ts`
- Relevant existing `/api/arena/match/*` and `/api/arena/robot/*` route handlers
- Existing arena state/progress utilities and tests

Do not assume these are the only files; confirm actual paths in the repository. Do not refactor unrelated UI or redesign the entire challenge page.

## Testing and Verification
Add or update regression tests for at least these cases:
1. Normal PVP answer submission does not show a blocking sync overlay.
2. Delayed socket acknowledgement does not permanently disable the keypad.
3. Socket disconnect during an active question recovers through the existing REST fallback.
4. Reconnect and duplicate events do not advance progress twice.
5. A late response from an old question/match cannot overwrite current state.
6. A delayed PVP answer response cannot regress monotonic progress.
7. Robot progress polling delay does not block the child's answer input.
8. A stale robot snapshot cannot erase a newer child answer or regress progress.
9. A definitive server rejection is shown clearly and can be recovered from.
10. Match completion, expiry, surrender, and result finalization still work correctly.
11. Duplicate answer requests do not duplicate score/point effects.
12. Leaving and reopening a challenge cleans up old subscriptions and timers.

Run the project's required checks:
- `bun run lint`
- Relevant existing regression scripts, including `scripts/comprehensive-test.sh` and `scripts/new-mods-test.sh` where applicable
- Browser end-to-end checks for both PVP and robot challenges
- Review `dev.log` for new errors
- Verify DB/schema drift if any data model is changed; avoid schema changes unless genuinely required.

## Acceptance Criteria
- The child can answer without being blocked by routine background synchronization.
- The "جاري المزامنة" message no longer appears as an indefinite or blocking state during normal challenge play.
- PVP and AI challenges recover from temporary connection delays without resetting the match or losing confirmed answers.
- Answer submission remains server-authoritative, sequential, idempotent, and secure.
- No duplicate scoring, progress regression, stale state overwrite, or endless loading state is introduced.
- The agent reports the verified root cause, exact files changed, recovery behavior, tests run, and any remaining limitation.

## Do NOT
- Do not simply remove the text while leaving the input blocked.
- Do not solve the issue by accepting answers locally without server validation.
- Do not send correct answers/correctness to the client during live play.
- Do not remove idempotency, sequential validation, monotonic progress, or expiry checks.
- Do not replace Socket.IO/REST fallback architecture with a new transport.
- Do not alter the AI robot's server-side plan/verdict authority.
- Do not change scoring, wager, reward, or match-finalization rules unless the investigation proves a necessary bug and the change is explicitly documented.
- Do not use `pkill -f next`.
