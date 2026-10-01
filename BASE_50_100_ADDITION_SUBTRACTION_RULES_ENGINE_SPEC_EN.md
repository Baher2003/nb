# Task: Base-50 and Base-100 Addition/Subtraction in the Shared Rules Engine

**Project:** ساحة العباقرة (Genius Arena)  
**Task type:** Implementation specification  
**Authoritative references:** `PLATFORM_KNOWLEDGE_BASE.md` and `NEW_WEBSITE_MODIFICATIONS_02_AR.md`

## 1. Goal and Scope

This task has three connected goals:

1. Implement and test the Soroban/mental-math **complement-of-50 and complement-of-100 techniques for addition and subtraction** in the existing shared Rules Engine.
2. Inspect the supplied `level 9 A,B.pdf` worksheet screenshots as examples of the platform's expected **multiplication/division exercise format and number scale**. Preserve the existing multiplication/division rules, but verify that their generators and per-level settings can represent appropriately large operands/results where the current level configuration allows them. Do not silently impose small-number-only limits if the documented level settings and exercise model call for larger numbers.
3. Improve the Admin per-level math-settings experience so each level is visually distinct, logically grouped, and easy to configure without settings from different levels appearing mixed together.

The screenshots show a worksheet style with multiplication such as two-digit × two-digit problems (including examples with answers in the thousands) and division with multi-digit dividends and two-digit divisors, including exact integer-quotient exercises. Some pages show larger dividends/results. Treat these as examples of exercise shape and numeric range, not as a complete authoritative specification of every level. Inspect the actual source, existing settings, and `PLATFORM_KNOWLEDGE_BASE.md` before deciding which ranges are valid for each level. Preserve exact division where that is the existing game rule. Do not derive addition/subtraction complement mappings from multiplication/division answers, and do not rewrite multiplication/division algorithms as part of the base-50/base-100 implementation.

## 2. Mandatory Reconnaissance

Before editing:
1. Read `/home/z/my-project/worklog.md`; append a concise work record at completion.
2. Read `PLATFORM_KNOWLEDGE_BASE.md` and the math sections of `NEW_WEBSITE_MODIFICATIONS_02_AR.md`.
3. Inspect the actual current implementations and tests for:
   - `src/lib/rules-engine.ts` (the single authoritative generator/classifier)
   - `src/lib/training-engine.ts`
   - `src/lib/game-engine.ts`
   - `src/lib/arena-engine.ts`
   - `src/lib/ai-opponent.ts`
   - Relevant addition/subtraction training and arena API handlers
   - Per-level math settings, sanitization/validation, `math_levels_json`, and Admin controls
4. Confirm actual exported function names, types, rule IDs, setting keys, and test commands from source. Do not invent new APIs or assume a previous implementation is present without checking it.

## 3. Architecture and Safety Constraints

- Keep exactly one authoritative addition/subtraction generator in `src/lib/rules-engine.ts`.
- Reuse the existing rule registry/classifier and documented IDs such as `direct`, `comp5`, `comp10`, `comp50`, and `comp100` if they exist in the current source.
- Normal training, AI robot matches, and friend/PVP must use the same rules engine; do not create mode-specific math implementations.
- Preserve server authority, PVP answer/correctness secrecy, scoring, idempotency, session restoration, level matching, and current API contracts.
- Use existing API route handlers for mutations; do not introduce server actions.
- Preserve Firestore/Supabase dual-backend behavior. Avoid schema changes unless demonstrably necessary.
- Do not create `src/app/page.tsx`; the app uses `src/app/[[...slug]]/page.tsx`.
- Do not change multiplication, division, abacus, payouts, or unrelated UI/features.

## 4. Complement of 50

A complement transformation must preserve the exact net value.

### Addition
| Target move | Equivalent sequence |
|---:|---|
| `+10` | `+50 -40` |
| `+20` | `+50 -30` |
| `+30` | `+50 -20` |
| `+40` | `+50 -10` |

### Subtraction
| Target move | Equivalent sequence |
|---:|---|
| `-10` | `-50 +40` |
| `-20` | `-50 +30` |
| `-30` | `-50 +20` |
| `-40` | `-50 +10` |

`+50` and `-50` are direct operations when valid. They must not be registered as complement transformations.

## 5. Complement of 100

