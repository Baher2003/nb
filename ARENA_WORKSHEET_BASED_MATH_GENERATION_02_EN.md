# Implementation Task: Worksheet-Based Challenge Math Generation

**Task ID:** ARENA-WORKSHEET-BASED-MATH-GENERATION-02  
**Project:** ساحة العباقرة (Genius Arena)  
**Primary reference:** `PLATFORM_KNOWLEDGE_BASE.md`  
**Related specifications:** `BASE_50_100_ADDITION_SUBTRACTION_RULES_ENGINE_SPEC_EN.md` and `ARENA_MATH_DIGIT_CONTROLS_AND_ZERO_AUDIT_EN.md`

## 1. Source-Based Understanding of the Supplied Worksheets

Use the original user-supplied worksheet images/PDFs as the authoritative visual and mathematical reference. Do not replace their style with generic easy arithmetic.

### Multiplication patterns identified
- The supplied material includes multi-digit multiplication, including two-digit × two-digit problems.
- Products can reach the thousands. The answer field, grading, serialization, score/history records, and result review must support the full product without truncation.
- Do not silently restrict challenges to tiny one-digit factors when the selected settings call for larger operands.
- Display the problem clearly with a multiplication sign and enough room for the full answer.

### Division patterns identified
- The supplied material includes multi-digit dividends, including values in the thousands, and divisors that can have two digits.
- The platform knowledge base describes inverse-generation that guarantees an exact quotient. Preserve exact integer division unless the actual source has an explicit, approved decimal/remainder mode.
- Large dividends, divisors, and quotients must survive generation, display, input, grading, history, and final review without truncation.

### Addition/subtraction rule patterns
The earlier addition/subtraction materials and existing specification require Soroban complement techniques:
- Direct moves remain direct; `+5`, `-5`, `+10`, `-10`, `+50`, `-50`, `+100`, and `-100` must not be mislabeled as complement moves where the engine classifies them as direct.
- Complement-50 examples: `+10 = +50 -40`, `+20 = +50 -30`, `+30 = +50 -20`, `+40 = +50 -10`; subtraction counterparts are `-10 = -50 +40`, `-20 = -50 +30`, `-30 = -50 +20`, `-40 = -50 +10`.
- Complement-100 examples: `+10 = +100 -90` through `+90 = +100 -10`; subtraction counterparts: `-10 = -100 +90` through `-90 = -100 +10`.
- Higher-level questions can combine multiple enabled techniques in one sequence. Validate every intermediate move, not just the final total.
- These complement mappings come from the math specification, not from the multiplication/division pages.

**Evidence boundary:** This file summarizes the worksheet patterns previously identified; it does not claim that every exact numeral on every uploaded page has been transcribed. Inspect the original attachments during implementation and record representative examples with page references. If an original page is unavailable, report that limitation instead of inventing a transcription.

## 2. Goal

Audit and correct question generation for:
1. Friend/PvP challenges.
2. Student-versus-AI robot challenges.

Questions should reflect the supplied course worksheets, support large numbers where configured, avoid accidental overproduction of operands ending in zero, and use clear Admin controls for each operation and level. Fix the actual cause in generation, configuration, serialization, or rendering—not just the visual appearance.

## 3. Mandatory Reconnaissance

1. Read `/home/z/my-project/worklog.md`; append a concise work record at completion.
2. Read `PLATFORM_KNOWLEDGE_BASE.md` and both related specifications listed above.
3. Inspect the original worksheet images/PDFs directly. Record representative examples by operation, operand digit lengths, operation sign, and answer scale. Do not infer rules for one operation from another operation's examples.
4. Inspect current code, tests, types, API routes, settings keys, and components before editing. Do not invent APIs or assume previous specifications were implemented.
5. Reproduce several zero-heavy questions and identify whether the cause is generation, fallback generation, normalization, serialization, or rendering.

## 4. Architecture and Safety Constraints

