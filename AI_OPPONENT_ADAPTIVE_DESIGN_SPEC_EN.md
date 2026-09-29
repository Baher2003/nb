# AI Opponent Redesign Specification
## Adaptive, Human-Like, Synchronized AI Competitor for Children's Mental Math Challenges

> **Document type:** Standalone implementation specification.
>
> **Critical instruction:** Do not immediately rewrite the existing AI opponent. First inspect and understand exactly how the current AI works, how questions are generated, how answers are submitted, how timing is calculated, how the match state is stored, and how the frontend receives opponent updates.
>
> The objective is to transform the AI from a fixed/overly-fast opponent into a believable, adaptive competitor whose behavior is synchronized with the child and whose difficulty is based on the child's actual demonstrated ability.

---

# 1. Core Problem

The current AI opponent has several major problems:

1. It can solve problems unrealistically fast.
2. Its timing is not synchronized with the child.
3. It may appear to answer independently of the actual match timeline.
4. Its speed does not adequately reflect the child's current ability.
5. It may make the child feel that the AI is impossible to beat.
6. The AI may not sufficiently respect:
   - Current level.
   - Current difficulty.
   - Historical performance.
   - Current session performance.
   - Accuracy.
   - Speed.
   - Fatigue/consistency signals.
7. The opponent behavior may feel like a script rather than a human competitor.

This is especially problematic in a children's educational competition system.

The AI should not behave like a machine that simply computes the answer as fast as technically possible.

It should behave like a **simulated human competitor whose performance is intentionally modeled around the child's current ability**.

---

# 2. Mandatory First Step — Reverse Engineer the Existing AI

Before modifying the AI, inspect the entire current implementation.

Do not assume how it works.

Identify the exact mechanism currently used for:

## Question generation

Find:

- Where challenge questions are generated.
- How the question set is stored.
- Whether the AI receives the same questions as the child.
- Whether the AI generates a separate question set.
- Which service/function creates the question sequence.
- Whether the level/rules configuration is shared with normal training, robot challenge, and PVP.

## AI answer generation

Find:

- Where the AI answer is calculated.
- Whether it uses deterministic arithmetic.
- Whether it uses fixed delays.
- Whether it calculates a delay from difficulty.
- Whether it uses random timing.
- Whether it uses a hard-coded answer sequence.
- Whether AI results are generated before the child starts.
- Whether AI results are generated in real time.
- Whether AI progress is stored server-side.

## Timing

Determine:

- What starts the match clock?
- What starts the AI clock?
- Is the AI clock synchronized to the match start timestamp?
- Is AI timing based on wall-clock time?
- Is it based on frontend timers?
- Is it based on setTimeout/setInterval?
- Is timing trusted from the client?
- Does server time exist?
- Can frontend lag produce incorrect opponent timing?

## State synchronization

Find:

- Match/session IDs.
- Current question ID/sequence.
- AI current question.
- AI completed-question count.
- AI answer timestamps.
- Match start timestamp.
- Match completion timestamp.
- WebSocket/SSE/polling mechanism.
- APIs used to refresh AI state.
- Any localStorage/sessionStorage state.
- Any stale cache.
- Any duplicated state source.

## Child performance data

Find all available signals such as:

- Previous scores.
- Accuracy.
- Average response time.
- Best response time.
- Recent response time.
- Recent accuracy.
- Number of completed sessions.
- Current level.
- Current difficulty.
- Number of questions completed.
- Correct/wrong patterns.
- Progress history.
- Challenge history.

Before implementation, produce a short technical report showing:

```text
Current AI Architecture
Current Match Architecture
Current Timing Architecture
Current Question Architecture
Current Data Sources
Current Problems
```

---

# 3. Do Not Optimize the AI for Maximum Speed

This is a critical product rule.

The AI objective is:

```text
Provide a believable, educational, competitive opponent.
```

Not:

```text
Solve as fast as possible.
```

