# Website Modification Specification 03 — Master Document

> **Master merged specification**
>
> This document combines the implementation requirements and the deep UI/UX, color, typography, accessibility, and design research for the current project.
>
> **Critical rule:** Preserve all existing functionality and previously completed fixes. Do not introduce regressions in routing, browser history, refresh behavior, training sessions, PVP sessions, scoring, timers, authentication, reports, or responsive behavior.

---

# Part I — Implementation Specification

# Website Modification Specification 03
## Question Transition Bug, Human-Centered UI Redesign, Calm Child-Friendly Visual System, and Full Security Audit

> **Document type:** New implementation specification  
> **Language:** English  
> **Scope:** Apply these changes to the current working project.  
> **Critical requirement:** Preserve all existing functionality and previously completed fixes. Do not introduce regressions in routing, browser history, refresh behavior, training sessions, PVP sessions, scoring, timers, authentication, reports, or responsive behavior.

---

# 1. Critical Bug — Question Changes to the Next Question, Then Jumps Back

## 1.1 Current bug

There is a recurring issue across the multi-question experiences:

```text
Question N
   ↓
User submits answer
   ↓
Question N+1 appears
   ↓
UI unexpectedly returns to Question N
```

This is not acceptable.

The same issue can appear in different training/challenge modes, so this must be investigated as a **shared state/navigation/session bug**, not patched separately on one page.

## 1.2 Required behavior

Once an answer has been accepted:

```text
Current Question
      ↓
Submit
      ↓
Validate answer
      ↓
Persist answer/progress
      ↓
Advance exactly once
      ↓
Show next question
```

After Question N+1 is shown, it must remain active.

There must be no automatic return to Question N unless the user explicitly uses an intentional question-review feature.

## 1.3 Apply the fix everywhere

Audit and fix the issue in all relevant flows:

- Addition.
- Subtraction.
- Multiplication.
- Division.
- Normal training.
- Robot challenge.
- Friend challenge / PVP.
- Any multi-question challenge.
- Any shared question/session component.

## 1.4 Root-cause investigation

Do not solve this by simply adding another timeout, another state update, or another navigation call.

Audit for:

- Duplicate click handlers.
- Duplicate form-submit handlers.
- Double submission.
- Multiple event listeners.
- Duplicate API requests.
- Race conditions.
- Stale frontend state.
- Stale React/Vue/etc. closures if applicable.
- Effects/hooks that restore old state.
- Session restoration firing after the next question is loaded.
- Timer callbacks writing stale question state.
- WebSocket/SSE events overwriting current progress.
- Responses arriving out of order.
- Browser-history manipulation.
- Component remounts caused by unstable keys.
- Cached state replacing the newest state.
- Multiple sources of truth for `currentQuestion`.
- Multiple sources of truth for `progress`.
- Backend responses returning an older session snapshot.

## 1.5 Use one authoritative question-transition mechanism

The application should have one clear source of truth for question progression.

The transition must be:

- Atomic where possible.
- Idempotent.
- Race-safe.
- Executed exactly once per accepted answer.
- Protected from duplicate submission.
- Based on the newest persisted session state.

Use an appropriate mechanism such as:

- Question ID.
- Question sequence number.
- Session version.
- Transition token.
- Request ID.
- Server-side version check.

The exact implementation depends on the current architecture.

## 1.6 Acceptance test

For every relevant mode:

```text
Question 1
→ submit
→ Question 2
→ wait
→ Question 2 remains visible
```

Then:

```text
Question 2
→ submit
→ Question 3
→ wait
→ Question 3 remains visible
```

There must be no automatic jump back to the previous question.

---

# 2. Full UI/UX Redesign — Human-Centered, Not Template-Like

The current pages, especially the **competitions/challenges and training pages**, need a substantial visual and structural redesign.

The objective is not to create another generic AI-generated dashboard.

The objective is to make the product feel like a **real, intentionally designed educational product** built for children, parents, trainers, and administrators.

The UI should communicate:

- Trust.
- Warmth.
- Simplicity.
- Energy without visual overload.
- Professional quality.
- Child-friendly interaction.
- Clear hierarchy.
- Human-centered design.

## 2.1 Design research requirement

Before implementing the redesign, inspect current high-quality product design references and extract the underlying design principles rather than copying visual identities.

Useful references include:

### Apple Human Interface Guidelines

Use Apple HIG as a reference for:

- Adaptable layouts.
- Safe areas.
- Typography.
- Responsive behavior.
- Orientation changes.
- RTL support.
- Clear hierarchy.
- System-consistent interaction patterns.

Apple's current layout guidance emphasizes adapting layouts to actual available space, display size, orientation, text size, and RTL/LTR behavior rather than relying on one fixed device layout.

### Vercel Dashboard

Study the current Vercel dashboard navigation patterns for:

- Clear sidebar structure.
- Consistent navigation.
- Prioritization of common workflows.
- Mobile navigation.
- Resizable/collapsible navigation.

Vercel's 2026 dashboard redesign moved navigation into a resizable sidebar and added mobile navigation optimized for one-handed use.

### Linear

Study Linear's current interface approach for:

- Calm visual hierarchy.
- Consistent headers.
- Consistent navigation.
- Reduced visual noise.
- Strong focus on the current task.
- Deliberate information density.

