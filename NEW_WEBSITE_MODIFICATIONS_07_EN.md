# NEW_WEBSITE_MODIFICATIONS_07_EN.md
## Fully Open, Admin-Configurable Math Levels 1–10 + Unified Level Control UI

**Task ID:** NEW_WEBSITE_MODIFICATIONS_07  
**Priority:** High  
**Type:** Admin configuration + Rules Engine + Training/PvP/AI compatibility + UI/UX  
**Language:** English implementation prompt; all user-facing UI remains Arabic RTL.

---

## 1. Goal

Rebuild the current **Admin → Levels & Rules** configuration system so that **all study levels 1 through 10 are fully open and independently configurable**.

The current architecture treats math/rules levels as only 1–4 and collapses study levels 4–10 into the same engine level. That restriction must be removed.

The final system must allow the Admin to configure every level **1, 2, 3, 4, 5, 6, 7, 8, 9, and 10** independently.

For every level, the Admin must be able to decide:

- question structure;
- question size;
- number of operands / terms;
- allowed numeric range;
- maximum operand size;
- maximum/minimum result size;
- whether ones or tens are allowed;
- which Soroban rules are enabled;
- difficulty;
- multiplication settings;
- division settings;
- decimal-number behavior;
- and any equivalent existing generator setting that is already supported by the platform.

There must be **no hard-coded educational restriction that prevents the Admin from combining rules/settings**.

The Admin should be able to make Level 1 mathematically difficult, Level 10 simple, or any other valid combination. The system must follow the Admin configuration rather than imposing its own curriculum assumptions.

---

# 2. Source of Truth / Existing Architecture

Before changing code:

1. Read `/home/z/my-project/worklog.md`.
2. Read and follow `PLATFORM_KNOWLEDGE_BASE.md`.
3. Inspect the current implementation of:
   - `src/lib/rules-engine.ts`
   - `src/lib/game-engine.ts`
   - `src/lib/training-engine.ts`
   - `src/lib/arena-engine.ts`
   - `src/lib/ai-opponent.ts`
   - `src/lib/db-types.ts`
   - `src/lib/api-helpers.ts`
   - `src/app/api/admin/levels/state/route.ts`
   - `src/app/api/admin/levels/settings/route.ts`
   - the current Admin Levels view/panel under `src/components/views/admin/`
   - the current training game settings components
   - any shared types/constants related to level configuration.

**Critical architecture rule:** keep the existing single source of truth for addition/subtraction question generation in:

`src/lib/rules-engine.ts`

Do NOT create a second addition/subtraction rules generator.

All training, PvP and AI/robot addition-subtraction questions must continue to use the same Rules Engine.

---

# 3. Replace the Current 1–4 Math-Level Model

## Current limitation

The current system stores only four math-level configurations and maps:

- study level 1 → math level 1
- study level 2 → math level 2
- study level 3 → math level 3
- study levels 4–10 → math level 4

This mapping is no longer acceptable.

## Required behavior

Create a true independent configuration for:

`Level 1` through `Level 10`

Every study level must have its own configuration object.

The engine must no longer collapse levels 4–10 into Level 4.

Replace logic such as:

```ts
1 -> L1
2 -> L2
3 -> L3
4-10 -> L4
```

with direct lookup:

```ts
studyLevel 1  -> config level 1
studyLevel 2  -> config level 2
...
studyLevel 10 -> config level 10
```

The Admin configuration is authoritative.

Do not silently substitute another level when a valid configured level exists.

---

# 4. The 1–4 UI Must Become the Template for 5–10

The screenshot provided with this request is the visual reference.

The existing Level 1–4 cards already establish the intended pattern:

- level header;
- question-count / problem-structure control;
- rule controls;
- maximum value controls;
- difficulty selector;
- explanatory/helper text;
- consistent card spacing;
- consistent RTL alignment;
- consistent switches/sliders/buttons.

Use **the exact same visual system and component pattern** for Levels 5–10.

Do not create a second visual design for the higher levels.

The final page should visually feel like:

