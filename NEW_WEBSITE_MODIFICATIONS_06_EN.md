# Website Modification Specification 06
## Study Levels 5–10 Configuration, Multiplication/Division Availability, Typography Scaling, and Admin RTL UI Corrections

> **Standalone implementation specification.**
>
> Use `PLATFORM_KNOWLEDGE_BASE.md` as the source of truth before changing code.
>
> **Critical:** Inspect the existing implementation first. Do not guess architecture, create duplicate configuration systems, or break the existing distinction between the student's **Study Level (1–10)** and the current **Math Engine Levels (1–4)**.

---

## 1. Mandatory Pre-Implementation Analysis

Before writing code:

1. Read `/home/z/my-project/worklog.md`.
2. Read the current `PLATFORM_KNOWLEDGE_BASE.md`.
3. Inspect `src/lib/rules-engine.ts`.
4. Inspect the current `math_levels_json` configuration.
5. Inspect `engineLevelFor()` and where it is used.
6. Inspect the Admin Levels settings panel and its API.
7. Inspect the training setup flow and how `User.level` is used.
8. Inspect the multiplication/division training availability logic.
9. Inspect the shared typography and responsive styles in `globals.css` and related UI components.
10. Inspect Admin pages/components for RTL/layout problems.

Before implementation, report:

```text
Current Study-Level Architecture
Current Math-Level Architecture
Current Admin Level Settings Architecture
Current Multiplication/Division Availability Logic
Current Typography/Responsive Architecture
Current Admin RTL/Layout Problems
```

Do not begin a large rewrite before understanding these pieces.

---

# 2. Add Admin Configuration for Study Levels 5–10

The platform currently has:

- Student Study Levels: `User.level = 1–10`
- Math Engine Levels: currently `1–4`
- Central Rules Engine configuration: `math_levels_json`

The new requirement is to give the Admin full configurable settings for:

```text
Level 5
Level 6
Level 7
Level 8
Level 9
Level 10
```

The configuration experience should be comparable to the existing administration experience for Levels 1–4.

## Important architectural requirement

Do not automatically turn the existing four Math Engine Levels into ten independent engines.

First inspect the current implementation and determine whether the clean architecture is:

### Option A
Keep the existing 4 Math Engine levels and add a separate **Study-Level Configuration** layer for Study Levels 1–10.

### Option B
Expand the actual Math Rules Engine into 10 independently configurable levels.

Choose the architecture that best matches the current code and the intended platform behavior.

The Agent must explain the chosen architecture before implementation.

---

# 3. Per-Level Configuration

For every Study Level 5–10, the Admin should be able to manage the same applicable types of educational configuration already supported for the earlier levels.

Potential fields include:

- Enabled/disabled.
- Difficulty.
- Minimum floors.
- Maximum floors.
- Allowed exact floor counts.
- Number of problems.
- Allowed number ranges.
- Digit length.
- Answer range.
- Complement rules.
- Carry/borrow behavior.
- Problem complexity/density.
- Other configuration properties already supported by the Rules Engine.

Do not invent duplicate configuration fields when an existing configuration primitive can be reused.

---

# 4. Multiplication Availability for Levels 5–10

For each Study Level:

```text
5
6
7
8
9
10
```

the Admin must be able to choose:

```text
Multiplication: ON / OFF
```

Example:

```text
Level 5 → ON
Level 6 → OFF
Level 7 → ON
Level 8 → ON
Level 9 → OFF
Level 10 → ON
```

These are examples only. The Admin controls the actual values.

---

# 5. Division Availability for Levels 5–10

For each Study Level:

```text
5
6
7
8
9
10
```

the Admin must be able to choose:

```text
Division: ON / OFF
```

The settings must be independent.

For example:

```text
Level 5 → Division ON
Level 6 → Division OFF
```

---

# 6. Availability Must Be Enforced Server-Side

The ON/OFF values must not be a frontend-only restriction.

If a student's configuration says:

```text
Multiplication = OFF
```

the backend must reject an attempt to start multiplication.

Likewise for Division.

Enforce the rule against:

- Normal UI interaction.
- Direct URL access.
- Manually crafted API requests.
- Modified frontend payloads.

The server should derive the student's Study Level from the authenticated session/user and then load the appropriate configuration.

---

# 7. Protect Direct Training Routes

The student must not bypass game availability by directly opening:

```text
/training/multiplication
```

or:

```text
/training/division
```

when the relevant game is disabled for their level.

Return an appropriate Arabic RTL response and redirect/state.

Do not rely only on hiding menu items.

---

# 8. Student Training Menu Must Follow Level Configuration

When the student opens the training area:

```text
Authenticated User
        ↓
Read User.level
        ↓
Read Study-Level Configuration
        ↓
Determine available games
        ↓
Render only allowed games
```

Example:

```text
Student Level 7
Multiplication = ON
Division = OFF
```

The training UI should expose multiplication and hide/disable division.

Backend enforcement must still remain in place.

---

# 9. Keep Addition/Subtraction Automatic

Do not reintroduce the manual four-option Math Level selector.

The platform currently distinguishes:

```text
Study Level = User.level (1–10)
```

from:

```text
Math Engine Level = 1–4
```

The normal addition/subtraction flow should continue to determine the engine level automatically using the existing mapping mechanism (`engineLevelFor()`), unless the architecture review proves that the new Study-Level configuration requires a carefully designed extension.

Students should not manually select Engine Levels 1–4.

---

# 10. Preserve the Single Rules Engine

The platform's current rule is:

```text
src/lib/rules-engine.ts
```

is the shared Rules Engine.

Do not create separate rule-engine files for Levels 5–10 unless the architecture review demonstrates that this is absolutely necessary.

Avoid:

```text
rules-engine-5.ts
rules-engine-6.ts
rules-engine-7.ts
...
```

Prefer one configurable engine and one source of truth.

---

# 11. Admin Level Settings UI

Extend the Admin level settings area so that Levels 1–10 can be managed clearly.

A possible structure:

```text
Level Settings

Level 1
[configuration]

Level 2
[configuration]

Level 3
[configuration]

Level 4
[configuration]

Level 5
[configuration]
  Multiplication [ON/OFF]
  Division       [ON/OFF]

Level 6
[configuration]
  Multiplication [ON/OFF]
  Division       [ON/OFF]

...

Level 10
[configuration]
  Multiplication [ON/OFF]
  Division       [ON/OFF]
```

Use tabs, accordions, grouped panels, or another structure that prevents a giant unreadable Admin page.

---

# 12. Settings Validation and Persistence

Every new setting must be:

- Validated server-side.
- Sanitized.
- Persisted correctly.
- Given safe defaults.
- Compatible with the existing settings architecture.
- Compatible with the Firestore/Supabase dual backend.

The current central setting is:

```text
math_levels_json
```

If it can safely contain the new configuration, preserve backward compatibility.

If a separate setting is required, document the reason and make the relationship between the settings explicit.

Do not create two conflicting sources of truth.

---

# 13. Future Extensibility

Design the configuration layer so future controls can be added per Study Level without another structural rewrite.

Potential future examples:

```text
Abacus ON/OFF
Robot ON/OFF
PVP ON/OFF
Difficulty per game
Game-specific floor count
Game-specific limits
```

Do not implement these now unless required.

Make the architecture ready for them.

---

# 14. Typography Problem — Global Font Size Is Too Large

There is currently a broad UI problem where text and controls appear larger than necessary.

This causes:

- Important text to be cut off.
- Labels to disappear.
- Buttons to become oversized.
- Cards to become unnecessarily tall.
- Admin controls to become cramped.
- More scrolling than necessary.
- Information density to become poor.

This requires a **global typography audit**.

Do not solve it with random per-element font-size overrides.

---

# 15. Typography Audit

Inspect:

- Global/base font size.
- Heading scale.
- Section headings.
- Card headings.
- Body text.
- Labels.
- Buttons.
- Inputs.
- Tables.
- Badges.
- Navigation.
- Modals.
- Admin controls.
- Training content.
- Challenge HUD.

Identify the actual global source of the oversized typography.

---

# 16. Responsive Typography System

Create a coherent responsive typography scale.

Use responsive values such as `clamp()` where appropriate.

The goal is:

```text
Desktop
→ readable, balanced hierarchy

Mobile
→ compact but still readable
```

Not:

```text
Desktop typography
→ randomly shrink individual elements
```

Do not hide content to compensate for oversized typography.

---

# 17. Arabic Typography

The platform is:

```text
100% Arabic RTL
Cairo font
```

Optimize typography for Arabic content.

Review:

- Line height.
- Font weights.
- Heading hierarchy.
- Button text.
- Label wrapping.
- Mixed Arabic/English content.
- Large numbers.
- Long Arabic names.
- Arabic/number alignment.

Keep the existing RTL behavior.

---

# 18. Admin Buttons and Controls Are Reversed / Mirrored Incorrectly

Some Admin buttons and controls currently appear visually reversed or misaligned.

Examples may include:

- Icon/text order.
- Switch direction.
- Dropdown arrows.
- Pagination arrows.
- Form controls.
- Checkbox/radio alignment.
- Action button groups.
- Table action columns.

Audit these issues at the shared component level.

---