The AI should sometimes:

- Solve quickly.
- Solve at average speed.
- Take longer on difficult problems.
- Make occasional mistakes.
- Have natural variation between questions.
- Have performance variation across matches.

However, the behavior must remain controlled and fair.

Do not make the AI intentionally humiliating or impossible to beat.

Do not create a guaranteed AI win.

Do not create a guaranteed AI loss.

---

# 4. Use the Same Match Questions

Whenever possible, the AI and child should compete using the **same question sequence**.

Recommended:

```text
Match Created
      ↓
Generate Question Set
      ↓
Store Question Set / Seed
      ↓
Child receives Question 1
AI receives Question 1
      ↓
Child and AI progress independently
```

This makes the comparison meaningful.

Do not let the AI silently receive an easier or harder question set while presenting it as an equivalent competition.

---

# 5. Server-Authoritative Match Clock

The AI and child must use the same authoritative match timeline.

Recommended server state:

```text
match_id
match_started_at
question_sequence
player_progress
ai_progress
```

The backend should be authoritative for:

- Match start.
- Match end.
- Player progress.
- AI progress.
- Answer events.
- Final timing.

Do not rely on independent frontend timers for authoritative results.

---

# 6. Synchronization Model

The AI should behave as though it is participating in the same match.

Example:

```text
T = 0.0s
Match starts

T = 2.1s
Child answers Q1

T = 2.6s
AI answers Q1

T = 5.0s
Child answers Q2

T = 5.8s
AI answers Q2
```

The AI must not appear to complete all questions instantly at match start and then reveal the result later.

AI progress should be represented as realistic simulated answer events.

---

# 7. AI Performance Model

Create an explicit AI performance profile.

Recommended conceptual variables:

```text
child_accuracy
child_avg_time
child_recent_avg_time
child_best_time
child_level
child_difficulty
child_consistency
recent_form
question_complexity
```

The AI should derive a target performance range from these values.

Do not directly copy the child's exact speed.

Instead use a controlled relationship.

---

# 8. AI Performance Bands

Create internal behavioral modes such as:

```text
Supportive
Similar
Competitive
Challenging
```

These are internal behavior profiles, not necessarily child-facing labels.

Select an appropriate profile based on the child's demonstrated ability and current challenge level.

Do not expose internal formulas or ratings to the child.

---

# 9. Adaptive AI Between Matches

Use historical performance to estimate the child's current ability.

Example:

```text
Recent accuracy ↑
Recent speed ↑
Consistency ↑
        ↓
AI can become slightly more competitive
```

And:

```text
Recent accuracy ↓
Recent speed ↓
Consistency ↓
        ↓
AI should become more supportive/reachable
```

Do not react dramatically to a single answer.

Use a rolling window of meaningful recent performance.

---

# 10. Controlled Within-Match Adaptation

The AI may make small adjustments during a match, but avoid obvious rubber-banding.

Example:

```text
Child is significantly behind
        ↓
AI does not intentionally create a huge additional lead
        ↓
AI remains competitive and reachable
```

If the child is substantially outperforming the expected profile:

```text
Child performance exceeds expected ability
        ↓
AI can move toward the upper end of its allowed timing range
```

Adjust gradually.

Do not continuously alter AI speed after every answer.

Use:

- Multiple data points.
- Cooldowns.
- Small adjustment steps.
- Stable target profiles.

---

# 11. AI Timing Model

Do not use one fixed delay for every question.

Avoid:

```text
AI delay = 500ms
```

Instead model:

```text
AI response time =
base skill time
+ question difficulty adjustment
+ natural variation
+ reaction delay
+ consistency factor
```

Then constrain the result:

```text
minimum_time <= response_time <= maximum_time
```

The bounds should be level- and mode-aware.

---

# 12. Question Difficulty Must Affect AI Time

The AI should not answer every question in exactly the same amount of time.

Actual problem complexity should influence response time.

