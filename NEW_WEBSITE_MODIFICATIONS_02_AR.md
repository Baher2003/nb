# Website Modification Specification — Challenge UX, Math Rules Engine, Session Persistence, and Admin Controls

> **Document type:** Implementation specification  
> **Scope:** New modifications only, building on the current working project.  
> **Critical rule:** The existing functionality and previously completed fixes must remain intact. Do not regress routing, navigation, responsive behavior, challenge logic, training logic, authentication, scoring, or any existing feature.

---

# 1. Competitor Challenge Experience

## 1.1 Allow the first finished player to leave the waiting screen

Current behavior:
- When Player A finishes all questions before Player B, Player A is forced to remain on the challenge screen waiting for Player B.
- This waiting experience is unnecessary and can feel boring.

### Required behavior

When a player finishes all of their questions:

- Do **not** force the player to stay on the challenge screen.
- Allow the player to leave the challenge screen and return to the Challenge Arena.
- Leaving the waiting screen must **not** count as a loss.
- Leaving the screen must **not cancel the match**.
- The match must continue on the backend until the opponent finishes.
- The player's completed state must remain stored correctly.

Example:

```text
Player A finishes
        ↓
Player A leaves challenge screen
        ↓
Returns to Arena
        ↓
Player B continues playing
        ↓
Player B finishes
        ↓
Match result is finalized
```

---

## 1.2 Notify the finished player when the match is completed

When the opponent finishes and the final result becomes available:

- Store the completed match result.
- Mark it as available/unseen for the player who already finished.
- When that player enters the Challenge Arena again, check for completed matches that have not yet been presented to them.
- Show a clear popup/modal notification.

Example:

```text
"Your challenge result is ready."
```

The popup should provide an action such as:

```text
View Result
```

The notification must not create a duplicate result or duplicate match.

---

# 2. Hide Detailed Correct/Wrong Feedback During Live PVP

During an active competitor-vs-competitor match:

- Do not reveal detailed correct/wrong information in a way that exposes the player's progress.
- Do not reveal the opponent's answers.
- Do not reveal the opponent's answer history in real time.
- Do not continuously expose detailed accuracy statistics during the live match.
- The backend may store the data required to calculate the final result, but the frontend must not expose the detailed result before the match is complete.

## After both players finish

Only after the match is finalized may the system display, where applicable:

- Total correct answers.
- Total wrong answers.
- Accuracy percentage.
- Final score.
- Time.
- Speed/performance statistics.
- Answer-by-answer review.
- Other finalized match statistics.

---

# 3. Answer History / Review Data

The answer history should not permanently occupy the live challenge UI.

### Required behavior

- Do not continuously show the answer history during the match.
- Do not continuously show the opponent's answers.
- Show the detailed answer review only after the match is finished.
- Prefer a dedicated result/review section or modal rather than keeping the answer history inside the live challenge layout.
- Avoid storing duplicate frontend copies of the same answer history.
- Clear temporary frontend state/cache when it is no longer required.

### Important data rule

Do **not** delete the official backend match record or any permanent data needed for:

- Final scoring.
- Match history.
- Statistics.
- Auditing.
- Reopening the final result.

Only temporary presentation/state data should be cleaned when no longer needed.

---

# 4. Math Rules Engine — General Requirements

The addition and subtraction engine must use a **real rules engine**, not only hard-coded examples or visual formatting.

The generated problems must match the rules allowed for each level.

The system must distinguish between:

1. The digits/numbers allowed inside the problem.
2. The natural result of the operation.
3. The complement rule used to perform the operation.
4. The number of floors/terms.
5. The level-specific difficulty.

Do not apply an advanced complement rule to a level where that rule is not allowed.

The same rules engine should be reused across:

- Normal training.
- Robot Challenge.
- Friend Challenge / PVP.
- Any other addition/subtraction problem generator.

Do not create separate inconsistent rule implementations for different modes.

---

# 5. Level 1 — Direct Ones Only

## Allowed content

Level 1 must use:

- Ones only.
- Direct/basic operations.
- No complement-of-5 rule.
- No complement-of-10 rule.
- No tens inside the problem.
- The expected result must remain in the ones range.

### Minimum number of floors

Every generated problem must contain at least:

```text
3 floors
```

and support:

```text
3 floors
4 floors
5 floors
```

The exact distribution can be controlled by the Admin (see Section 12).

---

# 6. Level 2 — Complement of 5 Only

Level 2 must use:

- The complement-of-5 rule.
- **No complement-of-10 rule.**

## Very important Level 2 result restriction

Level 2 problems and answers must be **ones only**.

Do **not** generate a Level 2 problem whose final answer becomes a tens value.

Examples that would produce a tens result must be excluded by the generator.

The generator must validate the final result before accepting a problem.

### Floors

Support:

```text
3 floors
4 floors
5 floors
```

with Admin control over the exact configuration.

---

# 7. Level 3 — Complement of 5 + Complement of 10

Level 3 must support:

- Complement of 5.
- Complement of 10.
- Ones and tens as required by the selected difficulty.
- Final answers may naturally contain ones and tens.
- Multi-floor operations.

### Floors

Support:

```text
3 floors
4 floors
5 floors
```

The Admin must be able to control the problem structure and difficulty.

---

# 8. Level 4 — Advanced Rules

Level 4 should continue the advanced rules model and may use:

- Complement of 5.
- Complement of 10.
- Complement of 50.
- Complement of 100.
- Ones and tens as appropriate.
- More complex multi-floor operations.

The progression must remain controlled and intentional rather than random.

Support:

```text
3 floors
4 floors
5 floors
```

and any additional floor count only when enabled by Admin.

---

# 9. Complement of 5 Rules

## Addition

Use complement-of-5 only when the direct movement is not available and the level permits this technique.

```text
+1 = +5 -4
+2 = +5 -3
+3 = +5 -2
+4 = +5 -1
```

## Subtraction

Use the inverse relationship:

```text
-1 = -5 +4
-2 = -5 +3
-3 = -5 +2
-4 = -5 +1
```

## Important restriction

Do **not** define these as complement transformations:

```text
+5 = +5
-5 = -5
```

`+5` and `-5` are direct operations, not complement rules.

They must not be listed or treated as a complement rule in the Rules Engine.

---

# 10. Complement of 10 Rules

## Addition

When the level permits the complement-of-10 technique:

```text
+6 = +10 -4
+7 = +10 -3
+8 = +10 -2
+9 = +10 -1
```

For completeness, the same complement relationship can also represent smaller values when explicitly required by the algorithm:

```text
+1 = +10 -9
+2 = +10 -8
+3 = +10 -7
+4 = +10 -6
```

However, the engine should prefer the simpler allowed rule for the current level rather than unnecessarily converting a direct/smaller move into a 10-complement operation.

## Subtraction

```text
-6 = -10 +4
-7 = -10 +3
-8 = -10 +2
-9 = -10 +1
```

For completeness:

```text
-1 = -10 +9
-2 = -10 +8
-3 = -10 +7
-4 = -10 +6
```

## Important restriction

Do **not** define these as complement transformations:

```text
+10 = +10
-10 = -10
```

`+10` and `-10` are direct operations and must not be treated as complement rules.

---

# 11. Tens-Level Complement Rules

When the ten-base complement logic crosses the required boundary, the engine must support the following relationship:

## Addition

For example:

```text
+5 = +10 -5
+6 = +10 -4
+7 = +10 -3
+8 = +10 -2
+9 = +10 -1
```

## Subtraction

The inverse form must also be supported:

```text
-5 = -10 +5
-6 = -10 +4
-7 = -10 +3
-8 = -10 +2
-9 = -10 +1
```

### Important

`+5 = +10 -5` and `-5 = -10 +5` are complement representations used when the algorithm needs them.

They are **not** the same as the direct-operation statements:

```text
+5 = +5
-5 = -5
```

The direct form must not be stored as a complement rule.

---

# 12. Complement of 50 — Level 3 and Above

Starting from Level 3, support the 50-base complement when applicable.

## Addition

```text
+10 = +50 -40
+20 = +50 -30
+30 = +50 -20
+40 = +50 -10
```

## Subtraction

```text
-10 = -50 +40
-20 = -50 +30
-30 = -50 +20
-40 = -50 +10
```

## Important restriction

Do **not** define:

```text
+50 = +50
-50 = -50
```