# 19. Fix RTL Correctly — Do Not Disable RTL

The whole application must remain:

```text
dir="rtl"
```

Do not globally switch Admin pages to LTR as a shortcut.

Instead distinguish between:

```text
RTL page structure
```

and:

```text
Components with intentional directional semantics
```

Examples:

- Arabic text → RTL.
- Numeric keypad → LTR visual ordering.
- Numeric values → appropriate numeric alignment.
- Back arrow → semantic direction.
- Forward arrow → semantic direction.
- Toggle/switch → correct interaction direction.
- Progress indicator → correct logical direction.

---

# 20. Admin Button Pattern

Create/normalize a reusable button pattern for:

```text
Icon + Arabic label
```

Do not rely on scattered hacks like:

```css
flex-row-reverse
margin-left
margin-right
```

throughout individual screens.

Fix the underlying shared component where possible.

---

# 21. Admin Switches / Toggles

Audit all toggles.

Verify:

- ON state.
- OFF state.
- Handle/indicator position.
- Label position.
- Focus state.
- Disabled state.
- Hit area.
- RTL behavior.

Do not mirror a switch incorrectly just because the surrounding page is RTL.

---

# 22. Admin Tables

Review:

- Action columns.
- Action button ordering.
- Sort arrows.
- Pagination.
- Dropdown menus.
- Status badges.
- Numeric cells.
- Alignment.

Ensure the interface is visually logical in Arabic RTL.

---

# 23. Admin Forms and Modals

Audit:

- Labels.
- Inputs.
- Input icons.
- Password visibility controls.
- Save/Cancel actions.
- Close buttons.
- Dropdown/select arrows.
- Radio buttons.
- Checkboxes.
- Toggles.

Do not use global RTL CSS that fixes one control and breaks another.

---

# 24. Fix Shared Components at the Source

If the root cause is in:

```text
src/components/ui/*
```

or:

```text
src/components/views/admin/*
```

fix the shared component rather than adding page-specific workarounds everywhere.

The goal is:

```text
One correct Button
→ correct everywhere

One correct Switch
→ correct everywhere

One correct Select
→ correct everywhere
```

---

# 25. Do Not Break Numeric LTR Components

While correcting RTL:

Preserve intentionally LTR components.

The numeric keypad must remain:

```text
1 2 3
4 5 6
7 8 9
  0
```

Do not reintroduce RTL reversal.

---

# 26. Responsive Admin Layout

After typography and RTL fixes, verify Admin on:

- Small phones.
- Standard phones.
- Large phones.
- Tablets.
- Laptop.
- Desktop.
- Large desktop.
- Short-height screens.

Preserve comfortable touch targets.

Existing platform requirement:

```text
touch targets ≥44px
```

Reducing font size must not make important controls too small to operate.

---

# 27. Preserve Existing Calm Design System

The current design direction is:

```text
Calm Educational Play
```

Keep:

- Calm colors.
- Cairo.
- RTL.
- Light/dark themes.
- Responsive behavior.
- Reduced-motion support.

Do not replace the established design system just to solve typography or RTL bugs.

---

# 28. Backend Enforcement for Game Availability

The server must validate:

```text
Authenticated User
+
Study Level
+
Game Type
+
Current Study-Level Configuration
```

before starting training.

Relevant existing route:

```text
POST /api/student/training/start
```

The exact enforcement location should follow the current architecture.

The client cannot be trusted to decide whether a game is available.

---

# 29. Preserve Existing Game Configuration

Current known game settings remain:

```text
Multiplication:
  digits1 = 1–4
  digits2 = 1–3

Division:
  dividendDigits = 2–4
  divisorDigits = 1–2

Abacus:
  columns = 3–13
  mode = free | challenge
  theme = wood | neon | candy | gold | glass
```

Do not change these mathematical defaults unless explicitly requested.

This task adds **per-study-level availability/configuration** and UI improvements.

---

# 30. Backward Compatibility

Do not break existing:

- Student accounts.
- Study levels.
- Training history.
- Points.
- Match history.
- Active sessions.
- Admin settings.
- Existing Level 1–4 configurations.

Provide safe defaults for Levels 5–10 if no new configuration exists yet.

Existing data must remain valid.

---

# 31. Required Level 5–10 Tests

For each:

```text
Level 5
Level 6
Level 7
Level 8
Level 9
Level 10
```

verify:

- Admin can load settings.
- Admin can modify settings.
- Admin can save.
- Settings persist.
- Settings are validated.
- Student receives the correct level configuration.
- Multiplication ON/OFF works.
- Division ON/OFF works.
- Disabled games are hidden/disabled in the UI.
- Disabled games are rejected by the backend.
- Direct URLs cannot bypass restrictions.
- Enabled games continue working normally.