- Keep exactly one authoritative addition/subtraction generator in `src/lib/rules-engine.ts`.
- Reuse multiplication/division generation in `src/lib/game-engine.ts`, fixing it only where inspection demonstrates a defect.
- PvP and AI must use the same server-side generators and validated settings for equivalent levels/operations.
- Never send correct answers or correctness information to clients during live matches.
- Preserve sequential-answer validation, idempotency, monotonic progress, match expiry, point/wager transactions, refunds, and finalization.
- AI verdict and response plan must remain tied to server-stored `robotPlanJson`.
- Use existing API route handlers for mutations; no server actions.
- Preserve Firestore/Supabase dual-backend behavior and current settings persistence patterns.
- Never create `src/app/page.tsx`; the app uses `src/app/[[...slug]]/page.tsx`.
- Preserve Arabic RTL, Cairo typography, mobile keypad behavior, and the current design system.
- Do not change scoring, prizes, or match business rules in this task.

## 5. Addition/Subtraction: Fix Number Shape and Multi-Rule Sequences

### Number shape
- Do not add leading zeros (for example, do not render `07` as a normal number).
- Do not force all operands to have equal digit lengths unless Admin explicitly chooses exact digit length.
- Mixed operand lengths are allowed when settings permit them, e.g. `7 + 24`.
- Ones and tens can naturally appear in the same question.
- Avoid almost exclusively round tens such as 20, 30, 40, 50, 60 when varied valid candidates exist.
- Do not ban all round tens/hundreds; they remain valid when appropriate.
- Do not alter operands solely to force a complement label.
- Fix the root cause rather than hiding zeros in the UI.

### Rule correctness
- Every move must be legal for the current level and enabled rules.
- Preserve rule priority from the existing specification: direct → complement 5 → complement 10 → complement 50 → complement 100.
- Respect floors, operand/result bounds, ones/tens permissions, term count, and enabled complement flags.
- Higher-level sequences may combine direct, comp5, comp10, comp50, and comp100 when enabled. Validate each intermediate state.
- Lower levels must not gain advanced rules merely because the generator needs another candidate.
- If no valid candidate is found, retry safely or report generation failure; never return an invalid fallback question.

### Distribution tests
Use deterministic seeds or a controlled sample per level. Report digit-length distribution, percentage of operands ending in zero, rule usage, validation rejection reasons, and mathematical correctness. Avoid arbitrary universal limits on zeros; catch accidental near-exclusive round-number generation when alternatives are valid.

## 6. Multiplication: Match the Worksheet Scale

The platform knowledge base documents current training ranges as `digits1: 1–4` and `digits2: 1–3`. Verify whether challenge generation actually passes and respects these settings; do not assume training settings automatically apply to Arena.

Admin must be able to configure each factor independently per applicable level:
- First factor digit count/range.
- Second factor digit count/range.
- Explicitly distinguish “exactly N digits” from “up to N digits.”
- Expose minimum/maximum operand values and difficulty/structure only if the current engine can safely support them.
- Support multi-digit × multi-digit questions, including two-digit × two-digit products in the thousands, where configured.
- Never clamp the product to the digit count of either factor.
- Verify product values are not truncated in generation, types, API payloads, display, answer entry, grading, score/history, or final review.

Use representative examples from the original pages to compare numeric scale and visual structure. Do not assume every level should use the largest range.

## 7. Division: Match Worksheet Scale and Exactness

The knowledge base documents training ranges as `dividendDigits: 2–4` and `divisorDigits: 1–2`, with exact quotient generation. Verify that the challenge path uses these settings or explain any intentional difference.

Admin controls should support, within safe engine limits:
- Dividend digit count/range.
- Divisor digit count/range.
- Quotient/result bounds where supported.
- Exact integer division by default.
- Decimal division only if an existing, explicit, safely implemented setting supports it.

Requirements:
- Support multi-digit dividends reaching thousands and two-digit divisors where configured.
- Keep quotient integer and remainder zero under the current exact-division rule.
- Ensure full values survive display, input, grading, history, and final review.
- Do not compare raw floating-point decimals naively if a verified decimal mode exists.

## 8. Clear, Independent Admin Settings for Every Level

The Admin currently finds settings visually mixed together. Improve the existing settings UI so each level is distinct.

For each supported study level (the intended product configuration covers Levels 1–10), create a clearly separated card/section with:
- Prominent level number/name and short description.
- Separate groups for Addition/Subtraction, Multiplication, and Division.
- Controls for digit/range, enabled rules, term count/complexity, and decimal options only where genuinely supported.
- Arabic helper text clarifying “exactly N digits” vs “up to N digits.”
- A save/loading/success/error state identifying the level being saved.
- Saved values reloaded from the server after refresh.
- Protection against one level's form state overwriting another's; warn before discarding unsaved changes.

