# Website Modifications & Development Requirements

> **Document purpose:** This is the living requirements document for the website. Every new modification, correction, feature request, or UI/UX requirement should be added here and kept up to date.
>
> **Important for the Agent:** Before making any future change, read this file first. Preserve all previously approved requirements unless a newer requirement explicitly replaces or changes one of them.

---

## 1. Project Change Management & Living Document Rules

This file is a **permanent, continuously updated project requirements document** and must remain with the project as the main reference for all future website modifications and development work.

- **Single Source of Truth:** Treat this document as the primary source of truth for all requested website changes, corrections, improvements, and implementation requirements.
- **Always Read First:** Before making any future change, the Agent must read this file first and understand all currently active requirements.
- **Keep It Continuously Updated:** Every new modification or development request provided by the owner must be added to this same file. Do not create a separate requirements file for each new request unless explicitly instructed.
- **Preserve Previous Requirements:** Do not delete or silently weaken an existing requirement. Older requirements remain active unless they are explicitly replaced, removed, or marked obsolete.
- **Handle Conflicts Explicitly:** If a new request conflicts with an older requirement, keep the historical requirement for reference, mark it as **REPLACED**, and clearly define the new requirement as the active version.
- **Track Development Progress:** Each new requirement should have a clear status such as **NEW**, **IN PROGRESS**, **COMPLETED**, or **REPLACED**.
- **Maintain Change History:** Every meaningful modification should be recorded in the change history with its date, description, and current status.
- **Update Before/Alongside Implementation:** When a new request is received, update the requirements document before or together with the code changes so the file always reflects the latest agreed state of the project.
- **No Unapproved Destructive Changes:** Do not remove existing features, business logic, database structures, working components, or functionality unless the owner explicitly requests the removal or replacement.
- **Inspect Before Editing:** Before changing code, inspect the current project structure and understand how the existing implementation works. Do not guess about architecture or behavior.
- **Prefer Targeted Changes:** Make maintainable, focused changes instead of unnecessary rewrites. Refactoring is allowed when needed for correctness, maintainability, or consistency, but existing functionality must be preserved.
- **Regression Protection:** After every significant change, verify that previously completed requirements still work and that the new change has not introduced regressions.
- **Latest Version Wins:** The latest explicitly approved requirement is the active requirement when it replaces an older one. The document should always make the currently active behavior easy to identify.

### Permanent Update Workflow

For every future modification, follow this workflow:

```text
New Request
    ↓
Read WEBSITE_MODIFICATIONS.md
    ↓
Identify related existing requirements
    ↓
Determine: New / Extension / Replacement
    ↓
Update this same file
    ↓
Implement the code changes
    ↓
Test the requested change
    ↓
Regression-check important existing features
    ↓
Update status and Change History
```

**Important:** This file is not a one-time specification. It is a **living project document** that must continue to evolve with the website throughout development. Every future change should be reflected here so that the Agent always has the latest complete picture of the required system behavior.

---

# 2. Current Active Requirements

## 2.1 Independent URL/Route for Every Page

### Problem

The current website behaves too much like a single-page interface where multiple sections/pages are connected to one main URL instead of behaving like independent website pages.

### Required behavior

Every real page or major section must have its own unique route/URL.

The website must behave like a normal professional website where:

- Each page has its own unique URL.
- The URL changes when navigating to a different page.
- The URL clearly identifies the current page.
- A page URL can be copied and shared.
- Opening a copied URL directly must open the corresponding page instead of always opening the homepage/dashboard.
- Nested pages must also have their own routes.
- Browser history must correctly track navigation between pages.

### Example structure

```text
/
/dashboard
/training
/addition
/subtraction
/multiplication
/multiplication/table-2
/multiplication/table-3
/challenges
/profile
/settings
```

These are examples only. Apply the same routing principle to **all actual pages in the project**.

### Implementation requirements

- Use the appropriate routing system for the framework currently used by the project.
- Do not fake page separation by only changing content while keeping the same route.
- Avoid unnecessary full-page reloads during normal internal navigation.
- Ensure direct URL access and deep linking work correctly.
- Ensure every defined route resolves to the intended page/component.

