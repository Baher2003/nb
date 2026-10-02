# HOMEPAGE_REDESIGN_SECTIONS.md

## Task

Redesign the **main student homepage of ساحة العباقرة (Genius Arena / e-learn)** so it is no longer crowded.

The new homepage should work as a simple, attractive **main hub** with three clear sections:

1. **التدريبات**
2. **التحديات**
3. **المسابقات** — future feature / coming soon

The goal is not to add more content. The goal is to **organize the existing platform into clear areas** and give every area its own clean, polished page.

---

## 1. Read the Project Before Editing

Before writing code:

- Read `/home/z/my-project/worklog.md`.
- Read `PLATFORM_KNOWLEDGE_BASE.md`.
- Inspect the current homepage implementation.
- Inspect the current routing in `src/lib/url-router.ts`.
- Inspect existing student views under `src/components/views/...`.
- Identify the real existing routes for:
  - Addition/Subtraction training
  - Multiplication training
  - Division training
  - Abacus training
  - Friend/PVP challenge
  - AI challenge
- Check whether any competition feature already exists.

Do not assume routes or components. Reuse the actual project structure.

The application uses the existing SPA catch-all route:

`src/app/[[...slug]]/page.tsx`

Do **not** create:

`src/app/page.tsx`

---

# 2. New Homepage Structure

The homepage should become a **clean section-selection page**.

## Main layout

Use this hierarchy:

### Header

Keep the existing site header, branding, account access, language handling, and RTL behavior.

Do not replace the global navigation unless necessary.

### Welcome area

Keep it short.

Example:

**أهلاً بيك 👋**

**جاهز تبدأ؟**

Short supporting text:

**اختار اللي حابب تعمله النهارده.**

Do not place long statistics, multiple banners, or large dashboards here.

### Main sections

Display three large cards:

---

### Section 1 — التدريبات

Title:

**التدريبات**

Description:

**طوّر مهاراتك الحسابية وتدرّب على مستواك.**

Action:

**ابدأ التدريب**

Visual direction:
- learning/practice icon
- calm educational feeling
- clear primary action
- visually inviting but not childish
- no excessive animation

---

### Section 2 — التحديات

Title:

**التحديات**

Description:

**اختبر سرعتك ودقّتك في تحديات ممتعة.**

Action:

**ادخل التحديات**

Visual direction:
- competition/challenge icon
- energetic but still consistent with the main design
- visually distinct from Training without creating a new color system

---

### Section 3 — المسابقات

Title:

**المسابقات**

Description:

**تجربة تنافسية جديدة هتكون متاحة قريبًا.**

Status:

**قريبًا**

Do not pretend that competitions are currently available.

Action can be:

**قريبًا**

or a controlled entry into a Coming Soon page.

The card should look intentionally designed, not like a broken or disabled component.

---

# 3. Important Homepage Rule

The homepage must NOT contain all available platform features at once.

Remove the feeling of:

- too many cards
- too many buttons
- too many statistics
- too many settings
- too many shortcuts
- repeated information
- long lists
- multiple competing sections

The homepage should answer one question:

**"What do I want to do now?"**

The student should immediately understand:

**تدريب → ادخل التدريبات**

**تحدي → ادخل التحديات**

**مسابقات → قريبًا**

---

# 4. Training Section Page

When the student opens **التدريبات**, open a dedicated Training page.

This page must be much cleaner than the current crowded homepage.

## Training page layout

Header:

**التدريبات**

Subtitle:

**اختار نوع التدريب اللي عايز تبدأ بيه.**

Then display only the currently supported training types.

### Training cards

#### الجمع والطرح
Description:

**تدرّب على الجمع والطرح وطوّر سرعتك في الحساب.**

Action:

**ابدأ**

#### الضرب
Description:

**تدرّب على عمليات الضرب بطريقة منظمة وتدريجية.**

Action:

**ابدأ**

#### القسمة
Description:

**طوّر مهارتك في القسمة وحل المسائل بسرعة ودقة.**

Action:

**ابدأ**

#### الأباكس / العداد

Show this only when the current implementation actually supports it.

Description can be short and natural.

Action:

**ابدأ**

## Training page rules

Do not put every setting on this page.