Linear's March 2026 UI refresh focused on a calmer, more consistent interface, clearer hierarchy, and allowing the main content to visually dominate over navigation.

### Stripe Dashboard

Study Stripe's dashboard/reporting approach for:

- Clear operational navigation.
- Financial/transaction information hierarchy.
- Filters.
- Reporting.
- Data tables.
- Separate operational pages from overview dashboards.

Stripe separates quick overview information from detailed transaction/reporting workflows and supports customizable charts and exports.

## 2.2 Do not copy

Do not copy:

- Branding.
- Logos.
- Exact page structures.
- Proprietary illustrations.
- Exact color systems.
- Exact component designs.

Use the references to learn design principles and create an original system for this project.

---

# 3. Human-Centered Visual Direction

Avoid the feeling that the UI was generated from a generic AI dashboard prompt.

The design should have:

- Deliberate spacing.
- Natural visual rhythm.
- Strong alignment.
- Clear information hierarchy.
- Practical component sizing.
- Consistent typography.
- Purposeful colors.
- Simple interactions.
- Human-friendly empty states.
- Human-friendly error states.
- Meaningful feedback.
- Visual consistency between pages.

## Avoid excessive use of:

- Glassmorphism.
- Neon gradients.
- Excessive shadows.
- Excessively rounded containers.
- Glow effects.
- Decorative blobs.
- Random floating cards.
- Too many badges.
- Too many icons.
- Every section being a separate card.
- Large decorative areas that provide no functional value.
- Visually noisy KPI grids.

Every visual element should have a purpose.

---

# 4. Calm, Comfortable, Child-Friendly Color System

Redesign the color system to use **soft, calm, comfortable colors** that are pleasant for children and do not create visual fatigue.

The goal is:

```text
Calm + Friendly + Bright enough + Comfortable + Professional
```

Not:

```text
Dark neon + high saturation + excessive contrast everywhere
```

## 4.1 Suggested direction

Use a restrained palette with:

- Soft blue.
- Soft teal.
- Gentle green.
- Warm yellow/orange accents.
- Soft purple where appropriate.
- Neutral backgrounds.
- Comfortable text colors.

Avoid using highly saturated colors across large screen areas.

## 4.2 Color hierarchy

### Primary
Used for:

- Main actions.
- Important navigation state.
- Primary buttons.

### Secondary
Used for:

- Supporting actions.
- Secondary sections.

### Success
Used for:

- Completed actions.
- Positive progress.
- Successful results.

### Warning
Used sparingly for:

- Attention.
- Caution.
- Time-sensitive information.

### Error
Used only when the user needs to understand something went wrong.

### Neutral
Used for most of the interface.

The brand color should not overwhelm the entire page. Use brand color intentionally rather than applying it to every control.

## 4.3 Child-friendly does not mean childish

The interface should appeal to children without looking like a toy template.

Use:

- Soft rounded shapes where appropriate.
- Friendly iconography.
- Small playful accents.
- Positive micro-interactions.
- Encouraging progress visualization.

But maintain:

- Professional spacing.
- Clean typography.
- Mature information architecture.
- Clear usability.

---

# 5. Redesign the Training Pages

Training pages should be redesigned as **focused learning environments**.

## Primary priority

```text
Question
↓
Answer Area
↓
Numeric Keyboard / Input
↓
Progress / Timer
```

Everything else should remain secondary.

## The screen should clearly communicate:

- Current level.
- Current question.
- Current progress.
- Answer input.
- Numeric keyboard.
- Timer when applicable.
- Score/progress without distracting from the task.

## Avoid

- Large decorative headers.
- Excessive cards.
- Unnecessary banners.
- Excessive statistics during active solving.
- Navigation elements that compete with the question.

---

# 6. Redesign Competition / Challenge Pages

Challenge pages should feel like a **game/competition interface**, but still remain calm and easy to use.

They should communicate:

- This is a challenge.
- This is the current task.
- Here is the progress.
- Here is the time.
- Here is the answer interface.

## Layout priority

```text
Challenge Status
        ↓
Current Question
        ↓
Answer/Input
        ↓
Keyboard
        ↓
Progress / Timer
```

Do not overload the active challenge screen with:

- Opponent answer history.
- Detailed accuracy.
- Large dashboards.
- Unnecessary statistics.
- Large decorative content.

The user's attention should remain on solving the current problem.

---

# 7. Competition Result Design

After a competition is complete, the experience can become more visual.

Show:

- Final score.
- Accuracy.
- Time.
- Correct answers.
- Wrong answers.
- Progress.
- Final status.
- Answer review when appropriate.

Use a clear hierarchy:

```text
Result
↓
Main Score
↓
Key Statistics
↓
Performance Visualization
↓
Detailed Review
```

Do not overload the result screen with unnecessary data.

---

# 8. Responsive and Real-World Layout Review

Perform a complete review of the UI at actual available sizes rather than designing for one fixed device.

Test:

- Small phones.
- Large phones.
- Tablets.
- Laptops.
- Desktop.
- Large desktop.
- Short-height screens.
- Tall screens.
- Narrow landscape windows.
- Wide windows.

Consider:

- Width.
- Height.
- Safe areas.
- Orientation.
- Dynamic text size.
- RTL/LTR.
- Long Arabic text.
- Long usernames.
- Large numbers.
- Long mathematical expressions.

---

# 9. Admin UI Redesign