---

# 32. Typography Tests

Check:

- Dashboard.
- Training setup.
- Multiplication.
- Division.
- Addition/Subtraction.
- Arena.
- Robot.
- PVP.
- Statistics.
- Profile.
- All Admin panels.

Verify:

- No important text is clipped.
- Arabic labels wrap correctly.
- Buttons remain readable.
- Cards are not unnecessarily tall.
- Tables remain usable.
- No unexpected horizontal overflow.

---

# 33. Admin RTL Tests

Verify:

- Buttons.
- Icon buttons.
- Toggles.
- Checkboxes.
- Radio buttons.
- Selects.
- Dropdowns.
- Tables.
- Pagination.
- Modals.
- Forms.
- Status badges.
- Admin navigation.

Also verify that components requiring intentional LTR behavior remain LTR.

---

# 34. Required Agent Report Before Coding

Before implementation, report:

## Level architecture
- Current difference between `User.level` and Math Engine Levels.
- Current `math_levels_json` structure.
- Current `engineLevelFor()` behavior.
- Best architecture for Study Levels 5–10.
- Whether a separate Study-Level Configuration layer is needed.

## Game availability
- Where multiplication/division availability currently comes from.
- Where it is enforced.
- Which endpoints/components need modification.

## Typography
- Root cause of oversized text.
- Global styles/tokens responsible.
- Components with local overrides.

## RTL
- Root causes of reversed Admin controls.
- Shared components causing the issue.
- Components that intentionally require LTR.

Do not begin a large rewrite without this analysis.

---

# 35. Verification Requirements

After implementation:

1. Run:
   ```text
   bun run lint
   ```
2. Check `dev.log`.
3. Run the relevant regression suite.
4. Perform browser E2E.
5. Test Level 5–10 configuration.
6. Test multiplication/division availability.
7. Test direct-route/API restrictions.
8. Test addition/subtraction automatic engine selection.
9. Test typography on mobile and desktop.
10. Test Admin RTL controls.
11. Verify existing sessions and historical data remain intact.
12. If settings/database changes are made, verify dual-engine consistency/drift according to `PLATFORM_KNOWLEDGE_BASE.md`.

---

# 36. Final Goal

The Admin should be able to manage the platform across:

```text
Study Level 1
Study Level 2
Study Level 3
Study Level 4
Study Level 5
Study Level 6
Study Level 7
Study Level 8
Study Level 9
Study Level 10
```

For Levels 5–10, the Admin must be able to control whether:

```text
Multiplication = ON/OFF
Division       = ON/OFF
```

These settings must be enforced at both the UI and backend levels.

At the same time:

- Typography becomes balanced and compact enough to keep important content visible.
- Arabic RTL Admin controls display correctly.
- Direction-sensitive controls remain semantically correct.
- Touch targets remain usable.
- The existing calm visual system remains intact.
- The distinction between Study Levels and Math Engine Levels remains clear.
- No duplicate Rules Engine is created.
- Existing training, robot, PVP, session, and scoring behavior remains intact.

The result should be a scalable Level 1–10 administration system and a cleaner, more usable interface without creating conflicting sources of truth.

---

# 37. Full-Viewport Layout — Eliminate Unnecessary Side White Space

There is another global layout problem across the website:

On desktop and some larger viewport sizes, the application content does not properly use the available screen width. Noticeable empty/white areas remain on the left and right sides of the main content.

The application should use the available viewport much more effectively.

## Required behavior

Review the global page container and all major layouts.

Do not force the entire application into an unnecessarily narrow centered column.

The main application shell should be capable of expanding appropriately on large screens while still maintaining readable content widths inside individual text-heavy sections.

### Important distinction

There is a difference between:

```text
Useful content max-width
```

and:

```text
Unnecessarily narrow page container
```

Do not simply set every element to `width: 100%`.

Instead:

- The application shell should use the available viewport appropriately.
- Major sections should expand to use available horizontal space.
- Grids should gain columns or larger content regions when space allows.
- Training, Admin, Dashboard, Arena, and Statistics layouts should not leave large unused side margins without a clear design reason.
- Text blocks may still keep a sensible `max-width` for readability.
- Full-width sections should actually reach the intended application content boundaries.

## Audit the likely root causes

Inspect for:

- Fixed `max-width` containers that are too narrow.
- Excessive horizontal padding.
- Nested containers each adding their own side margins.
- `mx-auto` combined with restrictive widths.
- Hard-coded desktop widths.
- Unnecessary `max-w-*` Tailwind classes.
- Parent containers preventing children from expanding.
- Grid columns not using available space.
- Width calculations that assume a specific screen size.