`Level 1`
`Level 2`
`Level 3`
`Level 4`
`Level 5`
`Level 6`
`Level 7`
`Level 8`
`Level 9`
`Level 10`

with the same card structure, control styling, spacing, labels, switches, sliders, and interaction model.

Only the data/configuration changes by level.

---

# 5. Remove All Artificial Locks

The current UI shows some rules with lock icons and disabled controls because of hard-coded level restrictions.

Remove this behavior.

For Levels 1–10:

- no rule is locked because of the level number;
- no operator is locked because of the level number;
- no configuration is disabled merely because it is considered "too advanced";
- no automatic educational restriction should override the Admin;
- no hidden rule matrix should silently undo the Admin's choice.

Every configurable rule must be editable.

The Admin should be able to enable, disable, or combine any supported rule in any level.

Example:

- Level 1 can use direct + complement 5 + complement 10.
- Level 1 can use tens.
- Level 1 can use complement 50.
- Level 10 can use direct-only.
- Level 10 can use simple one-digit questions.
- Any level can be configured to be easy, medium, or hard.

The level number itself must not determine what is allowed.

---

# 6. Remove the Existing Restrictive `sanitizeLevelConfig` Matrix

The current implementation contains a restriction matrix that enforces things such as:

- Level 1 cannot use complements/tens;
- some complements require `tensAllowed`;
- some combinations are automatically disabled;
- invalid combinations are forced back into allowed curriculum rules.

That behavior conflicts with this new requirement.

Refactor:

`sanitizeLevelConfig`

so it performs **technical safety validation and normalization only**, not curriculum restrictions.

It may still:

- validate data types;
- clamp impossible numeric values;
- normalize missing fields;
- enforce absolute generator safety limits;
- prevent NaN / Infinity;
- prevent malformed JSON;
- prevent impossible arithmetic states;
- prevent server abuse or pathological resource use.

It must NOT say:

> "Level X cannot use this rule because Level X is supposed to be easier."

The Admin's rule selection must survive sanitization whenever the selection is mathematically/technically valid.

---

# 7. Addition/Subtraction — Fully Configurable Per Level

For every one of Levels 1–10, expose the existing addition/subtraction controls and make them editable independently.

At minimum, support the existing concepts already present in the Rules Engine:

### Question structure

Allow Admin control over:

- minimum number of terms;
- maximum number of terms;
- number of operands;
- whether the question can be short or long;
- any existing floor/target structure control already implemented by the engine.

### Numeric size

Allow Admin control over:

- minimum operand;
- maximum operand;
- minimum result;
- maximum result;
- ones-only;
- tens allowed;
- any existing hundreds/greater place-value setting supported by the current engine.

Do not invent arbitrary mathematical limitations that are not necessary.

### Rules

Each level must independently expose toggles for:

- Direct
- Complement 5
- Complement 10
- Complement 50
- Complement 100

Keep the existing rule IDs and terminology.

Do not convert direct moves into complement rules.

Direct moves remain direct:

```text
+5 = +5
-5 = -5
+10 = +10
-10 = -10
+50 = +50
-50 = -50
+100 = +100
-100 = -100
```

Complement rules remain complement rules, including the existing examples such as:

```text
+5 = +10 -5
-5 = -10 +5
```

and the existing complement logic already defined by the platform.

### Rule combinations

A level may have any combination of the supported rules.

Examples:

```text
Direct only
Direct + comp5
Direct + comp10
Direct + comp50
Direct + comp100
Direct + comp5 + comp10
comp50 + comp100
All rules
```

No level-based lock should prevent these combinations.

---

# 8. Rule Fallback Must Remain Mathematically Correct

Preserve the existing fallback philosophy.

If a selected rule cannot legally be used for a specific generated number, the engine may choose another **enabled** rule that can represent that operation.

But the fallback must respect the Admin's enabled-rule set.

Do NOT:

1. enable a rule automatically;
2. use a disabled rule silently;
3. change the Admin configuration;
4. force a level back to a default curriculum.

The generated problem must remain valid and mathematically correct.

---