Important existing distinction: study levels are 1–10, while the current Rules Engine has levels 1–4, mapped as `1→1`, `2→2`, `3→3`, `4–10→4`. Do not silently confuse these. If independent operation settings are needed for all ten study levels, design a backward-compatible schema after inspecting the current implementation and document the mapping clearly.

Reuse the current Admin screen and persistence path where possible. Do not create a parallel settings system.

## 9. Apply Settings to Both Challenge Modes

For both friend/PvP and AI match creation:
1. Resolve the authoritative study/engine level server-side.
2. Load and sanitize the applicable Admin settings.
3. Generate questions on the server.
4. Validate every question against operation, digit/range, and level constraints.
5. Store the question set and settings snapshot in the existing match model.
6. Ensure both PvP players see the same ordered question set.
7. Ensure AI planning uses those same stored questions and remains consistent with `robotPlanJson`.
8. Do not regenerate or reorder questions on reconnect/retry.

New settings apply to new matches only. Existing active matches retain their stored questions/settings. Never trust client-submitted digit counts.

## 10. Required Tests

### Addition/subtraction
- Mixed one-/two-digit operands when permitted.
- No leading-zero padding.
- Round numbers possible but not overproduced.
- Complement mappings correct.
- Higher-level questions combine multiple enabled rules correctly.
- Every intermediate move and final result valid.
- Lower-level restrictions preserved.

### Multiplication
- One-digit × one-digit, two-digit × one-digit, and two-digit × two-digit where configured.
- Correct products in the thousands.
- No truncation in display/input/grading/history/review.
- Invalid settings sanitized server-side.

### Division
- One- and two-digit divisors where enabled.
- Multi-digit dividends including thousands where enabled.
- Exact integer quotient and zero remainder when required.
- No truncation in display/input/grading/history/review.
- Decimal tests only if decimal mode already exists and is supported.

### Admin and PvP/AI consistency
- Save/reload each level independently.
- Switching levels does not leak settings.
- PvP and AI use the same settings for equivalent levels/operations.
- Both PvP clients receive identical ordered questions.
- Admin edits do not mutate active matches.
- Normal training settings remain unchanged unless explicitly included in scope.
- No live answer-key/correctness leak.
- Existing idempotency, progress, scoring, wagers, expiry, and finalization remain correct.

## 11. Verification

Run actual applicable checks:
1. `bun run lint`
2. Relevant Rules Engine and game-engine tests.
3. `scripts/comprehensive-test.sh`
4. `scripts/new-mods-test.sh`
5. Browser E2E for PvP and AI using small and large configured operands.
6. Inspect `dev.log`.
7. Run dual-backend drift checks if persistence/schema changes are made.

Report only tests that actually ran. State exact blockers for tests that could not run.

## 12. Required Completion Report

Report:
- Root cause of zero-heavy question generation.
- Representative worksheet examples inspected, with page references where possible.
- Current versus supported digit ranges for each operation.
- Exact files changed.
- How settings are persisted and shared by PvP/AI.
- How study levels 1–10 map to engine levels 1–4.
- Actual test commands and results.
- Any pattern or range that could not be verified or is not supported.

## 13. Do Not

- Do not claim every worksheet question was transcribed unless every question was inspected.
- Do not invent examples and attribute them to the user's pages.
- Do not merely hide zeros.
- Do not ban all round numbers.
- Do not create a second addition/subtraction generator.
- Do not create separate PvP and AI math generators.
- Do not expose correct answers during live play.
- Do not trust client-controlled ranges/rules.
- Do not change scoring, wagers, rewards, or finalization.
- Do not remove exact division unless the current specification explicitly permits another mode.
- Do not create `src/app/page.tsx`.
- Do not use `pkill -f next`.

## 14. Definition of Done

- Generated questions match the verified worksheet patterns as closely as supported.
- Large multiplication/division values work end-to-end when configured.
- Addition/subtraction avoids accidental zero-heavy generation and supports valid multi-rule sequences.
- Admin can configure levels without mixing settings.
- PvP and AI share the same authoritative question settings/generation.
- Required tests ran and worklog records files changed and actual results.
