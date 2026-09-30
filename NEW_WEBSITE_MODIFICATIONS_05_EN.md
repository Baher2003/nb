# Website Modification Specification 05
## Registration Approval, Automatic Math Level Selection, and Multiplication/Division Page Rebuild

> **Standalone implementation task.**
>
> Use `PLATFORM_KNOWLEDGE_BASE.md` as the project source of truth before making any change.
>
> **Critical:** Do not guess the current architecture. Inspect the real code first, use the exact existing file paths/routes/settings, and preserve all previously working functionality.

---

# 1. Mandatory Pre-Implementation Analysis

Before editing code:

1. Read `/home/z/my-project/worklog.md` first.
2. Read the current `PLATFORM_KNOWLEDGE_BASE.md`.
3. Inspect the actual registration flow.
4. Inspect the current level picker/training setup flow.
5. Inspect the current multiplication and division implementation.
6. Identify the exact files/components/services used by those pages.
7. Inspect the current API calls and server-side generators.
8. Confirm whether multiplication/division are using the shared `game-view` and `game-engine.ts` paths described in the knowledge base.
9. Identify any existing duplicated or obsolete code before changing it.

Do not create replacement files until the existing implementation has been understood.

---

# 2. Registration: Default Student Level Must Be Level 1

The default study level for a newly created student account must be:

```text
Level 1
```

### Required behavior

When the student opens the registration form:

- The default selected level is **1**.
- If the level field is shown, Level 1 must be preselected.
- If the application can safely omit manual level selection in a future UX, the server must still default the account level to `1`.

### Security requirement

Do not trust the frontend default.

The backend registration flow must explicitly apply:

```text
default level = 1
```

when no valid level is supplied.

Relevant existing route:

```text
POST /api/auth/register/complete
```

The valid study-level range remains:

```text
1–10
```

as documented in the platform knowledge base.

---

# 3. New Student Accounts Must Require Admin Approval

Current platform behavior is:

```text
auto_approve = 1
```

which causes new accounts to be approved immediately, assigned validity, and logged in.

This must change.

## Required new default

Change:

```text
auto_approve = 1
```

to:

```text
auto_approve = 0
```

The system should therefore default to **manual Admin approval**.

---

# 4. Required Registration Flow After the Change

The desired flow is:

```text
Student registers
        ↓
Account created
        ↓
Default level = 1
        ↓
Status = pending
        ↓
Student is NOT treated as approved
        ↓
Admin reviews/approves account
        ↓
Status = approved
        ↓
Validity period is assigned according to the existing approval rules
        ↓
Student can log in/use the platform
```

## Important

The new pending account must not gain access to student functionality before approval.

Do not rely only on hiding the UI.

The existing server-side authentication behavior must continue to reject:

```text
status = pending
```

through:

```text
POST /api/auth/login
```

which currently returns `403` for pending accounts.

Keep that server-side protection intact.

---

# 5. Admin Approval Must Remain the Activation Authority

The Admin must remain able to activate/approve the account using the existing admin student-management flow.

Relevant existing route:

```text
POST /api/admin/student/update
```

The current system already supports:

- Status changes.
- Validity end changes.
- Smart activation.
- `+30 days` activation behavior.

Do not remove this functionality.

The change is to make **Admin approval mandatory by default**, not to remove the existing Admin controls.

---

# 6. Registration UI Feedback

After successful registration of a pending account, show a clear Arabic RTL message explaining that the account was created and is awaiting Admin approval.

Example:

```text
تم إنشاء الحساب بنجاح، وحسابك في انتظار موافقة الإدارة قبل التفعيل.
```

Do not tell the student that the account is active when it is still pending.

The exact wording can be polished, but it must be Arabic and Egyptian-friendly.

---

# 7. Training — Do Not Show Four Math Levels to the Student in Normal Addition/Subtraction

The current training system exposes a level picker for addition/subtraction.

This is no longer desired for normal student training.

The student should **not see four math-level options** such as:

```text
Level 1
Level 2
Level 3
Level 4
```