# 9. Question Difficulty Must Be Fully Admin Controlled

Each of Levels 1–10 needs its own difficulty control.

Keep the current difficulty concepts:

- Easy
- Medium
- Hard

The difficulty must affect generation behavior, not just the label.

The exact difficulty implementation should continue using the existing engine mechanisms where available.

However, the Admin must be able to choose:

```text
Level 1 = Hard
Level 2 = Easy
Level 3 = Medium
...
```

or any other combination.

Do not infer difficulty from the level number.

Do not force:

```text
L1 = Easy
L2 = Easy
L3 = Medium
L4 = Hard
```

unless that is the Admin's saved configuration.

---

# 10. Multiplication Settings for ALL Levels

Add a multiplication configuration section to every Level 1–10.

The current system already supports multiplication digit controls in the training system.

Use and extend the existing mechanism instead of creating a parallel configuration system.

At minimum expose:

- digits in first factor;
- digits in second factor;
- minimum/maximum values where supported;
- difficulty;
- question structure / term count where applicable;
- decimal-number toggle where mathematically supported.

The same multiplication configuration UI pattern must appear for Levels 1–10.

Example:

```text
Level 1
  Multiplication
  First factor digits: [1–4]
  Second factor digits: [1–3]
  Difficulty: [Easy]
  Decimal: [No]

Level 2
  ...
```

But these are examples only. The Admin must be able to change the saved values.

Do not hard-code Level 1 or Level 2 to the example values.

---

# 11. Division Settings for ALL Levels

Add a division configuration section to every Level 1–10.

Use the existing division generator and current settings as the baseline.

At minimum support:

- dividend digits;
- divisor digits;
- minimum/maximum operand where supported;
- result constraints where supported;
- difficulty;
- exact division mode when decimals are disabled;
- decimal division mode when decimals are enabled.

The same division configuration UI pattern must appear for all ten levels.

Do not keep division available only in a small subset of levels.

---

# 12. New Decimal Option

Add a new per-level decimal setting.

The Admin must be able to choose:

```text
Allow decimal numbers:
[ OFF ] / [ ON ]
```

This must be a real generator setting.

It must not be only a visual option.

## When OFF

Generation must remain integer-based.

For division, keep the existing exact-result behavior.

## When ON

The relevant game generator(s) may generate decimal values according to the configured decimal rules.

The implementation must define and validate a safe decimal representation so that:

- arithmetic equality remains exact from the platform's perspective;
- floating-point comparison errors do not create false answers;
- displayed values are formatted consistently;
- answer validation is deterministic;
- the same generated question is interpreted consistently by training, PvP and AI.

Prefer a controlled decimal representation / integer scaling strategy rather than trusting raw binary floating-point equality.

Also provide a configurable decimal precision field when needed by the existing generator architecture, for example:

```text
Decimal places: [1] [2] [3]
```

Do not force this extra control into the UI if the existing architecture can safely implement decimal precision another way; however, the engine must have a deterministic precision rule.

---

# 13. One Unified Per-Level Configuration Schema

Keep the master level configuration inside the existing:

`math_levels_json`

setting unless the current architecture demonstrates a strong reason to extend it.

The schema should evolve from 4 level objects to 10 level objects.

Use a structure conceptually similar to:

```ts
{
  "1": {
    "additionSubtraction": {
      ...
    },
    "multiplication": {
      ...
    },
    "division": {
      ...
    },
    "decimalEnabled": false,
    ...
  },
  "2": {
    ...
  },
  ...
  "10": {
    ...
  }
}
```

Do not blindly copy this exact schema if a cleaner backward-compatible schema fits the existing types better.

The important requirements are:

- all ten levels exist;
- each level is independent;
- all supported controls are stored;
- configuration survives reload;
- configuration is shared consistently by all relevant generators;
- no hidden second configuration source exists.

---

# 14. Backward Compatibility

Existing installations may still contain a 1–4 configuration.

Implement a safe migration/defaulting strategy.

When old `math_levels_json` contains only levels 1–4:

- preserve their currently saved values;
- initialize levels 5–10 from a reasonable normalized copy of the existing Level 4 configuration, or another deterministic migration rule;
- then allow the Admin to modify Levels 5–10 independently.

Do not overwrite the user's existing saved configurations for Levels 1–4.

Do not silently change Levels 1–4 to the platform defaults when migrating.

---

# 15. Admin API Changes

Update the existing Admin Levels APIs instead of creating unrelated duplicate routes.

Current APIs:

```text
GET  /api/admin/levels/state
POST /api/admin/levels/settings
```

These should now return/save the complete Level 1–10 configuration.

### `GET /api/admin/levels/state`

Must return:

- all levels 1–10;
- all addition/subtraction settings;
- all multiplication settings;
- all division settings;
- decimal settings;
- difficulty;
- enabled rules;
- any other supported per-level configuration.

### `POST /api/admin/levels/settings`

Must validate and save all ten levels atomically.

Requirements:

- validate payload shape;
- normalize values;
- reject malformed payloads;
- save the full configuration consistently;
- audit Admin changes if the current system already audits settings changes;
- preserve existing auth / Admin guards;
- preserve rate limiting.

Do not create separate endpoint families such as:

```text
/api/admin/level1
/api/admin/level2
...
```

Use the existing Levels Settings API.

---

# 16. Database / Types

Update all affected types and mirrors.

Inspect and update:

- `src/lib/db-types.ts`
- any persisted settings typing;
- any shared level configuration types;
- Firestore representation if needed;
- Supabase representation if needed;
- dual-engine synchronization typing.

The project uses dual Firestore + Supabase.

Do not update only one backend schema/type and leave the other inconsistent.

Since `math_levels_json` is a single settings value, maintain one canonical serialized configuration and ensure dual-write / synchronization behavior continues to work.

---

# 17. Training Integration

Training must resolve the configuration from the student's actual study level.

If a student has:

```text
User.level = 7
```

the training engine must use:

```text
math_levels_json["7"]
```

not:

```text
math_levels_json["4"]
```

and not a hard-coded engine level.

Update:

- training settings normalization;
- question-start configuration;
- question generation;
- answer validation where necessary.

Maintain the existing server authority:

- question generation server-side;
- answers/correctness never trusted from the client;
- score mutations remain server-side.

---

# 18. PvP Integration

PvP must resolve the same per-level configuration.

For an addition/subtraction PvP match for Level 8:

```text
Use Level 8 configuration
```

The PvP client must NOT calculate the rule configuration.

The server remains authoritative.

Do not create a separate PvP level/rule configuration.

---

# 19. AI Robot Integration

The adaptive AI system must remain compatible with the new 1–10 configuration model.

The robot must generate questions according to the same level configuration used by the student.

For example:

```text
Student Level = 6
Robot questions = Level 6 configuration
```

Do not collapse the robot to the old 1–4 model.

Keep the existing `robotPlanJson` authority and deterministic robot verdict behavior.

Do not modify the robot's server-authority contract merely to support the new level configuration.

---

# 20. Remove Old `engineLevelFor` Compression

Inspect:

`src/lib/rules-engine.ts`

and any caller relying on:

```ts
engineLevelFor()
```

The old 1–4 abstraction may be removed or rewritten.

The final architecture must support:

```text
engineLevelFor(1)  -> 1
engineLevelFor(2)  -> 2
...
engineLevelFor(10) -> 10
```

or replace the function with a direct configuration lookup.

Do not leave a hidden fallback that turns Levels 5–10 into Level 4.

---

# 21. Keep Absolute Safety Limits

"Fully open" means the Admin controls the curriculum and difficulty.

It does NOT mean the server accepts impossible or dangerous values.

Keep hard technical safety boundaries such as:

- valid integers where an integer is required;
- bounded decimal precision;
- finite numbers only;
- no Infinity / NaN;
- safe maximum question size to avoid browser/rendering failure;
- safe maximum terms;
- safe maximum digits;
- safe execution time;
- no oversized JSON payload abuse;
- no pathological generation loops.

These are implementation safety boundaries, not educational locks.