For example, these are not necessarily equal in cognitive difficulty:

```text
2 + 3
```

and:

```text
7 + 8 - 4 + 6
```

and:

```text
18 + 27 - 9 + 14
```

The difficulty analyzer should consider the actual mental-operation structure used by the platform.

Possible factors:

- Number of floors.
- Digit length.
- Carry requirements.
- Borrow requirements.
- Complement-5 operations.
- Complement-10 operations.
- Complement-50 operations.
- Complement-100 operations.
- Multiplication complexity.
- Division complexity.

---

# 13. Soroban / Mental-Math Awareness

Because the platform teaches mental arithmetic/Soroban techniques, difficulty must reflect the **mental transformation required**, not just the final numerical result.

For example:

```text
Direct move
```

versus:

```text
+7 → +10 -3
```

versus:

```text
+40 → +50 -10
```

versus:

```text
+90 → +100 -10
```

should be recognized as potentially different cognitive workloads.

The AI timing model should use the same problem/rule metadata as the child's training engine.

Do not build a separate difficulty interpretation that contradicts the actual training rules.

---

# 14. AI Accuracy Model

Do not make the AI 100% correct by default.

Use a controlled accuracy range based on:

- AI skill profile.
- Child ability.
- Level.
- Question difficulty.
- Match context.

Example conceptual ranges only:

```text
High skill       → 96–99%
Strong skill     → 93–97%
Developing skill → 88–95%
```

These numbers are placeholders for architecture/design and must be tuned using the application's real difficulty system and telemetry.

Do not hard-code them without validation.

---

# 15. Plausible AI Error Model

If the AI makes a mistake, do not replace the answer with an arbitrary random number.

Use plausible mistake categories when appropriate:

- Small arithmetic slip.
- Carry mistake.
- Borrow mistake.
- Complement mistake.
- Off-by-one error.
- Digit/transposition error where relevant.
- Delayed/failed response on a harder question.

The error probability should depend on AI skill and problem complexity.

Do not make the mistake pattern so predictable that children can exploit it.

---

# 16. Build a Child Ability Profile

The system should maintain an internal **Child Ability Profile**.

Do not reduce ability to one score.

Suggested dimensions:

```text
Speed
Accuracy
Consistency
Current Level
Current Difficulty
Recent Form
Long-Term Trend
Question-Type Strength
Question-Type Weakness
```

Example structure:

```text
Child Ability Profile

Level: 3

Accuracy: 91%
Recent Accuracy: 94%

Average Time: 3.2s
Recent Average Time: 2.8s

Consistency: High

Addition: Strong
Subtraction: Medium
Multiplication: Strong
Division: Developing
```

Use only data actually available.

Never invent performance data.

---

# 17. Weight Recent Performance More Heavily

A child's current ability can change.

Use a weighted/rolling model in which:

```text
Recent sessions > older sessions
```

Possible structure:

```text
Recent sessions: high weight
Previous sessions: medium weight
Old sessions: low weight
```

The exact weighting should be configurable.

---

# 18. Cold-Start Behavior

When there is not enough historical information:

- Do not pretend to know exact ability.
- Use level and current difficulty to choose a conservative initial AI profile.
- Gradually learn from the child's real performance.

Example:

```text
No/limited history
      ↓
Initial conservative AI profile
      ↓
Collect meaningful observations
      ↓
Estimate child ability
      ↓
Gradually adapt future AI matches
```

Do not dramatically change difficulty based on one answer.

---

# 19. Prevent Unrealistic AI Speed

The AI should not consistently be faster than a plausible human response for the current level unless the product explicitly defines an expert/superhuman mode.

Example of an unacceptable standard match:

```text
Child average = 3.5 seconds
AI average    = 0.2 seconds
```

unless a deliberate expert mode is enabled.

Normal child competition should use believable response ranges.

---

# 20. Fairness Constraint