Do not place:

- digit controls
- timer controls
- advanced generation settings
- long statistics
- detailed history
- large configuration forms

on the section landing page.

The Training landing page is only for choosing the training type.

The existing training screen should handle its own settings.

Navigation flow should feel like:

**Homepage → التدريبات → نوع التدريب → إعدادات/بدء التدريب**

---

# 5. Challenges Section Page

When the student opens **التحديات**, show a dedicated challenge hub.

Header:

**التحديات**

Subtitle:

**اختبر نفسك في مواجهة منافس.**

Then show the actual challenge modes that exist in the current code.

## Friend Challenge

Title:

**تحدي صديق**

Description:

**واجه صديقك وشوف مين يقدر يحل أسرع وأدق.**

Action:

**تحدي صديق**

Only show this if the current PVP/friend functionality is available to the student.

## AI Challenge

Title:

**تحدي الذكاء الاصطناعي**

Description:

**اختبر مستواك في مواجهة منافس ذكي.**

Action:

**ابدأ التحدي**

Only show this if the current AI opponent functionality is available and enabled.

## Challenge page rules

Do not move the actual live match interface into this page.

Keep existing:

- matchmaking
- waiting rooms
- live challenge screen
- answer interface
- challenge timer
- wager/points behavior
- result screen

in their current flows.

This page is only the **entry point**.

---

# 6. Competitions Section — Future

Competitions are intentionally planned for a later stage.

Do not build the actual competition system in this task.

If there is no real competitions feature yet:

Create a beautiful Coming Soon page.

Example:

**المسابقات قريبًا 🚀**

**بنجهزلك تجربة منافسات أكبر وأمتع.**

**هتظهر هنا أول ما تكون جاهزة.**

Keep it visually polished and consistent.

Do NOT create fake:

- competitions
- event dates
- prizes
- rankings
- winners
- registrations
- countdowns
- reward amounts
- schedules

Do not create database tables or APIs for the future competition system as part of this task.

If the repository already contains a functional competition feature, inspect it first and integrate it instead of replacing it.

---

# 7. Visual Design

The most important visual requirement:

## Less crowded, more organized.

Use the existing **Calm Educational Play** visual direction from the project.

Maintain the existing brand identity.

Use:

- generous whitespace
- strong visual hierarchy
- consistent cards
- moderate rounded corners
- subtle shadows
- restrained borders
- clear typography
- balanced spacing
- simple icons
- subtle hover/focus transitions

Avoid:

- excessive gradients
- excessive glow
- too many colors
- giant decorative illustrations
- large empty hero areas
- unnecessary badges
- excessive animation
- dense dashboards
- card overload

The page should look modern and professional while remaining suitable for children.

---

# 8. Design Relationship Between Pages

The three section pages must look like they belong to the same platform.

Use a shared page pattern:

**Page Header**
→ title
→ one short subtitle
→ grid of cards
→ clean spacing
→ optional back navigation

Do not design three completely different visual styles.

Instead:

- Training can feel educational.
- Challenges can feel energetic.
- Competitions can feel exciting/future-oriented.

But all three must still use the same typography, spacing system, card language, RTL behavior, and brand identity.

---

# 9. Card Design

Cards should be visually meaningful but not overloaded.

Each card should normally contain:

1. Icon
2. Title
3. Short description
4. One action

Avoid adding several buttons to the same card.

Do not show long text.

Keep card heights consistent inside the same grid.

Make the clickable area clear.

Add visible hover and keyboard focus states.

---

# 10. Desktop Layout

On desktop:

- The three homepage section cards should preferably appear as a balanced 3-column composition.
- Give all three cards similar visual importance.
- Keep generous horizontal and vertical spacing.
- Do not make one card dramatically larger unless there is a real product reason.
- Keep the main content centered with a reasonable maximum width.

The result should feel like a modern learning platform landing page, not an admin dashboard.

---

# 11. Tablet Layout

On tablet:

- Allow cards to wrap cleanly.
- Maintain comfortable card widths.
- Preserve readable Arabic text.
- Prevent cramped buttons.
- Keep spacing consistent.

---

# 12. Mobile Layout

On mobile:

- Stack the three main sections vertically.
- Use large touch-friendly cards.
- Keep actions easy to tap.
- Minimum interactive target: approximately 44 × 44 CSS pixels.
- Avoid horizontal scrolling.
- Avoid text clipping.
- Avoid cards becoming excessively tall.

Recommended order:

1. التدريبات
2. التحديات
3. المسابقات

Training should naturally appear first because it is the primary learning entry point.

---

# 13. RTL and Arabic Quality

The entire experience must remain RTL.

Check:

- title alignment
- card alignment
- icons
- arrows
- navigation
- spacing
- button placement
- breadcrumbs/back navigation
- text wrapping

Arabic text must not break visually.

Do not use awkward machine-translated Arabic.

Keep the wording natural and student-friendly.

---

# 14. Navigation

Use the existing router and URL system.

Do not hardcode a second navigation architecture.

Use the actual existing route for each feature.

For example, conceptually:

Homepage
→ `/training`
→ `/challenges`
→ `/competitions`

But **do not assume these exact routes** until the repository is inspected.

Use existing route helpers where they exist.

Every section should have a clean way to return to the homepage.

---

# 15. Preserve Existing Functionality

This is a UI/information-architecture redesign.

Do NOT change:

- math generation
- Rules Engine
- question correctness
- scoring
- training points
- Arena points
- PVP logic
- AI opponent logic
- wagers
- match state
- idempotency
- server authority
- answer security
- database schema
- authentication
- role permissions

unless a tiny change is absolutely required only to connect the new navigation.

Do not create duplicate generators or duplicate game flows.

---

# 16. Security Constraints

Keep all existing security rules.

In particular:

- Never expose answer keys to the client during a live challenge.
- Preserve server-side authority.
- Preserve idempotency.
- Preserve challenge state protections.
- Do not bypass role checks.
- Do not expose admin/trainer pages through student navigation.

---

# 17. Accessibility

Verify:

- semantic headings
- keyboard navigation
- visible focus states
- accessible button/link labels
- icons have appropriate accessible treatment
- coming-soon status is not conveyed by color alone
- sufficient text contrast
- reduced-motion behavior where supported

---

# 18. Error / Empty / Loading States

Every new section page must have clean states.

Do not leave:

- blank white pages
- broken cards
- undefined labels
- loading forever
- empty grids with no explanation

For competitions, the empty state should intentionally be the designed Coming Soon experience.

---

# 19. Testing

After implementation:

### Required checks

Run:

`bun run lint`

Run when available/applicable:

`scripts/comprehensive-test.sh`

`scripts/new-mods-test.sh`

Run relevant browser/E2E tests.

Check the homepage and all three section pages on:

- desktop
- tablet
- mobile

Verify:

- navigation
- RTL
- responsive behavior
- no horizontal overflow
- no clipped Arabic text
- no broken links
- no permission regression
- no regression to existing training
- no regression to existing PVP/AI challenge flows

If a test cannot be run, state exactly why.

Never claim a test passed unless it was actually executed.

---

# 20. Worklog

At the end:

Append a clear work record to:

`/home/z/my-project/worklog.md`

Include:

- what was changed
- files/components changed
- routes used
- competition status
- tests executed
- test results
- unresolved issues

---

# 21. Definition of Done

The redesign is complete when:

- The homepage is no longer crowded.
- The student immediately sees three clear sections:
  - التدريبات
  - التحديات
  - المسابقات
- Training has its own clean entry page.
- Challenges have their own clean entry page.
- Competitions has an honest future/Coming Soon experience unless a real competition feature already exists.
- Existing functionality and routes continue working.
- The visual design matches the existing site.
- Mobile, tablet, and desktop layouts are clean.
- RTL is correct.
- The navigation feels simple and intentional.

The final result should feel like:

**Homepage = choose what you want to do**

not:

**Homepage = show every feature the platform has.**

---

# 22. Final Report Required From the Agent

At completion, report:

1. Exact files modified.
2. Exact routes used for Training, Challenges, and Competitions.
3. What already existed versus what was newly created.
4. Whether Competitions is implemented or Coming Soon.
5. Responsive screens checked.
6. Exact test commands executed.
7. Results of each test.
8. Any unresolved issue.

Do not report assumptions as facts.