### Addition
| Target move | Equivalent sequence |
|---:|---|
| `+10` | `+100 -90` |
| `+20` | `+100 -80` |
| `+30` | `+100 -70` |
| `+40` | `+100 -60` |
| `+50` | `+100 -50` |
| `+60` | `+100 -40` |
| `+70` | `+100 -30` |
| `+80` | `+100 -20` |
| `+90` | `+100 -10` |

### Subtraction
| Target move | Equivalent sequence |
|---:|---|
| `-10` | `-100 +90` |
| `-20` | `-100 +80` |
| `-30` | `-100 +70` |
| `-40` | `-100 +60` |
| `-50` | `-100 +50` |
| `-60` | `-100 +40` |
| `-70` | `-100 +30` |
| `-80` | `-100 +20` |
| `-90` | `-100 +10` |

`+100` and `-100` are direct operations when valid. They must not be registered as complement transformations.

## 6. Fallback Complements for Values 1–9

The existing specification requires a 50/100 fallback for target values 1–9 only when direct movement and available/enabled smaller complements (5 and/or 10) cannot be used.

### Addition using 50
`+1 = +50 -49`; `+2 = +50 -48`; `+3 = +50 -47`; `+4 = +50 -46`; `+5 = +50 -45`; `+6 = +50 -44`; `+7 = +50 -43`; `+8 = +50 -42`; `+9 = +50 -41`.

### Subtraction using 50
`-1 = -50 +49`; `-2 = -50 +48`; `-3 = -50 +47`; `-4 = -50 +46`; `-5 = -50 +45`; `-6 = -50 +44`; `-7 = -50 +43`; `-8 = -50 +42`; `-9 = -50 +41`.

### Addition using 100
`+1 = +100 -99`; `+2 = +100 -98`; `+3 = +100 -97`; `+4 = +100 -96`; `+5 = +100 -95`; `+6 = +100 -94`; `+7 = +100 -93`; `+8 = +100 -92`; `+9 = +100 -91`.

### Subtraction using 100
`-1 = -100 +99`; `-2 = -100 +98`; `-3 = -100 +97`; `-4 = -100 +96`; `-5 = -100 +95`; `-6 = -100 +94`; `-7 = -100 +93`; `-8 = -100 +92`; `-9 = -100 +91`.

These are fallback representations, not default choices. Do not use them unnecessarily when a simpler legal operation exists.

## 7. Rule Selection Priority

Follow the existing documented priority, subject to current level configuration and valid abacus state:

1. Direct operation
2. Complement 5
3. Complement 10
4. Complement 50
5. Complement 100

Skip disabled or invalid techniques. A mathematically equivalent sequence is not automatically legal: validate the current place value, level configuration, intermediate moves, and result bounds.

Distinguish explicitly between:
- **Target operation:** e.g. `+30`
- **Technique selected:** direct, `comp50`, or `comp100`
- **Internal technique sequence:** e.g. `+50, -20`
- **Net value:** must exactly equal the target operation

Do not alter the student-facing presentation of terms unless required by the existing UI contract. The technique metadata must not accidentally change the actual mathematical answer.

## 8. Place Value, State, and Level Constraints

- Do not implement this as a global numeric substitution detached from the existing Soroban/place-value model.
- Validate intermediate corrections as well as the net value.
- Reject moves that are invalid for the current place/state, violate digit or answer bounds, or produce a forbidden intermediate state under the existing engine's rules.
- Preserve all existing level restrictions, including Level 2's ones-only final answer.
- Do not enable complement-50/100 in a level where current policy or Admin configuration prohibits it.
- For Level 3 and above, use 50/100 only when enabled by that level's validated configuration.
- If no legal technique can perform a candidate move, reject/regenerate the candidate rather than silently using a forbidden rule.
- Adapt validation to the existing engine's abstractions; do not introduce a parallel abacus state model.

## 9. Admin Configuration

Inspect existing per-level settings and Admin controls first.
- Reuse `math_levels_json` and its current sanitize/validate/update pipeline if that is the established mechanism.
- Add or correct per-level enable/disable controls for complement-50 and complement-100 only if missing.
- Missing legacy flags must receive safe defaults consistent with the documented level policy.
- Validate on the server, not only in the frontend.
- Preserve existing saved configurations and Arabic RTL design.
- Do not create a new settings page or persistence system if the existing Admin level settings can support this.
- If support already exists, verify and test it instead of duplicating it.

## 10. Multi-Rule Sequences in Higher Addition/Subtraction Levels