Define an internal target relationship between the child's demonstrated ability and the AI's expected performance.

Conceptually:

```text
AI target performance
≈
Child demonstrated performance × controlled difficulty factor
```

The factor must vary according to:

- Level.
- Question difficulty.
- Child accuracy.
- Child speed.
- Child consistency.
- Selected challenge difficulty.

Do not use a single fixed percentage such as "AI is always 20% faster."

---

# 21. AI Progress Must Be Real-Time

AI progress should be represented by the same match event architecture.

Recommended event model:

```text
MATCH_STARTED
QUESTION_AVAILABLE
PLAYER_ANSWERED
AI_ANSWERED
PLAYER_FINISHED
AI_FINISHED
MATCH_COMPLETED
```

Each event should contain enough information for deterministic state updates.

Example:

```json
{
  "match_id": "ABC123",
  "participant": "AI",
  "question_index": 4,
  "event": "AI_ANSWERED",
  "answered_at": "SERVER_TIMESTAMP"
}
```

Do not expose internal AI parameters to the frontend.

---

# 22. Server Time Is the Source of Truth

Do not use the user's device clock as the authoritative timing source.

Use server timestamps where possible.

This protects against:

- Device clock manipulation.
- Timer drift.
- Different device performance.
- Refresh inconsistencies.
- Client-side result manipulation.

---

# 23. Refresh and Reconnection

If the child refreshes or temporarily disconnects:

- Restore the same match.
- Restore the same AI progress.
- Restore the same question sequence.
- Do not restart the AI.
- Do not generate a new AI opponent.
- Do not regenerate the question set.
- Continue from the authoritative server state.

If the challenge is intended to continue while disconnected, AI timing must continue from server state/time. If the application's rules intentionally pause a match, implement that explicitly.

---

# 24. Do Not Make Browser Code Authoritative

Do not make the browser responsible for official AI progress.

Never accept client-submitted authoritative values for:

```text
ai_score
ai_time
ai_correct
ai_progress
ai_answer
ai_completion_status
```

The server must generate or validate these values.

---

# 25. Consider an AI Simulation Engine Instead of an LLM for Every Question

Do not automatically use a generative language model to solve every math question.

For real-time mental-math competition, a deterministic engine plus a behavioral simulation model is generally easier to control and verify.

Recommended architecture:

```text
Question Engine
      ↓
Math Solver
      ↓
Difficulty Analyzer
      ↓
Child Ability Model
      ↓
AI Behavior Model
      ↓
Timing Model
      ↓
Match Engine
```

An LLM may optionally be used for higher-level analysis or future coaching/adaptive-learning features, but it should not be the sole authority for:

- Arithmetic correctness.
- Official answer generation.
- Official match timing.
- Official match state.

---

# 26. Separate Math Correctness From Human-Like Behavior

Use two distinct responsibilities.

## Math Solver

Responsible for:

- Computing the correct answer.
- Validating question generation.
- Understanding the active math rules.

## AI Behavioral Model

Responsible for:

- Whether the AI answers correctly.
- How long the AI takes.
- Natural timing variation.
- Performance profile.
- Difficulty adaptation.

This separation improves testing and tuning.

---

# 27. Suggested AI Architecture

Possible internal architecture:

```text
AI Match Controller
│
├── Child Ability Profiler
├── Question Difficulty Analyzer
├── Math Solver
├── AI Skill Profile
├── Accuracy Model
├── Response-Time Model
├── Error Model
├── Match State Manager
└── Event / Sync Manager
```

Keep responsibilities separated.

---

# 28. AI Skill Profile

Create a configurable internal profile with fields such as:

```text
skill_level
accuracy_mean
accuracy_variance
response_time_mean
response_time_variance
difficulty_sensitivity
consistency
mistake_rate
reaction_delay
```

Do not hard-code these values throughout multiple files.

Keep the model centralized and tunable.

---

# 29. Difficulty Analyzer