as complement rules.

`+50` and `-50` are direct operations.

---

# 13. Complement of 100 — Level 3 and Above

Starting from Level 3, support the 100-base complement when applicable.

## Addition

```text
+10 = +100 -90
+20 = +100 -80
+30 = +100 -70
+40 = +100 -60
+50 = +100 -50
+60 = +100 -40
+70 = +100 -30
+80 = +100 -20
+90 = +100 -10
```

## Subtraction

```text
-10 = -100 +90
-20 = -100 +80
-30 = -100 +70
-40 = -100 +60
-50 = -100 +50
-60 = -100 +40
-70 = -100 +30
-80 = -100 +20
-90 = -100 +10
```

## Important restriction

Do **not** define:

```text
+100 = +100
-100 = -100
```

as complement rules.

These are direct operations.

---

# 13.1 Fallback Complements for Values 1–9 Using 50 and 100

The Rules Engine must also support a fallback mechanism for **values 1 through 9** when the required direct move cannot be performed and the available smaller complement bases (`5` and/or `10`) cannot be used.

This fallback must be treated as an actual calculation rule, not merely as documentation.

## Addition — 50-base fallback

For values `1–9`, when direct movement, the 5-complement, and the 10-complement are unavailable or explicitly disabled for the current level/configuration, the engine may represent the value using the 50-base complement:

```text
+1 = +50 -49
+2 = +50 -48
+3 = +50 -47
+4 = +50 -46
+5 = +50 -45
+6 = +50 -44
+7 = +50 -43
+8 = +50 -42
+9 = +50 -41
```

## Subtraction — 50-base fallback

Use the inverse form:

```text
-1 = -50 +49
-2 = -50 +48
-3 = -50 +47
-4 = -50 +46
-5 = -50 +45
-6 = -50 +44
-7 = -50 +43
-8 = -50 +42
-9 = -50 +41
```

## Addition — 100-base fallback

If the 50-base fallback is also unavailable or disabled, the engine may use the 100-base complement for values `1–9`:

```text
+1 = +100 -99
+2 = +100 -98
+3 = +100 -97
+4 = +100 -96
+5 = +100 -95
+6 = +100 -94
+7 = +100 -93
+8 = +100 -92
+9 = +100 -91
```

## Subtraction — 100-base fallback

Use the inverse form:

```text
-1 = -100 +99
-2 = -100 +98
-3 = -100 +97
-4 = -100 +96
-5 = -100 +95
-6 = -100 +94
-7 = -100 +93
-8 = -100 +92
-9 = -100 +91
```

## Rule priority

The engine should choose the simplest valid method according to the current level and Admin configuration.

Recommended priority:

```text
Direct operation
    ↓
Complement 5
    ↓
Complement 10
    ↓
Complement 50
    ↓
Complement 100
```

However, if a specific rule is disabled for the current level, skip it and move to the next allowed rule.

### Example

If the system needs to perform:

```text
+7
```

and:

- direct `+7` is not available,
- complement 5 is disabled/unavailable,
- complement 10 is disabled/unavailable,
- complement 50 is enabled,

then the engine may use:

```text
+7 = +50 -43
```

If 50 is also disabled/unavailable but 100 is enabled:

```text
+7 = +100 -93
```

The same logic applies in reverse for subtraction.

### Important

These 50/100 representations are **fallback complement representations**. They must not be confused with direct operations:

```text
+50
-50
+100
-100
```

Direct `+50`, `-50`, `+100`, and `-100` remain direct operations and are not themselves complement rules.

# 14. Direct Operations vs Complement Operations

The Rules Engine must clearly separate **direct operations** from **complement operations**.

### Direct operations

Examples:

```text
+5
-5
+10
-10
+50
-50
+100
-100
```

These should be treated as direct moves when available.

### Complement operations

Examples:

```text
+1 = +5 -4
+4 = +5 -1
+7 = +10 -3
+9 = +10 -1
+40 = +50 -10
+90 = +100 -10

-1 = -5 +4
-4 = -5 +1
-7 = -10 +3
-9 = -10 +1
-40 = -50 +10
-90 = -100 +10
```

The engine should choose the valid direct/complement technique according to the level and current state.

---

# 15. Rule Availability by Level