---

## 2.2 Browser and Mobile Back Navigation

The Back action must behave like a normal website.

When the user presses:

- Browser Back.
- Mobile device Back.
- Supported back gestures.

The user must return to the **actual previous page in navigation history**.

### Example

```text
Dashboard
   ↓
Training
   ↓
Multiplication
   ↓
Table 5
```

Pressing Back from `Table 5` must produce:

```text
Table 5 → Multiplication
```

Then:

```text
Multiplication → Training
```

And so on.

### Required safeguards

- Back must not unexpectedly close/exit the website.
- Back must not always redirect to the homepage.
- Do not create unnecessary duplicate history entries.
- Do not force the user to press Back multiple times because of artificial navigation states.
- Internal navigation must integrate correctly with browser history.

---

## 2.3 Refresh Must Preserve the Current Page

When the user is on a specific page and presses Refresh/Reload, the same page must be reloaded.

### Example

If the current URL is:

```text
/multiplication/table-5
```

Refreshing the page must keep the user on:

```text
/multiplication/table-5
```

It must **not** redirect the user to:

```text
/
```

or:

```text
/dashboard
```

unless the application explicitly requires authentication and the user is not authenticated.

### Required behavior

- Refresh must reload the current route.
- Direct access to any valid route must work.
- Deep links must not produce an incorrect redirect.
- Valid routes must not become 404 pages after refresh.
- Server/hosting configuration must support client-side routing when required by the framework.

---

# 3. UI / UX / Responsive Design Overhaul

## 3.1 Current UI Problems

The current website has layout and visual consistency problems, including but not limited to:

- Primitive/unrefined page layout.
- Inconsistent dimensions.
- Poor spacing.
- Components overlapping each other.
- Content extending outside containers.
- Buttons or controls extending outside their intended boxes.
- Incorrect element sizing.
- Horizontal overflow.
- Vertical layout problems.
- Text breaking the layout.
- Elements being clipped or hidden.
- Layout breaking at smaller screen sizes.
- Reversed/mirrored/misconfigured elements.
- RTL/LTR alignment issues.
- Inconsistent spacing and sizing between pages.

These issues should be identified and corrected across the whole website, not only on one page.

---

## 3.2 Professional and Consistent Layout

The website should have a modern, professional, and consistent visual system.

Review and standardize:

- Containers.
- Page widths.
- Padding.
- Margins.
- Gaps.
- Typography.
- Button dimensions.
- Cards.
- Forms.
- Inputs.
- Tables.
- Modals/dialogs.
- Navigation.
- Sidebar.
- Headers.
- Footers.
- Icons.
- Images.
- Charts/statistics areas.
- Training interfaces.
- Challenge/game interfaces.

The design should feel like one coherent product rather than separate pages built with different sizing rules.

---

## 3.3 Full Responsive Design

The website must adapt correctly to different screen sizes and browsers.

At minimum, test the UI on:

```text
Small Mobile
Mobile
Tablet
Laptop
Desktop
Large Desktop
```

The design must not depend on one fixed screen size.

### Responsive requirements

- No unintended horizontal scrolling.
- No components outside the viewport.
- No overlapping controls.
- No clipped text.
- Long text must wrap or truncate appropriately.
- Buttons must remain usable on small screens.
- Forms must remain readable and usable.
- Tables must have a practical mobile behavior (responsive layout, controlled scrolling, or another appropriate approach).
- Cards must resize appropriately.
- Navigation must adapt to the available width.
- Sidebar behavior must be appropriate for mobile and desktop.
- Modals must fit within the viewport.
- Charts/statistics must resize without breaking their containers.
- Training and challenge interfaces must remain usable on mobile.

---

## 3.4 RTL / LTR and Mirroring Issues

The project must be reviewed for Arabic RTL and English LTR behavior.

Correct any cases where:

- Elements are mirrored incorrectly.
- Icons appear on the wrong side.
- Text alignment is reversed.
- Padding/margins are applied to the wrong direction.
- Buttons or controls are visually reversed.
- Direction-sensitive layouts break when switching language.

The final UI must behave correctly in both supported directions.

---

# 4. Compatibility and Stability Requirements

The website should behave consistently across modern browsers and common devices.

Review at least:

- Desktop browsers.
- Mobile browsers.
- Different viewport widths.
- Different pixel densities where practical.
- Arabic and English layouts if both are supported.

Avoid browser-specific hacks unless they are genuinely necessary and documented.

---

# 5. Preserve Existing Functionality

While implementing these changes:

- Do not remove existing features.
- Do not delete existing pages unless explicitly requested.
- Do not remove data or database functionality.
- Do not change business logic unnecessarily.
- Do not break authentication/authorization.
- Do not break existing API calls.
- Do not break existing training/challenge functionality.
- Do not replace working components without a clear technical reason.

Any structural refactor must preserve the existing behavior unless a newer requirement intentionally changes it.

---

# 6. Validation Checklist

After implementation, verify all of the following:

### Routing

- [ ] Every major page has a unique URL.
- [ ] Nested pages have unique URLs.
- [ ] URL changes correctly during navigation.
- [ ] Direct URL access opens the correct page.
- [ ] Deep linking works.

### History

- [ ] Browser Back returns to the real previous page.
- [ ] Browser Forward works correctly.
- [ ] Mobile Back works correctly.
- [ ] No unnecessary duplicate history entries are created.

### Refresh

- [ ] Refresh keeps the user on the current page.
- [ ] Refresh does not unexpectedly send the user to the homepage/dashboard.
- [ ] Refreshing nested routes works.
- [ ] Server/hosting routing is configured correctly for the application's routing strategy.

### UI

- [ ] No overlapping elements.
- [ ] No content outside containers.
- [ ] No unintended horizontal overflow.
- [ ] No clipped or hidden important content.
- [ ] Consistent spacing.
- [ ] Consistent sizing.
- [ ] Buttons and controls stay inside their intended areas.
- [ ] Forms work correctly on small screens.
- [ ] Tables remain usable.
- [ ] Modals fit the viewport.

### Responsive behavior

- [ ] Small Mobile.
- [ ] Mobile.
- [ ] Tablet.
- [ ] Laptop.
- [ ] Desktop.
- [ ] Large Desktop.

### Language direction

- [ ] Arabic RTL tested.
- [ ] English LTR tested.
- [ ] No incorrect mirroring.
- [ ] Icons and directional controls are correctly positioned.

---

# 7. Future Modifications Log

> Add every new request below. Keep entries ordered by date. Do not erase older entries; mark them as REPLACED when necessary.

## Change Template

```markdown
## [DATE] — [Short Change Title]

### Status
- NEW / IN PROGRESS / COMPLETED / REPLACED

### Requirement
[Describe exactly what needs to be changed.]

### Current behavior
[Describe the current behavior/problem if known.]

### Required behavior
[Describe the exact expected behavior.]

### Affected pages/files
[List known pages/files/components, if applicable.]

### Constraints
[List anything that must not be changed.]

### Validation
[List the tests that must pass after implementation.]
```

---

# 8. Agent Instructions for Future Changes

Whenever a new change request is provided:

1. Read this entire document before modifying the project.
2. Identify whether the new request is new, an extension of an existing requirement, or a replacement for an older requirement.
3. Update this document before or together with the code change.
4. Preserve all active requirements.
5. Do not silently remove or weaken previous requirements.
6. Implement the change in a way that remains compatible with the routing, navigation, responsive, RTL/LTR, and stability requirements above.
7. Run a regression check against the validation checklist after significant changes.
8. Report what was changed and identify any requirement that was intentionally replaced.

---

# 9. Change History

| Date | Change | Status |
|---|---|---|
| 2026-09-28 | Initial requirements: independent routes, browser/mobile Back navigation, refresh-current-page behavior, responsive UI overhaul, layout fixes, RTL/LTR corrections, and validation rules. | ACTIVE |