The difficulty analyzer should derive a problem complexity score from the actual problem metadata.

Possible inputs:

```text
floor_count
digit_length
carry_count
borrow_count
complement_5_count
complement_10_count
complement_50_count
complement_100_count
operation_type
operand_range
```

Possible output:

```text
difficulty_score = 0–100
```

Keep this score internal unless product requirements explicitly expose difficulty.

---

# 30. AI Response-Time Model

Conceptual model:

```text
response_time =
skill_base
+ difficulty_factor
+ reaction_delay
+ random_variation
+ consistency_adjustment
```

Then clamp:

```text
min_time <= response_time <= max_time
```

The minimum and maximum should be configurable and level-aware.

Do not use impossible timing values.

---

# 31. Natural Variation

A believable human competitor does not answer every question in exactly the same amount of time.

Example:

```text
Question 1 → 2.9s
Question 2 → 3.4s
Question 3 → 2.7s
Question 4 → 4.1s
Question 5 → 3.1s
```

The distribution should remain consistent with the AI skill profile.

Avoid pure white-noise randomness.

Variation should be controlled and reproducible enough for debugging when necessary.

---

# 32. Match Outcome Must Remain Genuine

Do not guarantee:

```text
Child wins
```

or:

```text
AI wins
```

The AI should be tuned to create a meaningful competition while allowing real performance to determine the result.

The child should be able to win through genuine performance.

---

# 33. Avoid Obvious Rubber-Banding

Do not constantly adjust AI speed after every child answer.

That creates artificial behavior.

Instead:

- Establish a target performance band.
- Make small gradual adjustments.
- Use multiple observations.
- Use cooldowns between adjustments.
- Avoid sudden unexplained changes.

The child should not feel that the AI is cheating whenever the child performs well.

---

# 34. Optional AI Skill Tiers

The system may internally define:

```text
AI Beginner
AI Developing
AI Intermediate
AI Advanced
```

These are internal profiles.

Map them to child ability and/or the selected challenge level.

The child-facing interface does not need to display technical AI parameters.

---

# 35. Admin Configuration

The Admin should eventually be able to control AI behavior without editing source code.

## AI Behavior

Potential settings:

- Enable/disable AI.
- Automatic skill selection.
- Adaptive mode.
- Minimum AI response time.
- Maximum AI response time.
- Accuracy range.
- Difficulty sensitivity.
- Timing variation.
- Error probability.
- Whether mistakes are allowed.
- Whether within-match adaptation is enabled.

## Child Adaptation

Potential settings:

- Historical data window.
- Recent-performance weight.
- Minimum observations before adaptation.
- Maximum adjustment per match.
- Maximum adjustment per question.
- Adaptation cooldown.

## Safety limits

Potential settings:

- Absolute minimum response time.
- Absolute maximum response time.
- Maximum accuracy.
- Maximum per-match adjustment.

Do not expose these technical settings to children.

---

# 36. Telemetry and Evaluation

Measure actual AI behavior after implementation.

Useful internal metrics:

```text
AI average response time
Child average response time
AI accuracy
Child accuracy
AI lead over time
Child win rate
AI win rate
Average time difference
Question difficulty vs AI response time
Question difficulty vs AI accuracy
```

Use this telemetry to tune the model.

Do not expose hidden telemetry to the child.

---

# 37. Automatic Unrealistic-Behavior Detection

Add internal validation/monitoring for:

- AI response times below configured human-like minimum.
- Multiple AI answers at impossible intervals.
- AI answering before the corresponding question is logically available.
- AI progress jumping forward.
- AI progress exceeding total question count.
- Duplicate AI answer events.
- Missing AI answer events.
- AI answering after match completion.
- Timestamp ordering violations.

Flag or reject invalid states.

---

# 38. Security Requirements

AI match state must be protected against client manipulation.

Never allow the client to control official:

```text
AI score
AI time
AI accuracy
AI progress
AI answer
AI completion state
```

These must be server-authoritative.

Also prevent users from changing another participant's match state by manipulating:

```text
match_id
question_id
participant_id
session_id
```

The backend must verify ownership and authorization.

---

# 39. Performance Requirements

The AI engine should be lightweight enough for real-time matches.

Do not call an expensive generative model for every arithmetic question unless testing proves it is necessary and the latency/cost is acceptable.

Prefer deterministic code for:

- Arithmetic.
- Question validation.
- Difficulty scoring.
- Timing simulation.
- Match state transitions.

Use more advanced AI only where it provides measurable benefit.

---

# 40. Backward Compatibility

Preserve:

- Existing Robot Challenge routes.
- Existing match history.
- Existing scoring/history.
- Existing user accounts.
- Existing points logic.
- Existing training rules.
- Existing session persistence.
- Existing question generation.

If database changes are required:

- Use migrations.
- Preserve existing data.
- Provide safe defaults for old users/matches.
- Do not reset historical records.

---

# 41. Implementation Workflow

## Phase 1 — Inspect

Audit the existing AI implementation.

Deliver:

```text
Current Architecture
Current Timing
Current Synchronization
Current Data Model
Current Problems
Root Cause
```

## Phase 2 — Design

Design:

```text
Child Ability Model
AI Skill Model
Difficulty Model
Timing Model
Accuracy Model
Error Model
Synchronization Model
```

## Phase 3 — Implement

Implement the smallest robust architectural change that can satisfy the requirements.

Do not rewrite unrelated features.

## Phase 4 — Test

Test all scenarios below.

## Phase 5 — Tune

Use measured telemetry to tune speed, accuracy, and adaptation.

Do not guess final timing parameters without testing.

---

# 42. Required Agent Report Before Coding

Before changing code, the Agent must explain:

1. Exactly how the current AI opponent works.
2. Where AI answers are generated.
3. Where AI timing is generated.
4. How the current match is synchronized.
5. Whether AI state is server-side or client-side.
6. How the question sequence is generated.
7. Which child-performance data already exists.
8. Why the current AI is too fast.
9. Which files/components/services/endpoints are involved.
10. Whether an upgrade of the current system is enough or a dedicated AI simulation engine is needed.

Do not skip this analysis.

---

# 43. Definition of Done

The redesign is complete only when:

- AI speed is no longer unrealistically fast.
- AI timing is synchronized with the authoritative match clock.
- AI progress is synchronized with match state.
- AI timing reflects actual question difficulty.
- AI difficulty reflects demonstrated child ability.
- AI adapts gradually between matches.
- Optional within-match adaptation is controlled and subtle.
- AI can make plausible mistakes when enabled.
- The AI is competitive but not guaranteed to win.
- The AI is not guaranteed to lose.
- Refresh does not restart the AI.
- Reconnection does not create a duplicate AI session.
- Client manipulation cannot control official AI results.
- The same question/rules architecture remains consistent across relevant modes.
- The model is measurable and tunable through telemetry.
- There is one clear authoritative source of truth for match state.

---

# 44. Final Product Goal

The Robot Challenge should feel like:

> "I am competing against another player who is around my level."

It should not feel like:

> "I am competing against a calculator that answers instantly."

The AI should simulate **human-like competitive performance** while remaining:

- Mathematically correct.
- Synchronized.
- Fair.
- Adaptive.
- Secure.
- Educationally appropriate.
- Tunable.
- Explainable to the development team.

The core loop is:

```text
Understand the child
        ↓
Understand the question
        ↓
Choose a suitable AI skill profile
        ↓
Solve using the official math engine
        ↓
Simulate realistic human response behavior
        ↓
Synchronize through the authoritative match engine
        ↓
Measure actual performance
        ↓
Adapt gradually
```

Do not optimize for "fastest AI".

Optimize for **fair, believable, adaptive competition**.