The Admin panel should be redesigned separately from the student interface.

Admin users need **clarity, density, speed, and control**.

Use a structure such as:

```text
Admin
├── Overview
├── Users
├── Trainers
├── Students
├── Challenges
├── Training
├── Points
├── Transactions
├── Reports
├── Notifications
├── Settings
└── Security / Audit
```

Use tables and filters where they provide better operational clarity than cards.

Use dashboards only for information that benefits from summary visualization.

---

# 10. Full Operations and Admin Audit

Perform a complete review of all operations and administrative functionality.

Audit:

- Admin authentication.
- Role permissions.
- Authorization.
- User management.
- Trainer management.
- Student management.
- Account status changes.
- Password changes.
- Force logout.
- Session management.
- Points.
- Wallet/balance logic if applicable.
- Money transfers.
- Deposits/credits.
- Withdrawals/debits.
- Refunds/reversals.
- Rewards/redemptions.
- Subscription/payment operations.
- Reports.
- Exports.
- Notifications.
- System settings.
- Data modifications.
- Audit logs.
- Sensitive API endpoints.
- Background jobs/cron operations if applicable.

The audit must cover both frontend and backend.

---

# 11. Security Principle — Never Trust the Frontend

Do not treat frontend restrictions as security.

Hiding a button is not authorization.

Sensitive actions must be validated server-side.

Examples:

```text
Admin-only action
→ server checks admin permission

Point adjustment
→ server validates role + amount + target user

Money transfer
→ server validates sender + receiver + balance + limits + transaction state

Challenge result
→ server validates match/session identity
```

OWASP's API Security guidance highlights object-level authorization, authentication, function-level authorization, unrestricted resource consumption, and sensitive business-flow abuse as major API risks.

---

# 12. Authorization and Object Ownership

Audit every endpoint that receives IDs such as:

```text
user_id
student_id
trainer_id
match_id
session_id
transaction_id
wallet_id
points_transaction_id
report_id
```

For every request, validate that the authenticated user is actually allowed to access or modify that object.

Do not rely on IDs being hidden or difficult to guess.

---

# 13. Admin Function-Level Authorization

Create clear server-side permissions for privileged operations.

Examples:

```text
VIEW_USERS
EDIT_USERS
DEACTIVATE_USERS
MANAGE_TRAINERS
MANAGE_LEVELS
MANAGE_TRAINING_RULES
VIEW_TRANSACTIONS
ADJUST_POINTS
APPROVE_TRANSFER
REVERSE_TRANSACTION
EXPORT_SENSITIVE_DATA
MANAGE_ADMINS
CHANGE_SYSTEM_SETTINGS
```

Do not give every admin unlimited permissions unless the business explicitly requires this.

High-risk actions should have additional authorization where appropriate.

---

# 14. Money Transfer Security Audit

If the application has any real-money transfer functionality, treat it as a high-risk business flow.

Review:

- Sender validation.
- Receiver validation.
- Balance validation.
- Amount validation.
- Currency validation.
- Minimum/maximum limits.
- Transaction state.
- Duplicate requests.
- Retry behavior.
- Authorization.
- Concurrency.
- Database transactions.
- Reversals.
- Audit trail.
- Fraud/abuse controls.
- Rate limiting.
- Error handling.
- Notifications.
- Administrative overrides.

## 14.1 Server-controlled amount

Never trust the client to define the final authoritative transfer result.

The server must validate and calculate:

- Amount.
- Sender balance.
- Receiver credit.
- Fees if applicable.
- Final status.

---

# 15. Atomic Money Operations

Transfers must be atomic where the database supports it.

Avoid states such as:

```text
Debit succeeds
+
Credit fails
```

or:

```text
Credit happens twice
```

or:

```text
Transaction marked successful
+
Balance was never updated
```

Use database transactions and appropriate locking/concurrency protection.

---

# 16. Idempotency for Financial Operations

Sensitive operations must protect against duplicate execution caused by:

- Double click.
- Refresh.
- Retry.
- Network retry.
- Browser resend.
- Mobile reconnection.
- Client timeout followed by retry.

A transfer request should have a unique idempotency/transaction key or equivalent server-side protection.

A repeated request must not create a second transaction.

---

# 17. Concurrency / Double-Spend Protection

Test simultaneous requests.

Example:

```text
Balance = 100

Request A → transfer 100
Request B → transfer 100
```

The system must not process both as successful.

Protect against:

- Race conditions.
- Lost updates.
- Duplicate credits.
- Duplicate debits.
- Negative balances caused by concurrent requests.

---

# 18. Financial Ledger / Immutable History

Maintain an auditable transaction history.

Do not silently rewrite historical financial records.

For corrections:

```text
Original transaction
        ↓
Reversal / Adjustment transaction
        ↓
Updated balance
```

The history should allow an administrator/auditor to understand:

- What happened.
- Who initiated it.
- When it happened.
- What amount changed.
- Why it changed.
- Which related transaction caused the adjustment.

---

# 19. Transaction State Machine

Use explicit transaction states where applicable, such as:

```text
PENDING
PROCESSING
COMPLETED
FAILED
CANCELLED
REVERSED
```

Do not allow arbitrary client-side status changes.

Define valid state transitions on the server.

Example:

```text
PENDING → PROCESSING
PROCESSING → COMPLETED
PROCESSING → FAILED
COMPLETED → REVERSED
```