when starting normal addition/subtraction training.

---

# 8. Addition/Subtraction Level Must Be Determined Automatically

The math level used by the Rules Engine must be derived from the student's **study level**:

```text
User.level
   ↓
engineLevelFor()
   ↓
Math engine level
```

The existing mapping is:

```text
Study Level 1 → Engine Level 1
Study Level 2 → Engine Level 2
Study Level 3 → Engine Level 3
Study Levels 4–10 → Engine Level 4
```

This existing mapping must become the student-facing source of truth for normal addition/subtraction training.

Do not require the student to manually select the engine level.

---

# 9. Do Not Trust a Student-Submitted Math Level

For normal addition/subtraction training:

- The frontend must not be allowed to override the student's Rules Engine level.
- The server should derive the engine level from the authenticated `User.level`.
- A manipulated request attempting to send another level must be ignored/rejected.

### Example

If:

```text
User.level = 2
```

then normal addition/subtraction must use:

```text
engine level = 2
```

even if the browser submits:

```json
{
  "level": 4
}
```

The backend must not accept that override.

---

# 10. Keep Admin Control Over Math Rules

Removing the student's level selector does **not** remove Admin control.

The Admin can still configure:

```text
math_levels_json
```

and the existing `/api/admin/levels/settings` system.

The Admin configuration remains the source for what each engine level means.

The student simply cannot choose a different engine level manually.

---

# 11. Preserve the Single Rules Engine

Do not create a new addition/subtraction generator.

Keep:

```text
src/lib/rules-engine.ts
```

as the single source of truth for:

- Normal training.
- Robot challenge.
- Friend/PVP challenge.

The student-facing training flow should simply select the correct rules configuration through the authenticated study level.

This must not create a second rules engine.

---

# 12. Important Distinction Between Study Level and Engine Level

Maintain the distinction:

### Study level

```text
User.level
```

Range:

```text
1–10
```

This represents the student's educational/study level.

### Math engine level

```text
engineLevelFor(User.level)
```

Range:

```text
1–4
```

This determines the active Rules Engine difficulty.

Do not rename or merge these two concepts in a way that breaks the existing data model.

---

# 13. Update Training Setup UX

For normal addition/subtraction:

Remove the visible math-level selector.

Instead show a simple information label where useful, for example:

```text
مستواك الحالي: المستوى 2
```

or a more human-friendly equivalent.

Do not present four selectable engine levels.

The student should feel that the platform automatically knows which training level belongs to them.

---

# 14. Handle Existing Active Sessions Safely

Do not break currently active/resumable training sessions.

The platform already supports:

```text
resumeMode: "auto"
```

and a:

```text
3-hour
```

resume window.

If a user already has an active addition/subtraction session:

- Continue/resume the same session according to the existing rules.
- Do not silently switch its question set halfway through.
- Do not regenerate the session merely because the UI no longer shows a level picker.

New sessions should use the authenticated student's current study-level mapping.

---

# 15. Multiplication Page — Full Technical and UX Review

The multiplication page currently needs a serious modernization because its implementation/code has become poor and needs restructuring.

Do **not** assume the fix is just visual.

Perform a full audit of:

- UI structure.
- Component structure.
- State management.
- Event handling.
- Question progression.
- Timer behavior.
- Keyboard/input behavior.
- Responsive behavior.
- Loading states.
- Error states.
- Session resume.
- Duplicate logic.
- Re-render behavior.
- API calls.
- Data formatting.
- Accessibility.
- Mobile usability.

---

# 16. Preserve Multiplication Business Rules

The existing multiplication training configuration is:

```text
digits1: 1–4
digits2: 1–3
```

Do not change these ranges unless explicitly requested later.

Multiplication questions remain server-side.

The existing math generator lives in:

```text
src/lib/game-engine.ts
```

Do not move mathematical authority to the frontend.

---

# 17. Multiplication UI Requirements

Rebuild the multiplication training page so it feels like a purpose-built mental-math training experience.

Prioritize:

```text
Training title / level context
        ↓
Current multiplication problem
        ↓
Answer input / keypad
        ↓
Progress
        ↓
Timer
```

The question must be visually dominant.

Avoid:

- Huge empty areas.
- Multiple redundant cards.
- Unnecessary dashboards during answering.
- Excessive gradients.
- Excessive glass effects.
- Overly decorative elements.
- Information unrelated to the current task.

---

# 18. Multiplication Responsive Requirements

The page must work correctly on:

- Small phones.
- Large phones.
- Tablets.
- Laptops.
- Desktop.
- Short-height mobile screens.

The current application already uses:

```text
challenge-stage
useCompactViewport
```

for immersive training/challenge layouts.

Reuse the existing responsive infrastructure where appropriate instead of inventing another viewport system.

---

# 19. Multiplication Question Transition

The multiplication page must respect the previously required exactly-once question transition behavior.

After a valid answer:

```text
Question N
→ submit
→ save/grade
→ advance once
→ Question N+1
```

Never:

```text
Question N
→ Question N+1
→ Question N again
```

Audit the shared state/event logic so the fix is not duplicated only inside multiplication.

---

# 20. Multiplication Numeric Input

Preserve the existing mobile-friendly numeric input strategy.

The existing shared keypad is:

```text
numeric-keypad
```

with:

```text
dir="ltr"
```

and phone-style ordering:

```text
1 2 3
4 5 6
7 8 9
  0
```

Do not reintroduce RTL-reversed numeric ordering.

---

# 21. Division Page — Full Technical and UX Review

The division page should receive the same level of review and modernization.

Audit:

- UI structure.
- Component structure.
- State handling.
- Timer.
- Input.
- Question progression.
- API requests.
- Loading.
- Error states.
- Responsive behavior.
- Resume behavior.
- Accessibility.
- Duplicate or obsolete code.

Do not patch only the visible CSS.

---

# 22. Preserve Division Business Rules

The current division configuration is:

```text
dividendDigits: 2–4
divisorDigits: 1–2
```

Division generation must continue to use **exact/integer results** according to the existing `game-engine.ts` behavior.

Do not introduce approximate/random division answers.

Mathematical generation remains server-side.

---

# 23. Division UI Requirements

Use the same core visual language as multiplication while allowing the layout to reflect division-specific content.

Prioritize:

```text
Division problem
↓
Answer area
↓
Numeric keypad/input
↓
Progress
↓
Timer
```

The two pages should feel like the same product, but not like duplicated screens with only the title changed.

---

# 24. Multiplication vs Division — Shared Architecture, Distinct UX

Use reusable components for shared mechanics where appropriate:

```text
Question display
Answer input
Numeric keypad
Timer
Progress
Training shell
Loading/Error states
```

But keep game-specific presentation where it improves usability.

Do not duplicate entire page implementations unnecessarily.

---

# 25. Remove Obsolete / Broken Code Carefully

During the multiplication/division modernization:

- Identify components that are no longer used.
- Identify duplicate event handlers.
- Identify stale state.
- Identify obsolete CSS.
- Identify dead API calls.
- Identify duplicated calculation logic.

Remove code only after verifying that it is unused.

Do not delete shared infrastructure simply because it looks unused from one page.

---

# 26. Keep Server Authority

For multiplication and division:

- Questions are generated server-side.
- Correct answers are not exposed to the client before grading.
- Scoring remains server-side.
- Final training results remain server-side.
- The existing training transaction/idempotency behavior must remain intact.

Existing routes include:

```text
POST /api/student/training/start
POST /api/student/training/next
POST /api/student/training/answer
POST /api/student/training/finish
```

Do not alter their contracts unnecessarily.

---

# 27. Preserve Training Session Persistence

The existing platform supports:

```text
resumeMode: "auto"
```

with a:

```text
3-hour
```

resume window.

Multiplication and division must continue to preserve:

- Session ID.
- Current question.
- Answered rows.
- Settings.
- Progress.
- Score.
- Completion state.