Document the difference in code comments.

---

# 22. UI Layout — Same Design Across Levels 1–10

The screenshot is the main visual reference for this modification.

The existing 1–4 cards use a compact control-card model. Preserve that visual language.

For all 10 levels:

- same card structure;
- same headings;
- same switch design;
- same slider design;
- same difficulty buttons;
- same spacing;
- same border/radius system;
- same RTL direction;
- same typography hierarchy;
- same responsive behavior.

Do not make Levels 5–10 look like a completely different page.

Do not add a separate "advanced levels" visual section that uses unrelated styling.

Prefer reusable components such as:

```text
LevelConfigCard
RuleToggle
RangeSlider
DifficultySelector
GameConfigSection
DecimalToggle
```

only if consistent with the current architecture.

---

# 23. UI Organization

Each level card should make the configuration understandable without excessive scrolling.

Use a clear structure such as:

### Level Header

```text
المستوى 1
[Save/edited status if existing]
```

### Section A — Addition & Subtraction

- problem structure;
- operands / terms;
- numeric range;
- enabled rules;
- maximum result;
- difficulty;
- decimal option.

### Section B — Multiplication

- factor digit size;
- range settings;
- difficulty;
- decimal option if supported.

### Section C — Division

- dividend digits;
- divisor digits;
- result behavior;
- difficulty;
- decimal option.

Do not duplicate labels unnecessarily.

Use the existing Arabic terminology from the platform wherever possible.

---

# 24. No Automatic UI Locking Based on Rules

If a rule is selected, do not disable another rule merely because of a curriculum assumption.

For example, this must be allowed:

```text
Direct = ON
Comp5 = ON
Comp10 = ON
Comp50 = ON
Comp100 = ON
Tens = ON
```

within one level.

If a mathematical combination produces no valid candidates for a specific generated question, the generator should select another enabled valid candidate instead of changing the Admin configuration.

---

# 25. Do Not Couple Difficulty to Question Size Automatically

Avoid hidden code such as:

```ts
if (difficulty === "easy") maxOperand = 9;
```

unless that relationship is explicitly part of the Admin's saved configuration model.

Difficulty can influence generation weighting/selection as it currently does, but numeric ranges and structure should remain directly configurable.

The Admin must be able to create combinations such as:

```text
Easy + larger operands
Hard + smaller operands
Medium + many terms
Hard + direct only
Easy + all rules
```

where technically valid.

---

# 26. Decimal Handling Across Games

Make decimal configuration explicit per level and per game where the existing architecture supports it.

Do not introduce a generic global decimal flag that unintentionally changes every game.

At minimum make it possible to configure:

```text
Level X
  Addition/Subtraction decimal = ON/OFF
  Multiplication decimal = ON/OFF
  Division decimal = ON/OFF
```

If the product architecture intentionally treats decimals as one shared per-level setting, keep the implementation unified, but the Admin must still have a clear way to control whether decimals are used.

---

# 27. Student-Facing Behavior

The student should continue to get the configured experience automatically from their study level.

Example:

```text
Student.level = 5
```

The student enters Addition/Subtraction training.

The backend resolves Level 5 configuration and generates questions using it.

The student should not see internal admin controls.

The existing student training UX remains intact unless a change is required to consume the new configuration correctly.

Do not reintroduce a student-side manual math-level selector merely because the Admin now has ten configurable levels.

---

# 28. Preserve Existing Training Settings

Do not break unrelated training controls such as:

- `displayMode`
- `displayTime`
- `disappearTime`
- batch size;
- resume;
- scoring;
- session lifecycle.

The new level configuration should integrate cleanly with the existing training settings.

---

# 29. Validation / Generator Correctness

Every generated question must continue to pass the existing validation layer.

Preserve the current validation guarantees:

- mathematical equality;
- result constraints;
- operand constraints;
- non-negative behavior where required by current rules;
- rule classification correctness;
- configured bounds;
- decimal correctness when decimals are enabled.

Do not weaken validation to make generation easier.

---

# 30. Admin Save Behavior