Do not permit invalid transitions such as changing a finalized transaction directly to an arbitrary state.

---

# 20. Points Security

Treat points as a protected economic value inside the application.

Never trust the frontend for:

- Current points balance.
- Number of earned points.
- Number of spent points.
- Transfer amount.
- Redemption value.
- Reward eligibility.
- Admin adjustments.

All point-changing operations must be validated server-side.

---

# 21. Points Ledger

Where practical, maintain a ledger for:

```text
EARN
SPEND
TRANSFER
REWARD
REDEMPTION
ADMIN_ADJUSTMENT
REVERSAL
```

The system should be able to explain how a user's current points balance was produced.

Avoid untracked direct balance changes.

---

# 22. Protection Against Points Inflation / Abuse

Audit every endpoint capable of increasing points.

Check for:

- Replay attacks.
- Duplicate submissions.
- Repeated reward claims.
- Repeated challenge completion.
- Manipulated scores.
- Client-supplied point amounts.
- Direct API calls that bypass the UI.
- Race conditions.
- Excessive request frequency.

Business rules should be enforced server-side.

---

# 23. Security of Challenge Rewards and Scoring

If points/rewards are granted based on training or competition performance:

- Do not trust the submitted score blindly.
- Validate question/session ownership.
- Validate the session state.
- Validate completion.
- Validate timing rules where applicable.
- Prevent replaying the same completed session.
- Prevent submitting a fabricated result directly to an API.
- Prevent duplicate reward claims.

The backend should be capable of determining whether a reward is legitimately earned.

---

# 24. Session and Authentication Security

Perform a complete security review of:

- Login.
- Logout.
- Refresh.
- Session persistence.
- Device/session restrictions.
- Password reset.
- Token/session expiration.
- Forced logout.
- Admin sessions.
- Privileged session escalation.

Follow a structured application-security review covering authentication, session management, access control, input validation, data protection, business logic, API security, logging, and configuration.

---

# 25. Input Validation and Server-Side Validation

Review all inputs for:

- Type validation.
- Range validation.
- Length validation.
- Format validation.
- Business-rule validation.
- Authorization checks.
- Sanitization where appropriate.

Never rely exclusively on frontend validation.

---

# 26. Rate Limiting and Abuse Controls

Review sensitive endpoints for rate limiting, especially:

- Login.
- Password reset.
- Admin actions.
- Transfers.
- Point adjustments.
- Reward claims.
- Challenge creation.
- Challenge submissions.
- Report generation.
- Export generation.

Prevent users or automated clients from repeatedly triggering expensive or sensitive workflows.

---

# 27. Logging and Audit Trail

Sensitive operations should be auditable.

Log appropriate security/business events such as:

- Admin login.
- Admin permission changes.
- User deactivation.
- Point adjustment.
- Money transfer.
- Transaction reversal.
- Reward adjustment.
- Sensitive settings change.
- Password/security changes.
- Force logout.
- Failed authorization attempts.

Logs should contain enough information for investigation without exposing secrets.

Never log:

- Passwords.
- Raw authentication tokens.
- Private keys.
- Sensitive secrets.

---

# 28. Admin Audit Logs

Create or strengthen an Admin Audit Log.

For important administrative actions, record:

```text
actor
action
target
timestamp
result
reason/context when appropriate
request/reference ID
```

Example:

```text
Admin A
→ adjusted points
→ Student B
→ +100 points
→ 2026-09-28 20:14
→ reason: manual reward correction
```

Do not allow ordinary users to modify audit records.

---

# 29. Data Protection and Sensitive Information

Review:

- Database permissions.
- API responses.
- Exposed user data.
- Admin-only data.
- Financial information.
- Session identifiers.
- Error responses.
- Export files.
- PDF reports.

Return only the data the current user is authorized to see.

---

# 30. Security Review of API Endpoints

Create an inventory of API endpoints and classify them:

```text
Public
Authenticated User
Trainer
Admin
High-Risk Financial
High-Risk Points
Internal
```

For each endpoint verify:

```text
Authentication
Authorization
Object ownership
Input validation
Rate limiting
Business rules
Error handling
Logging
Idempotency where necessary
```

This should follow OWASP API Security Top 10 and ASVS principles rather than relying on informal checks.

---

# 31. Security Review of Admin Exports and Reports

Audit all:

- PDF exports.
- Excel exports.
- CSV exports.
- User reports.
- Financial reports.
- Point reports.

Make sure:

- Users cannot access another user's private report.
- Admin-only data is protected.
- Export endpoints perform authorization.
- IDs cannot be manipulated to access another person's file.
- Large export operations are protected against abuse.

---

# 32. Performance and UI Stability During Redesign

The redesign must not introduce unnecessary re-renders or repeated requests.

Pay special attention to the previously identified question-jump bug.

Avoid:

- Re-fetching the same session unnecessarily.
- Restoring stale data after every state update.
- Remounting the entire challenge screen after each answer.
- Duplicate API calls.
- State updates that trigger question restoration.

---

# 33. Implementation Strategy

Before coding:

1. Inspect the current project architecture.
2. Identify the shared training/challenge components.
3. Identify the central session/state model.
4. Identify all APIs related to questions and progress.
5. Identify all admin endpoints.
6. Identify all points/financial endpoints.
7. Identify the current design system/components.
8. Identify duplicate logic.
9. Identify security-sensitive business flows.