Do not fix this by removing every `max-width`.

Use the correct container strategy per page type.

---

# 38. Responsive Container Strategy

Create a consistent global container strategy.

The application should support a hierarchy such as:

```text
Viewport
    ↓
Application Shell
    ↓
Responsive Content Container
    ↓
Page Sections
    ↓
Readable Content Blocks
```

For example:

- Dashboard and Admin overview sections can use wide containers.
- Tables may use nearly the full available content width.
- Training/challenge interfaces can use the available viewport while maintaining intentional internal spacing.
- Long text paragraphs can retain a readable line length.

The exact breakpoints and widths must be derived from the existing design system, not guessed independently on every page.

---

# 39. Eliminate Horizontal Scroll Caused by the New Full-Width Layout

Expanding layouts must not introduce horizontal overflow.

After changing container widths, verify:

- No horizontal page scrolling.
- No cards extending outside the viewport.
- No tables forcing the entire page wider than the viewport.
- No buttons or inputs overflowing containers.
- No fixed-width components breaking on small devices.

Use responsive overflow strategies where appropriate for data-heavy elements without allowing the whole page to become horizontally scrollable.

---

# 40. Mobile Browser Zoom — Prevent Accidental Page Zoom

On mobile, I do **not** want users to be able to pinch-to-zoom or accidentally zoom the application interface.

The website should behave like a controlled mobile application interface.

## Required behavior

Configure the application's mobile viewport so that:

- The page uses the device width.
- The initial scale is exactly 1.
- Pinch-to-zoom is disabled for the application interface.
- The page does not open already zoomed in/out.
- Double-tap browser zoom behavior should not cause the training/application interface to jump in scale where the platform/browser permits control.

The implementation should use the appropriate viewport configuration in the Next.js application layout.

The existing application uses:

```text
src/app/layout.tsx
```

Inspect the current `<head>` / viewport configuration and update it correctly.

---

# 41. Important Accessibility / Scope Clarification for Mobile Zoom

Disabling browser zoom is requested specifically for the application's fixed mobile interaction model.

Do not use arbitrary CSS hacks that break text resizing or accessibility.

Before applying the restriction globally, verify the current application structure and consider whether the restriction should be limited to the student/training/challenge interface rather than every page.

The primary requirement is to prevent unwanted scaling during:

- Training.
- Robot Challenge.
- Friend/PVP Challenge.
- Other fast-interaction student screens.

Do not sacrifice readable text or accessibility solely to force a visual scale.

---

# 42. Mobile Viewport Requirements

Verify that the mobile viewport configuration works correctly with:

- iOS Safari.
- Android Chrome.
- Modern mobile browsers.
- Portrait orientation.
- The existing landscape orientation guard.

The application must continue to work with:

```text
orientation-guard
```

and the existing portrait-first training/challenge design.

Do not create a second orientation system.

---

# 43. Full-Width Layout Must Respect Safe Areas

On phones with display cutouts/notches or browser safe areas:

- Content must remain inside the safe area.
- Buttons must not be hidden under the status/navigation areas.
- The full-width layout must still have appropriate internal padding.
- The Header and bottom navigation must respect safe-area insets where applicable.

The goal is:

```text
Full use of the viewport
+
Safe internal spacing
```

not:

```text
Full width
+
Content touching the physical screen edge
```

---

# 44. Apply the Full-Viewport Review to All Major Pages

Audit at least:

- Dashboard.
- Statistics.
- Notifications.
- Profile.
- Arena.
- Robot Challenge.
- Friend/PVP Challenge.
- Addition/Subtraction training.
- Multiplication.
- Division.
- Abacus.
- Admin overview.
- Admin students.
- Admin arena.
- Admin trainers.
- Admin money.
- Admin notifications.
- Admin statistics.
- Admin levels.

Look for large unexplained side margins or unused regions.

Do not assume the same container width is appropriate for every page.

---

# 45. Acceptance Tests — Full Screen Usage

## Desktop

At common wide desktop sizes:

```text
1366px
1440px
1920px
2560px
```

verify:

- The application uses the available horizontal area intelligently.
- No unnecessary large white margins exist on both sides.
- Main sections expand appropriately.
- Content remains readable.
- Layout does not look stretched or empty.

## Tablet

Verify:

- The layout reflows correctly.
- Columns collapse appropriately.
- No large empty side spaces remain.

## Mobile

Verify:

- Content fills the usable viewport width.
- Internal padding remains comfortable.
- No accidental horizontal overflow.
- No accidental zooming/pinch scaling of the application interface.
- The page opens at scale 1.

---

# 46. Required Agent Report for This Change

Before implementation, report:

### Full-width issue