Higher levels may require one question to use **more than one legal technique** across its sequence of terms. Do not assume every generated question can be solved by applying one rule only.

- Inspect the current question format, `classifyMove`, rule registry, question generator, and level configuration to determine how a question represents multiple moves/terms.
- Support a sequence where different steps legitimately use different enabled techniques, for example a sequence containing a direct move, then a `comp50` move, then a `comp100` move, provided the level allows those techniques and every intermediate move is legal.
- The same question may use the same rule more than once or combine distinct rules. Keep each move's target operation, selected technique, internal sequence, and net value distinguishable in metadata/debugging/tests where the existing contracts allow it.
- Validate every step in order against the current place-value/abacus state, not only the final total. A valid final answer must not make an invalid intermediate sequence acceptable.
- Level configuration must control both which techniques are available and any existing limits on term count, digit/place-value range, answer range, and question complexity. Do not add arbitrary new settings if existing settings already express these constraints.
- Add deterministic tests for multi-step questions that combine direct + `comp50`, `comp50` + `comp100`, and three or more technique types where the current level model supports them. Assert the exact final value and each intermediate state.
- Keep lower levels restricted to their current permitted techniques; multi-rule composition must not accidentally enable advanced rules in lower levels.

## 11. Multiplication and Division Worksheet-Shape Audit

This is a compatibility/range audit, not permission to replace the multiplication/division algorithms.

1. Inspect the supplied screenshots and current `src/lib/game-engine.ts`, relevant training settings/types, Admin level settings, sanitizers, validators, and tests.
2. Compare the visible exercise shapes against what the current platform can generate. The examples include two-digit multiplication operands and division dividends reaching several thousand, with divisors sometimes two digits. Do not assume every level should use the largest range; respect each level's intended progression and current Admin-configurable ranges.
3. Where current configuration supports larger numbers but the generator, UI, input validation, serialization, or answer checking accidentally truncates or rejects them, fix that defect and add tests. Where a range is intentionally limited by documented rules, preserve it and explain the limit in the final report rather than expanding it without evidence.
4. Division must retain the existing exact-division/inverse-generation rule unless source inspection proves the project specification differs. Test larger dividends and divisors for integer quotient, correct answer, and safe numeric bounds.
5. Test multiplication with larger allowed operands and ensure product rendering, answer entry, checking, scoring, history, and result display handle the full answer without overflow/truncation or layout breakage.
6. Do not change the shared addition/subtraction Rules Engine to implement multiplication/division. Do not alter abacus behavior or unrelated games.
7. Report which screenshot patterns are already supported, which gaps were found, and which level settings govern the allowed ranges. Do not claim every screenshot number is supported unless verified in code and tests.

## 12. Admin Per-Level Settings: Clear, Separate, Organized

The Admin currently feels as though settings from multiple levels run together. Improve the existing UI so an administrator can immediately tell which settings belong to which level.

- Inspect the actual Admin level-settings component(s), current route/view, data contract, and `math_levels_json` persistence/sanitize/update flow before editing. Reuse the existing screen and storage.
- Give each level a clearly separated card/section with a prominent level number/name, short description of the level's purpose/current difficulty, and its own settings grouped inside it. Do not present all levels as one long visually blended form.
- Use a clear hierarchy: level header; enabled game types; addition/subtraction rules; multiplication settings; division settings; bounds/complexity; then save/status feedback, showing only groups that apply to that level. Do not invent controls unsupported by the existing schema.
- Make level boundaries visually obvious with consistent spacing, dividers/cards, headings, and aligned controls. Keep the calm educational design system, Cairo typography, Arabic RTL layout, responsive mobile behavior, and accessible focus/labels. Avoid excessive decoration or oversized typography.
- Prefer an accordion or tabs only if it makes levels easier to distinguish without hiding unsaved changes or making cross-level comparison confusing. Otherwise use separate cards with clear headers. Preserve currently saved values while switching levels/sections.
- Show an explicit save/loading/success/error state per level or clearly identify which level is being saved. Prevent one level's form state from accidentally overwriting another level's settings.
- Ensure controls for enabling/disabling `comp50` and `comp100` are clearly located within the corresponding level's Addition/Subtraction section. Higher-level multi-rule capability should be explained in simple Arabic helper text where useful.
- Validate settings on the server and preserve legacy configurations. Ensure the UI reflects sanitized/saved server values after reload.
- Keep the existing route and permission guards. Do not create a duplicate Admin settings system or unrelated redesign.
- Test levels 1–10 (or the actual supported set found in source), desktop and mobile widths, RTL alignment, save/reload, switching between levels with unsaved changes, and error handling.