Then implement the changes systematically.

Do not patch each screen independently when a shared component/service is responsible for the behavior.

---

# 34. Required Deliverables from the Agent

After implementation, provide a concise technical summary containing:

### UI/UX
- Pages redesigned.
- New layout structure.
- New color system.
- Responsive changes.
- Challenge/training UX changes.

### Bug Fix
- Root cause of the question jump.
- Fix applied.
- Why the fix prevents regression.

### Security
- Endpoints reviewed.
- Authorization changes.
- Points protections.
- Financial protections.
- Idempotency/concurrency protections.
- Audit logging changes.
- Remaining risks, if any.

### Testing
Report results for:

- Mobile.
- Tablet.
- Desktop.
- Training.
- Robot challenge.
- PVP.
- Refresh.
- Orientation changes.
- Question transitions.
- Admin actions.
- Points operations.
- Financial operations.

---

# 35. Final Acceptance Criteria

## Question behavior
- Answering a question advances exactly once.
- The next question does not jump back.
- The issue is fixed globally.
- No stale session state overwrites current state.

## Training UI
- Clear human-centered layout.
- Calm and comfortable colors.
- Child-friendly without looking childish.
- Fast focused interaction.

## Competition UI
- Focused competition experience.
- Clear hierarchy.
- Minimal distractions.
- Calm visual system.
- Result screen is informative without being cluttered.

## Responsive design
- Works correctly across real screen sizes.
- Handles short-height screens.
- Handles RTL/LTR.
- Handles long text/numbers.
- No accidental overflow.

## Admin
- Clear operational structure.
- Sensitive actions protected server-side.
- Permissions are explicit.
- Audit trail exists for important actions.

## Money
- Server-authoritative validation.
- Atomic transaction handling.
- Idempotency.
- Concurrency protection.
- Immutable/auditable history.
- Explicit transaction states.

## Points
- Server-authoritative balance.
- Ledger/history.
- Replay protection.
- Duplicate-claim protection.
- Abuse/rate-limit controls.

## Security
- Authentication reviewed.
- Authorization reviewed.
- Object-level authorization reviewed.
- Function-level authorization reviewed.
- Input validation reviewed.
- Sensitive business flows reviewed.
- Logging/auditing reviewed.
- API endpoints reviewed.

---

# 36. Design and Security References

Use these as current reference material during implementation:

- Apple Human Interface Guidelines — Layout: https://developer.apple.com/design/human-interface-guidelines/layout
- Apple Human Interface Guidelines — Right to Left: https://developer.apple.com/design/human-interface-guidelines/right-to-left
- Apple Human Interface Guidelines — Branding: https://developer.apple.com/design/human-interface-guidelines/branding
- Vercel — New dashboard redesign: https://vercel.com/changelog/dashboard-navigation-redesign-rollout
- Linear — UI refresh: https://linear.app/changelog/2026-03-12-ui-refresh
- Linear — Design refresh process: https://linear.app/now/behind-the-latest-design-refresh
- Stripe Dashboard: https://support.stripe.com/topics/dashboard
- Stripe Reports: https://support.stripe.com/questions/stripe-reports
- OWASP API Security Top 10: https://owasp.org/www-project-api-security/
- OWASP API Security Top 10 2023: https://api-security.owasp.org/editions/2023/en/0x11-t10/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/

---

# Final Objective

Transform the website from a visibly template-like interface into a polished, human-centered educational product with a calm child-friendly visual identity, focused training and competition experiences, reliable question/session behavior, and a thorough server-side security model for Admin operations, points, transactions, and all sensitive business flows.

The implementation must improve the product without breaking existing functionality.


---

# Part II — Deep UI/UX, Design, and Color Research Reference

The following research is the design reference that should guide implementation of the redesign requested above.

# Deep UI/UX & Color Research — 2026
## Human-Centered Educational Math Platform

This document summarizes current web research into interface design, educational UX, responsive layouts, color systems, typography, accessibility, and child-friendly interaction patterns.

The recommendations are intended for the current educational math/training platform and should be used as design direction rather than as a template to copy.

---

# 1. Research Conclusions

The strongest direction for this product is not a generic "AI dashboard" and not an overly playful game UI.

The recommended direction is:

**Human-centered educational product + calm visual system + playful accents + strong task focus + professional information architecture.**

The product serves multiple audiences:

- Children/Students.
- Parents.
- Trainers.
- Administrators.

These users need different information densities and different interaction priorities.

The student experience should be visual, simple, encouraging, and focused.

The Admin experience should be operational, dense, organized, and efficient.

---

# 2. Current Design References Studied

## Apple Human Interface Guidelines

Apple's current 2026 guidance emphasizes adaptable layouts that respond to:

- Display size.
- Window size.
- Orientation.
- Aspect ratio.
- Text-size changes.
- Locale.
- RTL/LTR direction.

The important lesson is to design for available space rather than hard-coding layouts for a specific device.

Reference:
https://developer.apple.com/design/human-interface-guidelines/layout

Apple also recommends using color consistently and carefully, especially when color communicates status or interactivity.

Reference:
https://developer.apple.com/design/human-interface-guidelines/color

---

## Microsoft Fluent 2

Fluent provides useful principles for:

- Spacing.
- Visual hierarchy.
- Semantic colors.
- Typography.
- Responsive layouts.
- Accessibility.

Fluent's spacing guidance uses a consistent spacing ramp and emphasizes proximity: elements placed closer together are perceived as related, while intentional empty space helps establish hierarchy.

Reference:
https://fluent2.microsoft.design/layout

Fluent's color system separates:

- Neutral.
- Shared/semantic.
- Brand.

It recommends using semantic colors intentionally and avoiding excessive use of brand colors.

Reference:
https://fluent2.microsoft.design/color

Fluent also recommends that color should not be the only signal for state, and that responsive layouts should reflow instead of losing information.

Reference:
https://fluent2.microsoft.design/accessibility

---

## Khan Academy / Khan Academy Kids

Khan Academy is particularly relevant because the target product is educational.

Their current 2026 color-system redesign introduced a more structured semantic token architecture with:

- Background.
- Border.
- Foreground.
- Shadow.
- Contexts such as Instructive, Neutral, Success, Warning, Critical.
- Subtle / Default / Strong intensity levels.

They also stress-tested their palette for contrast and designed light/dark themes using a semantic token system.

Reference:
https://blog.khanacademy.org/how-we-rebuilt-khan-academys-color-system-from-the-ground-up/

Khan Academy Kids uses a visual-first approach for young learners, including:

- Child-friendly icons.
- Characters.
- Visual rewards.
- Simple interaction patterns.
- Personalized learning paths.
- Interactive math activities.

Reference:
https://blog.khanacademy.org/supporting-english-language-acquisition-with-khan-academy-kids/

Current Khan Academy Kids math positioning emphasizes early math confidence, number sense, addition/subtraction, problem solving, and adaptive practice.

Reference:
https://www.khanacademy.org/kids/math

---

## Duolingo

Duolingo is useful as a reference for:

- Friendly typography.
- Strong visual identity.
- Character-led communication.
- Illustration systems.
- Simple and recognizable interaction patterns.

Duolingo's typography guidance shows how a friendly rounded secondary typeface can support longer content while a stronger display typeface provides emphasis.

Reference:
https://design.duolingo.com/identity/typography

Duolingo also treats illustration and characters as a coherent visual system rather than random decorations.

Reference:
https://design.duolingo.com/identity/imagery

The important lesson for this project is not to copy Duolingo's colors or branding. Instead, use the principle of giving the product a consistent visual personality.

---

## Nielsen Norman Group

NN/G identifies scale, visual hierarchy, balance, contrast, and Gestalt as core principles of visual design.

Reference:
https://www.nngroup.com/articles/principles-visual-design/

For this project, this means visual hierarchy is more important than filling every section with decorative cards.

---

## W3C WCAG 2.2

WCAG 2.2 should be used as an accessibility baseline.

Important criteria include:

- Normal text contrast of at least 4.5:1.
- Large text contrast of at least 3:1.
- UI component/non-text contrast of at least 3:1.
- Minimum target size of 24×24 CSS pixels under WCAG 2.2 AA.
- A larger 44×44 target size is recommended by the enhanced criterion for easier touch interaction.
- Focus indicators should remain visible and should not be hidden by other content.

Reference:
https://www.w3.org/TR/wcag/

---

# 3. Recommended Visual Personality

The product should feel like:

- Friendly.
- Modern.
- Calm.
- Trustworthy.
- Educational.
- Light.
- Playful in small doses.
- Designed by a human product team.

It should not feel like:

- A generic SaaS dashboard.
- An AI-generated landing page.
- A cryptocurrency interface.
- A neon gaming platform.
- A childish toy.
- An over-designed glassmorphism template.

---

# 4. Recommended Color Strategy

Do NOT make every card, button, header, icon, and background colorful.

Instead use a layered semantic system.

## Layer 1 — Base surfaces

Use very light, low-saturation backgrounds.

Examples:

```text
Page background:     #F5F9FB
Surface:             #FFFFFF
Soft blue surface:   #DCEFF7
Soft green surface:  #DFF2EC
Soft yellow surface: #FFF2D6
Soft purple surface: #EEEAFB
Soft pink surface:   #FFE8EC
```

These softer colors work primarily as surfaces or supporting backgrounds.

---

## Layer 2 — Strong interactive colors

Use stronger colors for primary actions so text remains readable.

Recommended starting points:

```text
Primary Blue:   #2E6F95
Primary Teal:   #287D73
Warm Action:    #9A6A00
```

Calculated contrast against white:

```text
#2E6F95 + white = 5.49:1
#287D73 + white = 4.92:1
#9A6A00 + white = 4.73:1
```

These values meet the WCAG 2.2 AA 4.5:1 threshold for normal text.

Do not assume any arbitrary pastel color will work with white text.

---

# 5. Semantic Color Architecture

Use color by meaning, not decoration.

Recommended semantic roles:

```text
Primary
Secondary
Neutral
Success
Warning
Error/Critical
Info
Disabled
```

Each role should have:

```text
Subtle
Default
Strong
```

variants.

This follows the same general direction used by modern design systems such as Khan Academy's 2026 semantic color architecture.

Example:

```text
Success / Subtle
Success / Default
Success / Strong
```

This makes Light and Dark themes easier to maintain.

---

# 6. Dark Mode

Dark mode should not simply invert the light mode.

Use:

- Dark neutral backgrounds.
- Slightly elevated dark surfaces.
- Carefully adjusted accent colors.
- Strong text contrast.
- Reduced saturation when appropriate.

Maintain the same information hierarchy in both themes.

The user should still understand:

```text
Primary
Secondary
Neutral
Success
Warning
Error
```

without having to relearn the interface.

---

# 7. Typography Direction

Typography should be friendly but highly readable.

A rounded or humanist sans-serif is appropriate for the student experience.

A good starting direction:

```text
Arabic:
Cairo / IBM Plex Sans Arabic / Noto Sans Arabic

Latin:
Inter / Nunito Sans / Nunito
```

Do not mix many fonts.

Recommended:

```text
1 primary UI font
+ optional display accent font
```

Use sentence case rather than excessive ALL CAPS.

Keep line height comfortable.

Use stronger weight and size for hierarchy rather than relying on bright colors.

---

# 8. Spacing System

Use a consistent spacing scale.

Recommended base:

```text
4px grid
```

Possible tokens:

```text
4
8
12
16
20
24
32
40
48
56
64
```

This is aligned with the spacing-ramp approach used by Fluent.

Do not choose margins independently for every component.

---

# 9. Border Radius

Avoid making every component extremely rounded.

Recommended hierarchy:

```text
Small controls: 8–10px
Inputs:          10–12px
Cards:           14–18px
Large containers: 18–24px
Pills:           999px only when semantically appropriate
```

Use rounding to communicate grouping and friendliness, not as decoration everywhere.

---

# 10. Shadows

Use shadows sparingly.

Preferred approach:

- Mostly flat surfaces.
- Very subtle elevation.
- Stronger elevation only for dialogs/popovers.
- Avoid large glowing shadows.

Hierarchy should primarily come from:

1. Spacing.
2. Scale.
3. Typography.
4. Surface contrast.
5. Borders.
6. Small elevation.

This produces a more mature and human-designed appearance.

---

# 11. Student Dashboard

The dashboard should answer three questions immediately:

```text
What should I do now?
How am I progressing?
What can I do next?
```

Recommended structure:

```text
Header
↓
Welcome / Current Level
↓
Primary Training Action
↓
Progress Summary
↓
Training Choices
↓
Recent Activity
↓
Achievements / Encouragement
```

Do not create ten equally important cards.

There should be one clear primary action.

---

# 12. Training Page

Training should behave like a focused learning tool.

Recommended hierarchy:

```text
Level / Mode
↓
Question
↓
Answer
↓
Keyboard
↓
Progress / Timer
```

Everything not required during solving should be visually secondary.

Use color to reinforce:

- Current state.
- Progress.
- Success.
- Warnings.

Do not flood the screen with color.

---

# 13. Competition Page

Competition should feel distinct from normal training.

Use:

- Clear challenge identity.
- Minimal but meaningful opponent information.
- Timer.
- Progress.
- Current problem.
- Answer keyboard.
- Strong completion feedback only after the match state permits it.

Do not show excessive statistics during the live competition.

The competition page should feel fast and focused.

---

# 14. Result Page

The result screen is where richer visuals are appropriate.

Recommended hierarchy:

```text
Result headline
↓
Main score
↓
Accuracy
↓
Time / speed
↓
Progress visualization
↓
Answer review
↓
Next action
```

Use visualizations that explain performance rather than decoration.

---

# 15. Charts and Data Visualization

Do not use charts just because the product has data.

Use a chart only when it helps answer a question.

Examples:

### Performance over time

Line chart.

### Accuracy

Progress/ring or bar.

### Correct vs incorrect

Simple bar or segmented visualization.

### Speed

Line chart.

### Level progress

Progress bar/path.

Use consistent axis labels and avoid unnecessary chart decoration.

---

# 16. Admin Dashboard

The Admin interface should feel different from the student interface.

Student UI:

```text
Friendly + visual + focused
```

Admin UI:

```text
Clear + dense + operational
```

Recommended structure:

```text
Sidebar
├── Overview
├── Students
├── Trainers
├── Groups
├── Sessions
├── Attendance
├── Training
├── Challenges
├── Points
├── Transactions
├── Reports
├── Notifications
├── Settings
└── Security / Audit
```

Use tables, filters, search, bulk actions, and clear statuses.

---

# 17. Responsive Design Strategy

Do not use a separate "mobile layout" as an afterthought.

Use one responsive system that reflows.

Important cases:

- 320px width.
- 360px.
- 390px.
- 430px.
- Tablet.
- Laptop.
- Desktop.
- Short height.
- Large text.
- RTL.
- Landscape windows.

The design should use intrinsic sizing, Flexbox, CSS Grid, `clamp()`, `min()`, `max()`, container queries where useful, and semantic HTML.

Do not use absolute positioning for the main page layout.

---

# 18. Touch Interaction

For touch interfaces:

- Make important controls comfortably tappable.
- Avoid crowded keyboard buttons.
- Avoid placing destructive actions directly beside primary actions.
- Maintain visible pressed/focus states.
- Keep enough spacing between adjacent controls.

WCAG 2.2 sets a minimum target-size requirement of 24×24 CSS pixels for most pointer targets, with 44×44 as the enhanced target-size recommendation.

For the child-focused interface, using approximately 44px or larger for major touch controls is a reasonable practical design target.

---

# 19. Animation and Motion