- Which global container(s) currently restrict the width.
- Which pages inherit those restrictions.
- Which `max-width`, padding, margin, grid, or wrapper rules are responsible.
- Proposed container strategy.

### Mobile zoom

- Current viewport configuration in `src/app/layout.tsx`.
- Whether a viewport meta configuration already exists.
- Exact implementation needed.
- Scope of the zoom restriction.

Do not make blind global CSS changes before identifying the real source of the layout behavior.

---

# 47. Final Goal for the Global Layout

The website should feel like it owns the full viewport.

Instead of:

```text
|       large white margin       |
|    [ narrow content area ]     |
|       large white margin       |
```

the goal is:

```text
| [ responsive application content across the available viewport ] |
```

while keeping:

- Comfortable internal spacing.
- Readable text widths.
- Safe areas.
- Correct RTL.
- Responsive grids.
- No horizontal overflow.

On mobile:

```text
Device viewport
      ↓
Application fills usable width
      ↓
Portrait-first interface
      ↓
No accidental browser zoom
      ↓
Stable interaction scale
```

This should be implemented as part of the global layout system, not as isolated page-specific patches.


---

# 48. Responsive Tablet Layout and Visual Proportion Improvement

The current interface does not have a sufficiently polished visual proportion on tablet-sized devices and tablet-like viewports.

The layout must be redesigned so that tablet screens feel intentionally designed rather than looking like an enlarged mobile layout or a compressed desktop layout.

This applies especially to:

- Training screens.
- Addition/subtraction gameplay.
- Multiplication gameplay.
- Division gameplay.
- Abacus gameplay.
- Arena waiting/match screens.
- Robot match screens.
- Student dashboard cards and grids.
- Any fullscreen numeric-answer interface.

Do not solve this by simply increasing or decreasing every font and component globally.

The implementation must use responsive composition, proportional spacing, content-aware sizing, and appropriate breakpoints.

---

# 49. Tablet Breakpoint Strategy

Audit the existing responsive behavior and establish a deliberate layout strategy for tablet-sized widths.

The exact breakpoint values may follow the existing design system where appropriate, but the UI must clearly handle at least these ranges:

```text
Small mobile      < 480px
Large mobile      480–767px
Tablet            768–1023px
Large tablet      1024–1279px
Desktop           1280px+
```

Do not create unnecessary breakpoint fragmentation.

Prefer a small number of coherent layout states over many one-off media-query patches.

The tablet state must have its own proportional behavior where required.

Examples of tablet-specific improvements:

- Increase useful content width compared with mobile.
- Keep cards visually balanced instead of excessively wide or excessively narrow.
- Use 2-column layouts where they improve density and readability.
- Increase horizontal spacing moderately, not proportionally without limits.
- Prevent controls from becoming oversized simply because the viewport is wider.
- Keep important gameplay content centered.
- Preserve comfortable touch targets.
- Avoid large empty vertical or horizontal regions.

---

# 50. Visual Proportion and Component Sizing

Review the relative sizes of:

- Game/question card.
- Question text.
- Answer input/display.
- Numeric keypad.
- Start/submit buttons.
- Timer.
- Progress indicator.
- Header controls.
- Status badges.
- Cards and panels.

The current problem is not only spacing; the components must have better visual proportion as a system.

For tablet-sized screens, avoid layouts where:

```text
[ very small question ]

[ huge empty card area ]

[ oversized answer control ]
```

or:

```text
[ oversized question ]
[ cramped controls   ]
```

The goal is a balanced composition where the primary learning/game interaction receives the most visual priority.

Use content-aware sizing rather than forcing every component to the same fixed height.

Avoid excessive `height`, `min-height`, or `padding` values that create large unused regions on tablet screens.

Also avoid removing all spacing simply to make the interface fill the viewport.

---

# 51. Tablet Gameplay Composition

On training, PVP, and robot screens, the primary interaction area should remain visually centered and easy to scan.

Prefer a structure conceptually similar to:

```text
┌─────────────────────────────────────────────┐
│             Compact Game Header             │
├─────────────────────────────────────────────┤
│                                             │
│                Progress / Timer             │
│                                             │
│          ┌───────────────────────┐          │
│          │                       │          │
│          │      QUESTION         │          │
│          │                       │          │
│          └───────────────────────┘          │
│                                             │
│           Answer / Input Area               │
│                                             │
│              Numeric Keypad                 │
│                                             │
└─────────────────────────────────────────────┘
```

The exact visual implementation may differ, but the hierarchy must remain clear:

1. Game status.
2. Question.
3. Answer interaction.
4. Input controls.
5. Secondary controls.

Do not let secondary UI compete visually with the question.