The intended rule configuration is:

| Level | Ones in Problem | Direct Ones | Complement 5 | Complement 10 | Tens Allowed | Complement 50 | Complement 100 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Level 1 | Yes | Yes | No | No | No | No | No |
| Level 2 | Yes | Yes | Yes | No | No | No | No |
| Level 3 | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Level 4 | Yes | Yes | Yes | Yes | Yes | Yes | Yes |

### Critical Level 2 clarification

Level 2 must **not** produce a final answer containing tens.

The final result must remain a ones value.

---

# 16. Problem Floor / Term Count

The minimum problem length is:

```text
3 floors
```

The system must support:

```text
3 floors
4 floors
5 floors
```

The same principle must be applied to the relevant levels and training/challenge modes.

The actual term count must be configurable by Admin.

---

# 17. Admin Control — Problem Structure for Each Level

Add an Admin configuration area that allows the Admin to control the **problem format and generation rules separately for each level**.

The Admin should not be forced to edit code to change these settings.

## Per-level configuration should include, as applicable:

### Basic structure

- Minimum floors.
- Maximum floors.
- Allowed exact floor counts.
- Number of problems per session.
- Number of digits/number length.
- Ones-only mode.
- Tens-enabled mode.
- Allowed value ranges.
- Minimum/maximum answer range.

### Rule controls

Allow Admin to enable/disable, per level:

```text
Direct Operations
Complement 5
Complement 10
Complement 50
Complement 100
```

The system must enforce logical dependencies so an invalid combination cannot create unsupported problems.

### Difficulty controls

Allow Admin to configure:

- Easy/medium/hard distribution.
- Maximum operand size.
- Maximum result.
- Carry/borrow frequency where applicable.
- Allowed number patterns.
- Repetition rules.
- Whether the problem can cross from ones to tens.
- Whether a specific rule is required, optional, or prohibited.

### Example

Admin should be able to define:

```text
Level 1
- Ones only
- 3–5 floors
- Direct only
- Complement 5 OFF
- Complement 10 OFF
- Tens OFF
```

and:

```text
Level 2
- Ones only
- 3–5 floors
- Complement 5 ON
- Complement 10 OFF
- Final answer must remain ones
```

and:

```text
Level 3
- Ones + tens
- 3–5 floors
- Complement 5 ON
- Complement 10 ON
- Complement 50 ON
- Complement 100 ON
```

The UI should make these controls easy to understand.

---

# 18. Admin Controls Must Affect All Relevant Modes

The per-level configuration must be used by:

- Normal addition training.
- Normal subtraction training.
- Robot addition challenge.
- Robot subtraction challenge.
- Friend/PVP addition challenge.
- Friend/PVP subtraction challenge.
- Any other feature using the same problem generator.

There must be a single source of truth for the rules configuration.

Do not allow the normal training generator, robot generator, and PVP generator to silently use different level rules.

---

# 19. Problem Validation Before Delivery

Every generated problem must pass validation before being shown to the student.

Validation must check:

- Correct level rules.
- Allowed digits.
- Allowed floor count.
- Allowed rule set.
- Final answer range.
- Level-specific tens restriction.
- No forbidden complement rule.
- No invalid or impossible operation sequence.
- No accidental use of a higher-level technique.
- Correct mathematical result.

### Level 2 example

If a generated sequence results in:

```text
10
11
12
...
```

the problem must be rejected because Level 2 requires the final answer to remain in the ones range.

Generate another valid problem instead.

---

# 20. Session Persistence — Training

The training session must survive normal page reloads.

If the user is in an active training session and refreshes the page:

- Do not restart the training.
- Do not create a duplicate session.
- Restore the existing session.
- Restore the current question.
- Restore completed questions.
- Restore recorded answers.
- Restore correct/wrong counts.
- Restore current score.
- Restore progress.
- Restore timer state where technically possible.
- Restore the training settings.

Example:

```text
Training
  ↓
Question 7
  ↓
Refresh
  ↓
Restore same session
  ↓
Continue from Question 7
```

The system must not reset to Question 1 unless the session has actually been completed/cancelled/expired according to the application's rules.

---

# 21. Session Persistence — Challenge / PVP

The same persistence model must apply to:

- Robot Challenge.
- Friend Challenge.
- PVP.

When the user refreshes:

- Keep the same match/session.
- Do not create another match.
- Do not reset the current question.
- Do not lose submitted answers.
- Do not lose score/progress.
- Do not log the player out.
- Restore the active match using the stored match identifier.

The backend should be the authoritative source of match state.

Example:

```text
Match ID: ABC123
Current Question: 6
Player Progress: 5 completed
Refresh
        ↓
Restore Match ID: ABC123
        ↓
Continue from Question 6
```

---

# 22. Orientation Handling on Mobile

The training and challenge interfaces are intended primarily for **Portrait orientation**.

When the device changes to Landscape:

- Show a clear overlay asking the user to rotate the phone back to Portrait.
- Example:
  **"Please rotate your phone back to portrait mode to continue."**
- Do not allow the landscape layout to become broken or unusable.
- When the phone returns to Portrait, restore the exact same screen/state.

Orientation changes must never:

- Restart training.
- Restart the challenge.
- Create a new match.
- Lose the current question.
- Lose progress.
- Lose score.
- Log the player out.

---

# 23. Session Restoration Flow

When opening a training/challenge route, the system should check:

```text
Is there an active session?
        ↓
      YES → Restore existing session
        ↓
       NO → Start a new session
```

Do not start a new session when an active recoverable session already exists.

When a session is permanently completed, cancelled, or expired:

- Clear temporary session state.
- Prevent accidental restoration of an old completed session.
- Keep permanent historical records when required.

---

# 24. Final Acceptance Tests

## Competitor challenge

- Player A finishes before Player B.
- Player A can leave the waiting screen.
- Leaving does not count as a loss.
- Match continues in backend.
- Player B finishes.
- Result is finalized.
- Player A returns to Arena.
- A popup announces that the result is ready.
- Detailed opponent answers were not exposed during the match.
- Final answer review is available only after completion.

## Level 1

- Ones only.
- Direct operations only.
- No complement 5.
- No complement 10.
- No tens.
- 3–5 floors.

## Level 2

- Ones only.
- Complement 5 allowed.
- Complement 10 forbidden.
- Final answer must remain ones only.
- 3–5 floors.

## Level 3

- Ones + tens.
- Complement 5 allowed.
- Complement 10 allowed.
- Complement 50 allowed.
- Complement 100 allowed.
- 3–5 floors.

## Level 4

- Advanced rules allowed.
- Complement 5 / 10 / 50 / 100 according to Admin configuration.
- Progressive difficulty.
- 3–5 floors or additional counts only when explicitly enabled.

## Refresh

- Training state survives refresh.
- Robot challenge state survives refresh.
- PVP state survives refresh.
- Same session/match is restored.
- No duplicate session is created.

## Orientation

- Landscape shows the portrait-required overlay.
- Returning to Portrait restores the same state.
- No progress is lost.

## Admin

- Admin can configure each level independently.
- Admin changes affect the actual problem generator.
- Admin does not need to modify source code.
- Configuration is applied consistently to normal training, robot challenge, and PVP.

---

# 25. Non-Regression Requirements

During implementation:

- Do not remove existing features.
- Do not reset user data.
- Do not break authentication.
- Do not break scoring.
- Do not break timers.
- Do not break PVP.
- Do not break robot challenge.
- Do not break normal training.
- Do not create a second conflicting Rules Engine.
- Do not hard-code level behavior in multiple unrelated files if a shared configuration/service can be used.
- Keep the implementation maintainable and extensible.

---

# Final Goal

The final system should provide:

1. A less frustrating competitor experience where a finished player does not have to remain trapped on a waiting screen.
2. No live exposure of detailed correct/wrong answers or opponent answer history.
3. Reliable restoration of active training and challenge sessions after refresh or orientation changes.
4. Portrait-first mobile behavior without losing progress.
5. A mathematically correct level-based addition/subtraction Rules Engine.
6. Correct separation between direct operations and complement operations.
7. Strict Level 2 ones-only final answers.
8. Support for 5, 10, 50, and 100 complement techniques at the intended levels.
9. One shared Rules Engine across normal training, robot mode, and PVP.
10. Full Admin control over the problem structure and difficulty of every level.