Refresh must not silently create a new training session.

---

# 28. Admin vs Student Controls

For this change:

### Student

- Cannot manually choose the addition/subtraction math engine level.
- Uses the engine level derived from `User.level`.

### Admin

Can still control:

- `math_levels_json`.
- Level rules.
- Floors.
- Difficulty.
- Allowed complements.
- Other existing Rules Engine configuration.

The Admin configuration remains centralized.

---

# 29. Regression Requirements

Do not break:

- Registration.
- Admin approval.
- Login.
- Device binding.
- Session handling.
- Training.
- Addition.
- Subtraction.
- Multiplication.
- Division.
- Abacus.
- Robot challenge.
- PVP.
- Points.
- Reports.
- Existing routing.

Do not reintroduce:

- Auto-approved accounts by default.
- Student-selectable addition/subtraction engine levels.
- Duplicate question generators.
- Client-authoritative answers.
- Duplicate training sessions.

---

# 30. Required Tests

## Registration

Test:

```text
New registration
→ default Level 1
→ status pending
→ no student access before approval
→ Admin approves
→ account becomes approved
→ validity rules apply
→ login works
```

Test both when the frontend sends no level and when a malformed level is attempted.

---

## Addition/Subtraction

For a user with:

```text
User.level = 1
```

verify the session uses:

```text
engine level 1
```

For:

```text
User.level = 2
```

verify:

```text
engine level 2
```

For:

```text
User.level = 3
```

verify:

```text
engine level 3
```

For:

```text
User.level = 4..10
```

verify:

```text
engine level 4
```

The UI must not show four selectable engine levels.

---

## Multiplication

Test:

- Question generation.
- Answer submission.
- Exactly-once advance.
- Numeric keypad.
- Timer.
- Progress.
- Refresh/resume.
- Mobile.
- Desktop.
- Short-height screens.

---

## Division

Test:

- Exact/integer question generation.
- Answer submission.
- Exactly-once advance.
- Numeric keypad.
- Timer.
- Progress.
- Refresh/resume.
- Mobile.
- Desktop.
- Short-height screens.

---

# 31. Required Agent Report

Before implementation, provide:

### Registration

- Current `auto_approve` flow.
- Current registration path.
- Exact files to modify.
- How pending users are currently handled.

### Addition/Subtraction

- Current level picker path.
- Current way `level` enters training settings.
- Where `engineLevelFor` is called.
- How the backend currently decides the Rules Engine level.

### Multiplication/Division

- Exact current component/view paths.
- Shared vs game-specific components.
- Current state flow.
- Current API flow.
- Any duplicated/obsolete code.
- Root cause of the current code quality/problem.

Do not begin a large rewrite without first identifying the real implementation.

---

# 32. Verification Requirements

After changes:

1. Run `bun run lint`.
2. Check the tail of `dev.log`.
3. Run the relevant regression suite.
4. Use browser E2E for:
   - Registration → pending → admin approval → login.
   - Addition/subtraction start.
   - Multiplication training.
   - Division training.
5. Verify session resume after refresh.
6. Verify the addition/subtraction engine level is derived server-side from the authenticated user's study level.
7. Verify no secret values are added to documentation or source.
8. If database schema/settings change, verify dual-engine parity/drift as required by the platform knowledge base.

---

# 33. Final Goal

The final product should behave as follows:

```text
New Student
    ↓
Default Study Level = 1
    ↓
Account = Pending
    ↓
Admin Approval Required
    ↓
Approved Account
    ↓
Student Training
```

For normal addition/subtraction:

```text
Student Study Level
        ↓
engineLevelFor()
        ↓
Automatic Rules Engine Level
```

No manual four-level selector should appear to the student.

For multiplication and division:

```text
Clean modern training UI
+
Reliable state management
+
Server-authoritative questions
+
Exactly-once question progression
+
Responsive mobile/desktop layout
+
Session persistence
```

The final implementation should feel like a coherent educational product and should simplify the student experience while preserving the existing backend architecture and business rules.