The page should make it obvious that each level can be edited independently.

After saving:

- the saved values remain after page reload;
- GET state exactly reflects POST state after normalization;
- switching between levels does not overwrite unsaved changes unexpectedly;
- changing Level 5 does not modify Level 4 or Level 6;
- changing multiplication does not overwrite division;
- changing decimal settings does not overwrite rule toggles.

Prefer a predictable save model consistent with the current Admin UX.

---

# 31. Preserve Current Arabic RTL Admin Design

All visible Admin UI must remain:

- Arabic;
- RTL;
- Cairo font;
- compatible with light/dark mode;
- consistent with the current Calm Educational Play design system;
- free from unrelated indigo/blue-default redesign.

Do not replace the current design language.

---

# 32. Responsive Behavior for 10 Level Cards

The 10-level page must work well on:

- desktop;
- laptop;
- tablet;
- mobile.

Reuse the responsive improvements from the previous modification work.

For tablet widths in particular:

- maintain proportional cards;
- prevent controls from becoming cramped;
- allow sections to stack when required;
- preserve readable labels;
- avoid excessive empty side margins;
- avoid horizontal scrolling.

Do not create a separate tablet design unless necessary.

---

# 33. Recommended Component Reuse

Before writing new components, inspect the existing Levels UI and reuse/refactor it.

The goal is:

```text
One reusable level card
        ↓
rendered 10 times
```

not:

```text
Level 1 component
Level 2 component
...
Level 10 component
```

Avoid ten copies of the same UI implementation.

---

# 34. Exact Functional Examples

After the change, the Admin must be able to configure:

### Example A

Level 1:

```text
Difficulty: Hard
Tens: ON
Direct: ON
Comp5: ON
Comp10: ON
Comp50: ON
Comp100: OFF
Multiplication: 3 × 2 digits
Division: dividend 3 digits / divisor 1 digit
Decimals: ON
```

### Example B

Level 5:

```text
Difficulty: Easy
Tens: OFF
Direct: ON
Comp5: OFF
Comp10: OFF
Comp50: OFF
Comp100: OFF
Multiplication: 1 × 1 digit
Division: simple exact division
Decimals: OFF
```

### Example C

Level 10:

```text
Difficulty: Medium
Tens: ON
All rules: ON
Multiplication: large configured factors
Division: larger configured dividend/divisor
Decimals: ON
```

These are only validation examples.

Do not hard-code these values as defaults.

---

# 35. API / Config Integrity

Make sure the same saved configuration is used consistently by:

```text
Admin Levels UI
        ↓
math_levels_json
        ↓
Rules Engine / game generators
        ↓
Training
PvP
AI Robot
```

There must not be one configuration shown in Admin while the generator still uses another hidden configuration.

---

# 36. Logging / Audit

The existing Admin settings mutation should continue to use the platform's existing audit mechanisms.

Record:

- Admin action;
- setting/configuration change;
- time;
- relevant identifiers;

but never:

- passwords;
- session tokens;
- secrets;
- sensitive credential material.

Do not dump the entire secret-bearing runtime environment into logs.

---

# 37. Testing Requirements

Add regression tests for all ten levels.

At minimum verify:

### Level resolution

```text
1 -> 1
2 -> 2
3 -> 3
4 -> 4
5 -> 5
6 -> 6
7 -> 7
8 -> 8
9 -> 9
10 -> 10
```

### Configuration independence

Change Level 5 and verify:

- Level 4 unchanged;
- Level 6 unchanged.

Change multiplication Level 8 and verify:

- addition/subtraction Level 8 unchanged;
- division Level 8 unchanged.

### Rule independence

Enable a rule in Level 1 and verify it does not automatically appear in Level 2.

### No level locks

Verify all supported rules can be toggled for all ten levels.

### Decimal OFF

Verify integer-only behavior.

### Decimal ON

Verify valid decimal generation and deterministic answer checking.

### Training

Verify each study level consumes its own saved config.

### PvP

Verify the match uses the configured level.

### AI

Verify the robot uses the configured level and existing robot-plan authority.