## 13. Shared Integration

Verify that addition and subtraction generation for normal training, AI robot challenges, friend/PVP challenges, and every other relevant existing mode all use the same Rules Engine. No complement logic should be duplicated in UI components or API handlers.

## 14. Required Tests

Add/update deterministic tests in the repository's existing test framework.

### Complement-50 mapping
Test all eight listed targets: `+10`, `+20`, `+30`, `+40`, `-10`, `-20`, `-30`, `-40`. Assert exact sequence and exact net value.

### Complement-100 mapping
Test every target from `+10` to `+90` and `-10` to `-90` in increments of 10. Assert exact sequence and net value.

### Fallback 1–9
Test every value 1–9 in both directions for both 50 and 100. Verify complement-50 is preferred over complement-100 when simpler techniques are unavailable and both are enabled.

### Priority and feature flags
- Valid direct operation wins over complement.
- Complement-5 wins over complement-10/50/100 when valid and enabled.
- Complement-10 wins over complement-50/100 when valid and enabled.
- Disabled complement-50 or complement-100 is never selected.
- If 50 is disabled but 100 is enabled, 100 can be used only when no simpler legal technique works.
- If no legal technique exists, reject the candidate.
- Direct `+50`, `-50`, `+100`, `-100` remain direct rules, never complement rules.

### State and level constraints
- Invalid intermediate steps are rejected even if the net value is correct.
- Level 2 final answers remain ones-only.
- Levels 1–2 do not gain 50/100 techniques where prohibited.
- Level 3+ uses 50/100 only when enabled in validated configuration.
- Existing floor counts, operand/result bounds, and level settings remain enforced.

### Cross-mode regression
Verify training, robot, and PVP consume the same shared generator/rules. Ensure no live PVP answer correctness is exposed.

## 15. Verification Workflow

Run the actual applicable project commands, including:
1. `bun run lint`
2. Relevant Rules Engine unit/integration tests
3. `scripts/comprehensive-test.sh`
4. `scripts/new-mods-test.sh`
5. Relevant browser E2E tests for addition/subtraction training and challenges
6. Inspect `dev.log` for new errors
7. Dual-backend drift checks if persistence/schema behavior changes

Report actual outcomes only. If a test cannot run, state the exact command and blocker; do not claim it passed.

## 16. Explicit Do-Not List

- Do not replace or reverse-engineer the multiplication/division algorithms from worksheet photos; use them to audit exercise shape and supported numeric ranges only. Fix verified range/truncation defects only where consistent with the source specification.
- Do not alter unrelated math games.
- Do not create a second addition/subtraction generator.
- Do not duplicate complement logic across modes.
- Do not classify direct `+50`, `-50`, `+100`, or `-100` as complement rules.
- Do not prefer 50/100 fallback over a simpler legal operation.
- Do not bypass level flags, place-value/state validation, or answer bounds.
- Do not expose live PVP answer correctness.
- Do not remove idempotency, session persistence, rate limits, or existing security checks.
- Do not add an unnecessary database layer or schema changes.
- Do not make unrelated UI changes.
- Do not allow higher-level multi-rule questions to bypass per-step validation.
- Do not leave Admin level settings visually blended together or allow cross-level state leakage.
- Do not expand multiplication/division numeric ranges without checking the documented level progression and current settings.

## 17. Definition of Done

Complete only when:
- 50/100 complement addition/subtraction mappings work in the existing Rules Engine.
- Higher-level questions can combine multiple enabled techniques where the current question model supports multi-step sequences, with every intermediate step validated.
- Multiplication/division numeric ranges and worksheet-shaped examples have been audited; any verified large-number handling defects are fixed and tested without rewriting their algorithms.
- Admin level settings are visually separated by level, grouped clearly, and verified not to leak state between levels.
- Fallback values 1–9 work in both directions under the specified conditions.
- Direct operations and complement transformations are correctly distinguished.
- Level/Admin configuration is respected and validated.
- Training, AI robot, and PVP use the same authoritative rules.
- Tests cover mappings, priority, disabled rules, state validation, and level constraints.
- Lint and applicable regression tests have actually run, with outcomes recorded.
- `/home/z/my-project/worklog.md` records files changed, implementation decisions, and verification results.
- No unrelated math games or platform behavior regress.