Use motion to communicate state, not as decoration.

Good examples:

- Button press feedback.
- Question transition.
- Progress update.
- Completion celebration.
- Modal appearance.
- Page transition when it improves orientation.

Avoid:

- Constant floating animations.
- Pulsing every important element.
- Large bouncing UI.
- Excessive particles.
- Long transitions that slow down answering math problems.

Respect reduced-motion preferences.

---

# 20. Child-Friendly Interaction Principles

For younger users:

- Prefer clear visual affordances.
- Use recognizable icons.
- Keep wording short.
- Use characters sparingly and intentionally.
- Provide immediate understandable feedback.
- Make the main action obvious.
- Avoid hidden navigation.
- Avoid dense forms.

Khan Academy Kids is a useful example of visual-first educational design: child-friendly icons, character guidance, interactive animations, and visual rewards reduce dependence on long textual instructions.

However, the platform should not copy its visual identity.

---

# 21. The "Human Design" Test

After redesign, every page should pass these questions:

### Purpose
Can a user understand the purpose of the page in 2–3 seconds?

### Priority
Is there one obvious primary task?

### Hierarchy
Can the user distinguish primary, secondary, and tertiary information?

### Rhythm
Do spacing and grouping feel intentional?

### Identity
Does the product have a recognizable visual personality?

### Restraint
Is anything present only because "it looks nice"?

### Consistency
Does the same component behave the same everywhere?

### Accessibility
Does color remain understandable without relying on color alone?

### Mobile
Can a child use it comfortably with one hand?

---

# 22. Recommended Design Language for This Platform

The strongest direction for this product is:

## "Calm Educational Play"

Visual characteristics:

```text
Soft neutral background
+
White/soft-surface cards
+
One strong primary accent
+
Small secondary accent colors
+
Friendly rounded typography
+
Simple geometric icons
+
Occasional character/illustration
+
Clear spacing
+
Subtle depth
+
Purposeful motion
```

Not:

```text
Neon
+
Glass everywhere
+
Huge gradients
+
Dozens of cards
+
Random illustrations
+
Huge shadows
+
Constant animation
```

---

# 23. Recommended Student Color Example

A sample theme:

```text
Background       #F5F9FB
Surface          #FFFFFF
Primary Blue     #2E6F95
Secondary Teal   #287D73

Soft Blue        #DCEFF7
Soft Green       #DFF2EC
Soft Yellow      #FFF2D6
Soft Purple      #EEEAFB
Soft Pink        #FFE8EC

Text Primary     #243447
Text Secondary   #526273

Success          #287D73
Warning          #9A6A00
Critical         #B54747
```

These are starting tokens, not mandatory final brand colors.

The final palette should be validated against real UI backgrounds and both themes.

---

# 24. What the Agent Should Actually Do

The Agent should not simply replace the CSS.

It should:

1. Audit the existing page hierarchy.
2. Identify repeated patterns.
3. Identify inconsistent components.
4. Build a small design-token layer.
5. Normalize typography.
6. Normalize spacing.
7. Normalize buttons/inputs/cards.
8. Redesign the student shell.
9. Redesign training screens.
10. Redesign competition screens.
11. Redesign results.
12. Redesign Admin separately.
13. Implement responsive rules.
14. Test Arabic RTL.
15. Test English LTR.
16. Test light/dark mode.
17. Test touch interaction.
18. Test short-height screens.
19. Remove unnecessary decoration.
20. Verify accessibility.

Do not redesign every page independently.

Build reusable primitives first.

---

# 25. Final Design Acceptance Criteria

The redesign is complete only when:

- The UI no longer looks like a generic AI template.
- Training feels focused.
- Competition feels fast and purposeful.
- Children can visually understand the next action.
- Colors are calm and controlled.
- Accent colors are used intentionally.
- Typography has a clear hierarchy.
- Spacing is consistent.
- Cards are not overused.
- Mobile layouts are genuinely responsive.
- RTL is correct.
- Dark mode remains readable.
- Buttons are touch-friendly.
- Feedback is understandable without color alone.
- Accessibility contrast is validated.
- The Admin panel has a separate operational design language.
- The entire product feels like one coherent system.

---

# 26. Key References

Apple HIG — Layout:
https://developer.apple.com/design/human-interface-guidelines/layout

Apple HIG — Color:
https://developer.apple.com/design/human-interface-guidelines/color

Microsoft Fluent 2 — Layout:
https://fluent2.microsoft.design/layout

Microsoft Fluent 2 — Color:
https://fluent2.microsoft.design/color

Microsoft Fluent 2 — Accessibility:
https://fluent2.microsoft.design/accessibility

Khan Academy — 2026 Color System:
https://blog.khanacademy.org/how-we-rebuilt-khan-academys-color-system-from-the-ground-up/

Khan Academy Kids — Math:
https://www.khanacademy.org/kids/math

Khan Academy Kids — Visual-first design:
https://blog.khanacademy.org/supporting-english-language-acquisition-with-khan-academy-kids/

Duolingo — Typography:
https://design.duolingo.com/identity/typography

Duolingo — Imagery:
https://design.duolingo.com/identity/imagery

Nielsen Norman Group — Visual Design Principles:
https://www.nngroup.com/articles/principles-visual-design/

W3C — WCAG 2.2:
https://www.w3.org/TR/wcag/