---

# 52. Critical Bug: Long Questions Must Never Enter the Answer Field

There is currently a layout problem where a long math question can visually extend into, overlap, or collide with the answer area.

This must be fixed structurally, not with a superficial margin adjustment.

A long question must always remain inside its own dedicated question region.

The answer region must remain a separate layout region.

Never allow the question text to:

- Overlap the answer field.
- Cover the answer field.
- Push the answer field into an invalid position.
- Visually collide with keypad buttons.
- Overflow outside the question card.
- Be clipped in a way that hides the mathematical expression.

The answer input/display must never be positioned directly over flowing question text.

Avoid absolute positioning for the relationship between question content and the answer field unless the bounding geometry is explicitly calculated and tested.

Prefer normal document flow, CSS Grid, or Flexbox with dedicated rows/regions.

---

# 53. Long Question Handling and Auto-Fit

The question display must be able to handle longer mathematical expressions safely.

Examples include:

```text
123 + 456 - 78 + 9
```

and longer combinations where the expression may exceed the width available on a mobile or tablet screen.

Implement a robust strategy such as:

- Wrapping at safe mathematical boundaries when appropriate.
- Responsive font sizing with a controlled minimum size.
- Content-aware line height.
- A question container that can grow vertically.
- Separate reserved space for the answer interaction.
- Recalculation when viewport size changes.

Do not use uncontrolled text shrinking until the expression becomes unreadably small.

Do not allow the question to force the answer control out of the viewport.

Do not use a fixed one-line question container if the expression can exceed the available width.

If the project already has an AutoFit utility/component, reuse and improve it rather than creating a competing implementation.

---

# 54. Question Card Height Must Be Content-Aware

The question card should adapt to content length while maintaining a stable overall interaction pattern.

The preferred behavior is:

```text
Short question
    ↓
Compact question region
    ↓
Answer region remains in normal position

Long question
    ↓
Question region grows or wraps safely
    ↓
Answer region moves down naturally
    ↓
No overlap
```

Do not allow the question card to grow indefinitely and push the keypad or primary action button beyond the usable viewport.

When a maximum visual height is necessary, define a safe fallback strategy that preserves readability and does not overlap adjacent controls.

The question area and answer area should have clear visual separation.

---

# 55. Mathematical Text Rendering

Review how mathematical expressions are rendered inside the gameplay UI.

Ensure:

- Digits remain clearly separated.
- Operators have consistent spacing.
- Multi-digit numbers do not visually collide.
- Wrapped expressions remain understandable.
- RTL layout does not reverse or distort the intended mathematical reading order.
- The answer area is visually distinct from the expression.

For gameplay expressions, the mathematical reading order and interaction behavior must remain correct even though the overall application is Arabic RTL.

Do not blindly apply RTL direction to mathematical expressions where that would make the expression visually incorrect.

Use the existing project convention for the mathematical question area and numeric keypad direction.

---

# 56. Responsive Numeric Keypad on Tablet

The numeric keypad must also be reviewed for tablet proportions.

Do not simply scale the mobile keypad to 150% or 200%.

The keypad should:

- Use the available width efficiently.
- Maintain equal button proportions.
- Preserve the established 3×4 layout where applicable.
- Keep touch targets comfortably large.
- Maintain consistent gaps.
- Remain centered and visually balanced.
- Never overlap the answer field or surrounding controls.

For the established keypad layout:

```text
1  2  3
4  5  6
7  8  9
0
```

preserve the existing interaction order and `dir="ltr"` requirement.

On wider tablets, use available width intelligently rather than making each button unnecessarily huge.

---

# 57. Viewport-Height Awareness

Tablet layouts must account for viewport height as well as viewport width.

A layout that fits correctly on a tall tablet must also work on a shorter tablet viewport.

Audit usage of:

- `100vh`
- `100dvh`
- `min-height`
- fixed card heights
- fixed keypad heights
- fixed header heights
- large vertical padding

Prefer modern viewport units such as `dvh` where appropriate, while preserving compatibility with the project's supported browsers.

The primary question and answer interaction must remain usable without accidental clipping below the viewport.

On devices with browser UI changes, the layout should not produce unstable jumps or overlap.

---

# 58. Tablet Portrait and Landscape Behavior

Respect the existing product requirement that certain challenge interfaces are portrait-first and already have an orientation guard.

Do not remove or bypass the existing orientation guard.

Instead:

- Make portrait tablet layouts polished and balanced.
- Make landscape behavior intentional where the screen is allowed to use landscape.
- Do not force desktop layouts onto a landscape tablet if that creates poor proportions.
- Do not allow question/answer overlap in either orientation.

Where portrait is required, preserve the current Arabic instruction:

```text
أدر موبايلك رأسي
```

and integrate it cleanly with the responsive layout.

---

# 59. Full-Width Does Not Mean Full-Stretch

The previous full-viewport requirement remains active, but it must be implemented together with this tablet redesign.

The UI must use the available viewport more effectively without stretching individual components to unnatural sizes.

Bad:

```text
Viewport width: 1024px

[ huge 1000px card ]
```

Also bad:

```text
Viewport width: 1024px

[ narrow 500px mobile-style card ]
```

Preferred:

```text
Viewport width: 1024px

[ balanced responsive content region ]
```

Use:

- Fluid width where appropriate.
- Sensible `max-width` only where it improves readability.
- Responsive gaps.
- Responsive padding.
- Grid/flex behavior based on available space.
- Centering that does not create excessive side whitespace.

The best container width should be determined by the actual content and interaction requirements, not by a single arbitrary global width.

---

# 60. Do Not Use Global Shrinking as a Fix

Do NOT solve the tablet or long-question problems by globally applying rules such as:

```css
transform: scale(...)
```

or:

```css
zoom: ...
```

or extremely small global font-size adjustments.

Do not use a global scale transform for the entire gameplay interface.

The solution must come from proper responsive layout and component sizing.

Likewise, do not reduce the question font below an appropriate readability threshold merely to keep everything on one line.

---

# 61. Pages That Must Be Checked

Audit and test at minimum:

### Student

- `/dashboard`
- `/statistics`
- `/notifications`
- `/profile`
- `/training/addition_subtraction`
- `/training/multiplication`
- `/training/division`
- `/training/abacus`
- `/arena`
- `/arena/match/{id}`
- `/arena/robot/{id}`

### Admin

Review the major responsive pages as well, especially any cards, tables, forms, and dashboards affected by the full-width/container changes.

Do not assume a fix for the student gameplay screens is safe for the rest of the application.

---

# 62. Required Visual QA Matrix

Test the revised layout at representative sizes including:

### Mobile

```text
375 × 667
390 × 844
430 × 932
```

### Tablet

```text
768 × 1024
820 × 1180
834 × 1112
1024 × 1366
```

### Desktop

```text
1366 × 768
1440 × 900
1920 × 1080
2560 × 1440
```

For each size, verify:

- No horizontal overflow.
- No question/answer overlap.
- No clipped mathematical expression.
- No distorted proportions.
- No excessive unused side whitespace.
- No excessively large empty regions.
- Answer controls remain reachable.
- Keypad remains usable.
- Timer/progress remain visible.
- RTL remains correct.
- Touch targets remain usable.

---

# 63. Required Before/After Inspection

Before changing the CSS, identify the actual causes of the tablet layout problems.

Inspect:

- Shared page/container components.
- Training screen wrapper.
- Question component.
- Answer component.
- Numeric keypad component.
- Any AutoFit implementation.
- Fixed width/height rules.
- `max-width` constraints.
- `overflow` behavior.
- Absolute positioning.
- Responsive breakpoints.
- `vh`/`dvh` usage.

Then report:

```text
1. Root cause of poor tablet proportions.
2. Root cause of long-question overlap.
3. Components/files that will be changed.
4. Responsive strategy used.
5. How long-question sizing will be handled.
6. How tablet width/height will be handled.
```

Do not apply unrelated visual changes outside the scope of this task unless required to preserve the shared responsive system.

---

# 64. Implementation Quality Requirements

The final implementation must be:

- Responsive.
- RTL-safe.
- Touch-friendly.
- Stable during viewport resizing.
- Compatible with the existing immersive gameplay architecture.
- Compatible with the existing `useCompactViewport` behavior.
- Compatible with the existing orientation guard.
- Compatible with the existing numeric keypad behavior.
- Free of horizontal overflow.
- Free of question/answer collisions.

Do not duplicate components or create a second responsive system when an existing reusable component can be improved.

Prefer reusable layout primitives and shared styles over page-specific hacks.

---

# 65. Final Visual Goal

The final interface should feel intentionally designed for each device class.

### Mobile

```text
Compact
↓
Readable
↓
Touch-friendly
↓
No accidental zoom
```

### Tablet

```text
More usable width
↓
Balanced proportions
↓
Better spacing
↓
Larger content area without oversized controls
↓
Stable question + answer separation
```

### Desktop

```text
Full viewport usage
↓
Controlled content widths
↓
No giant empty margins
↓
Clear visual hierarchy
```

The most important correction is that the math question must always occupy its own safe visual area and the answer interaction must remain completely separate.

A long question must never enter, cover, overlap, or visually collide with the answer field or keypad on any supported screen size.