---

# 38. Required Regression Checks

After implementation, run the standard project checks:

```text
bun run lint
```

Run the relevant existing regression scripts, including:

```text
scripts/comprehensive-test.sh
scripts/new-mods-test.sh
```

Also run targeted tests for:

- levels state/settings APIs;
- Rules Engine;
- training;
- multiplication;
- division;
- decimal behavior;
- PvP;
- AI robot.

Inspect `dev.log` for runtime errors.

Run browser E2E checks through the normal project workflow.

Do NOT run `bun run build` because the project rules explicitly prohibit that workflow.

---

# 39. Manual Browser Verification

Open the Admin Levels page and verify all ten cards are visible/accessible:

```text
/admin/levels
```

Verify:

- Level 1 through Level 10 are present;
- Level 5–10 visually match Level 1–4;
- no artificial lock icon remains on configurable rules;
- all controls are interactive;
- save works;
- reload preserves values.

Then verify at least:

```text
Student Level 1
Student Level 5
Student Level 10
```

and confirm each generates from the correct configuration.

---

# 40. Do NOT

Do not:

1. create a second addition/subtraction Rules Engine;
2. keep study levels 5–10 mapped to Level 4;
3. keep the old curriculum restriction matrix;
4. lock rules because a level is considered easy/hard;
5. silently override Admin-selected settings;
6. expose correctness/answers to the client as authoritative data;
7. create duplicate API families for each level;
8. create ten duplicated React components;
9. break the dual Firestore/Supabase configuration behavior;
10. overwrite existing Level 1–4 user configuration during migration;
11. change unrelated student UX unnecessarily;
12. introduce a new color/design system;
13. use raw floating-point equality for decimal answer validation;
14. bypass existing server-side validation/security;
15. weaken existing idempotency, authority, or audit rules.

---

# 41. Documentation / Worklog

Before finishing:

1. update the relevant platform documentation to reflect that Levels 1–10 are now independently configurable;
2. update the architecture/configuration description for `math_levels_json`;
3. document the new multiplication/division per-level configuration;
4. document decimal support;
5. append a clear record to:

`/home/z/my-project/worklog.md`

Include:

- what changed;
- files changed;
- schema/API changes;
- migration behavior;
- test results;
- any remaining limitations.

---

# 42. Final Acceptance Criteria

The task is complete only when all of the following are true:

- [ ] Admin can configure Levels 1–10 independently.
- [ ] Level 5–10 use the same UI structure and styling pattern as Levels 1–4.
- [ ] No educational level-based rule locks remain.
- [ ] The Admin can enable/disable all supported rules on every level.
- [ ] The Admin controls problem structure and size per level.
- [ ] The Admin controls difficulty per level.
- [ ] Multiplication settings exist for every level.
- [ ] Division settings exist for every level.
- [ ] Decimal mode can be enabled/disabled.
- [ ] Decimal validation is deterministic and safe.
- [ ] Level 1 no longer has hard-coded restrictions.
- [ ] Level 10 no longer inherits Level 4 settings.
- [ ] Study level 1–10 maps directly to its own configuration.
- [ ] Training uses the correct level configuration.
- [ ] PvP uses the correct level configuration.
- [ ] AI robot uses the correct level configuration.
- [ ] Existing server authority remains intact.
- [ ] Existing security/rate limits/audit remain intact.
- [ ] Existing Level 1–4 saved settings are preserved during migration.
- [ ] The Admin page saves and reloads all 10 levels correctly.
- [ ] Desktop/tablet/mobile layouts remain usable.
- [ ] Lint passes.
- [ ] Relevant regression tests pass.
- [ ] Browser verification passes.
- [ ] `worklog.md` is updated.

---

## Final Implementation Principle

The platform should no longer decide:

> "This level is allowed to use these rules."

Instead, the platform should work as:

> **"The Admin defines exactly how each level works, and the generators execute that configuration safely and consistently."**

Levels 1–10 are therefore **ten independently configurable profiles**, using the same reusable UI pattern and the same authoritative generation architecture.
