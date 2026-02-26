# GoalPost — Goals and Dashboard Screen Specifications

File: docs/design/02-userflow/screens/goal-screens.md
Scope: Goals domain and Dashboard-related surfaces for MVP
Traceability:
- SRS Functional: FR-1..5, FR-6..11 (context surfacing), FR-12..14 (review integrations via prompts), FR-15..17 (progress surfacing), FR-23..26 (AI assist in creation, optional), FR-27..29 (templates), FR-30..32 (attachments, export – shown as entry points only)
- SRS Non-Functional: NFR-1..7, 9..12
- Core Use Cases: UC-1, UC-2, UC-3 (context), UC-4/5 (prompts), UC-6 (links), UC-7/8 (AI assist links), UC-9/10 (links)

Conventions:
- Mobile-first layouts; responsive adaptations for tablet/web noted per screen.
- All destructive actions require confirmation.
- Offline-first: offline state does not block core interactions; AI and remote-only actions disabled gracefully.
- Privacy-first: sharing/export off by default; redaction defaults apply elsewhere.

Global Components Referenced
- App Bar: title, optional search, overflow menu
- FAB: context-sensitive primary action
- Banner: info/offline/warning/error
- Card: summary, list items with metadata and quick actions
- Bottom Sheet / Modal: editors, pickers, confirmations
- Date/Time Picker: standardized schedule picker
- Empty State: illustration-free (text + CTA) for fast load
- Skeletons: list/chart placeholders under 1s

Global Performance Targets (NFR)
- Interaction response: <150ms
- First contentful render: <1s for typical lists (≤100 items)
- Heavy recomputations (e.g., progress aggregation): background-thread, UI refresh <1s
- AI operations: show non-blocking loader, cancelable; timeout target 6–8s

Accessibility (A11y) Defaults
- WCAG AA contrast; hit targets ≥44dp; keyboard-focusable controls; semantic roles and labels
- Alt text required for images added by user; live regions for toasts and async updates

---

## SCR-DASH — Dashboard / Home

Purpose
- Daily command center: surface “Today” focus, overdue/soon items, quick entry for Goals/Routines, and shortcuts to Reviews and Progress.

Primary Data
- Today’s due/overdue routines (FR-6..8, FR-9..11)
- Today’s due/overdue milestones (FR-9..11)
- Quick stats: active goals, avg milestone %, longest streak (FR-15)
- Prompts: Morning/Night review windows (FR-12..14)
- Entry points: New Goal, Progress, Notification Center

Entry
- App launch; bottom nav “Home”; post-auth default route

Layout (Mobile)
- App Bar: “GoalPost”
  - Actions: Search (future), Overflow (Settings/Data, About)
- Top Banners (conditional):
  - Offline banner (red)
  - Morning/Night Review prompt within window (blue)
- Section: Today at a Glance
  - Cards: Due Now, Due Today, Overdue (counts; tap → filtered lists)
  - Streak Risk (e.g., routines likely to break)
- Section: Priority Shortcuts
  - Button: Start Morning Review (if in window)
  - Button: Open Today’s Checklist
  - Button: Notification Center
- Section: My Goals (horizontal scroll or 2-column grid)
  - Goal Card: title, milestone %, upcoming milestone, linked streak chips
  - Tap → SCR-GOAL-DETAIL
- FAB: “New Goal” → SCR-GOAL-NEW-ENTRY

States
- Empty: “Let’s get started — create your first goal” (CTA → New Goal, or “Browse Templates”)
- Loading: skeleton cards
- Error: inline card with retry
- Offline: banner + disable AI-dependent CTAs (still allow New Goal manually)

Key Actions
- New Goal (UC-1)
- Open Today’s Checklist (SCR-CHECKLIST-TODAY)
- Open Notification Center (SCR-NOTIFICATION-CENTER)
- Start Morning/Night Review (UC-4/UC-5)
- Tap Goal Card → SCR-GOAL-DETAIL
- View Progress → link to Progress Hub (UC-6)

Validation/Constraints
- None specific; lists filter by “due today/overdue” by local time zone (FR-33)
- Caps on number of home sections visible; overflow via “See all”

Instrumentation
- Event: dashboard_viewed { has_due, has_overdue, within_morning_window }
- Event: cta_tapped { type: “new_goal|checklist|notification_center|morning_review|night_review|progress” }

A11y
- Landmarks: header, main, complementary
- Dynamic counts announced via polite live region

---

## SCR-CHECKLIST-TODAY — Today’s Checklist

Purpose
- Execute today’s prioritized routines and milestones post-morning review or ad-hoc selection.

Primary Data
- Today’s selected priorities (from Morning Review or manual)
- Due/Overdue routines/milestones with quick actions

Entry
- From Dashboard shortcuts; after Morning Review; or via notification tap

Layout
- App Bar: “Today”
  - Actions: Edit selection (opens multi-select source list/modal)
- Stacked List (grouped):
  - Routines: item row with title, due time, streak chip, actions [Complete, Snooze, Dismiss]
  - Milestones: item row with title, due time, % ring (future), actions [Mark Done, Edit Time]
- Bottom Summary: X completed / Y planned today

States
- Empty: “No priorities selected” (CTA: Pick from due items)
- Offline: banner; all local actions work

Key Actions
- Complete routine (updates streak) (FR-7)
- Mark milestone done (updates % complete) (FR-15)
- Snooze/Dismiss reminders (FR-10)
- Edit time (schedule picker) (FR-9..11)

Validation/Constraints
- Prevent double-completion; offer undo for 5s
- Editing time enforces “today” unless user overrides

Instrumentation
- Event: checklist_viewed { planned_count }
- Event: checklist_action { type: complete|snooze|dismiss|mark_done, entity: routine|milestone }

A11y
- Each row focusable; actions exposed via accessible actions or sub-buttons
- Completion announced via live region

---

## SCR-NOTIFICATION-CENTER — Reminders

Purpose
- Consolidated list for upcoming and overdue reminders with bulk controls.

Primary Data
- Reminders for routines/milestones: due_at, status

Entry
- From Dashboard or system notification; bottom nav optional

Layout
- App Bar: “Reminders”
- Tabs: Overdue | Today | Upcoming
- List rows: title, entity type, due time, quick actions [Open, Snooze, Dismiss]
- Bulk toolbar (when multi-selected): Snooze, Dismiss

States
- Empty: “You’re all caught up”
- Offline: banner; still usable

Key Actions
- Open related entity
- Snooze/Dismiss single or bulk (FR-10..11)

Validation/Constraints
- Snooze presets: +10m, +1h, Tonight; custom via picker

Instrumentation
- Event: notifications_viewed { tab }
- Event: reminder_action { action, count }

A11y
- Selection affordances visible for keyboard; announce tab changes

---

## SCR-GOAL-LIST — All Goals

Purpose
- Browse/search/sort user’s goals; quick actions; create new.

Primary Data
- All goals with key metrics (milestone %), tags, domain, priority

Entry
- From Dashboard “See all” or bottom nav “Goals”

Layout
- App Bar: “Goals”
  - Search field (inline expand)
  - Filter/Sort (domain, status, priority; sort by % complete, updated_at)
- List/Grid of Goal Cards
  - Title, domain chip, milestone %, upcoming milestone, status
  - Overflow: Edit, Archive, Delete (confirm)
- FAB: “New Goal” → SCR-GOAL-NEW-ENTRY

States
- Empty: “No goals yet” (New Goal / Browse Templates)
- Loading skeletons
- Offline: banner (browse works)

Key Actions
- New Goal (UC-1)
- Open Goal Detail (UC-2)
- Archive/Delete with confirmation

Validation/Constraints
- Prevent deleting goals with active dependencies without confirm

Instrumentation
- Event: goal_list_viewed { count }
- Event: goal_card_action { action }

A11y
- Search labeled; card headings properly leveled; controls tab-order logical

---

## SCR-GOAL-NEW-ENTRY — New Goal Entry

Purpose
- Choose creation path: From Template (default fast path) or From Scratch (with optional AI).

Entry
- From FAB (Dashboard/Goals)

Layout
- App Bar: “New Goal”
- Two primary cards:
  - From Template (Recommended) → SCR-TEMPLATES
  - From Scratch → SCR-GOAL-FORM
- Info text: “You can edit everything before saving.”

States
- Offline: template path uses cached templates only; AI-only hints hidden

Key Actions
- Select path; cancel returns to previous

Instrumentation
- Event: goal_new_entry_opened
- Event: goal_new_path_selected { path }

A11y
- Both cards are large, fully focusable with descriptive labels

---

## SCR-TEMPLATES — Template Library

Purpose
- Browse, preview, and apply curated templates (FR-27..29; AC-1).

Primary Data
- Template catalog (local bundle; cache)

Entry
- From New Goal Entry (Template path) or Goals empty state

Layout
- App Bar: “Templates”
  - Filter by Category (Fitness, Finance, Education, Diet, Beauty, Health)
- Template List (cards):
  - Title, category, brief summary, includes: plans/milestones/routines counts
  - Actions: Preview, Use Template, Customize First
- Inline Preview (expandable) or dedicated panel:
  - Shows prefilled structures and guidance notes

States
- Empty (unlikely in MVP): “Templates coming soon”
- Offline: works with local set

Key Actions
- Use Template → SCR-GOAL-REVIEW (template draft)
- Customize First → SCR-GOAL-REVIEW (all fields editable)

Validation/Constraints
- Applying template creates a draft (not persisted until confirm)

Instrumentation
- Event: template_list_viewed { category }
- Event: template_applied { template_id }

A11y
- Ensure template counts are read with labels; preview sections have headings

---

## SCR-GOAL-FORM — Create Goal (Scratch)

Purpose
- Capture goal basics and add initial plan/milestones manually (or invoke AI).

Primary Data
- Goal: title, description, domain/tags, priority
- Plan: default “Main Plan” auto-created unless user adds named plan
- Milestones: titles, optional descriptions/targets

Entry
- From New Goal Entry (Scratch path)

Layout
- App Bar: “New Goal (Scratch)”
- Section: Goal Basics
  - Title (required)
  - Description (optional)
  - Domain (select/tag)
  - Priority (Low/Med/High)
  - Tags (chips)
  - Button: Get AI Suggestions (if enabled/online) → SCR-GOAL-AI-SUGGEST
- Section: Plan & Milestones
  - Auto plan chip (“Main Plan”) editable or “Add Plan” → SCR-PLAN-EDITOR (modal)
  - Milestone Quick Add (title input + Add)
  - List of current milestones with edit/delete
- Footer CTAs:
  - Review (enabled when Title present and ≥1 milestone recommended)
  - Cancel (discard draft)

States
- Validation errors inline (Title required)
- Offline: AI button disabled with info tooltip

Key Actions
- Add milestones; invoke AI; proceed to Review

Validation/Constraints
- Soft-nudge to create ≥2 milestones (not hard block)

Instrumentation
- Event: goal_form_opened
- Event: goal_form_ai_invoked { available }
- Event: goal_form_review_clicked { milestone_count }

A11y
- Field labels tied to inputs; error text with aria-describedby

---

## SCR-GOAL-AI-SUGGEST — AI Milestone Suggestions (Optional)

Purpose
- Present AI-proposed milestone candidates with rationale/confidence; accept/edit before mapping to plan and saving (FR-23, FR-26; AC-6).

Entry
- From SCR-GOAL-FORM “Get AI Suggestions” or SCR-GOAL-DETAIL context

Layout
- App Bar: “AI Suggestions”
  - Secondary: Refine Prompt (opens side panel: count 3–5, time horizon)
- Results Area:
  - List of Suggestion Cards: title, description (optional), rationale, confidence chip
  - Per-card actions: Accept, Edit (inline), Reject
- Footer CTAs:
  - Next: Map to Plan (enabled when ≥1 accepted)
  - Regenerate / Cancel

States
- Loading: progress with cancel; timeout ~8s then show Retry/Manual path
- Error: banner with retry
- Offline/disabled: screen not accessible; handled by entry

Key Actions
- Accept/Edit/Reject; Regenerate; Next → Plan Map (part of Review or separate step)

Validation/Constraints
- Accepted items persist only after final confirm in Review

Instrumentation
- Event: ai_suggest_opened { context: goal|milestone }
- Event: ai_suggest_selection { accepted, rejected, edited }

A11y
- Suggestion card structure semantic; edits accessible; focused feedback on accept

---

## SCR-GOAL-REVIEW — Goal Preview & Confirm

Purpose
- Final review of the new goal (from template or scratch/AI) before persisting.

Primary Data
- Goal basics; Plan(s); Milestones; optional Routines (if template)

Entry
- From SCR-TEMPLATES “Use/Customize First” or SCR-GOAL-FORM “Review”

Layout
- App Bar: “Review Goal”
- Summary sections (expand/collapse):
  - Goal Basics (edit inline)
  - Plans (list), each with milestones
  - Routines (if any; cadence/time editable inline)
- Footer CTAs:
  - Create Goal (primary)
  - Back (to previous)
  - Cancel (discard draft)

States
- Validation highlights missing Title; milestone rows with empty titles
- Offline: fully functional

Key Actions
- Inline edits; confirm create → persist entities → navigate to SCR-GOAL-DETAIL

Validation/Constraints
- Requires Title and ≥1 Plan; recommend ≥2 milestones

Instrumentation
- Event: goal_review_viewed { plan_count, milestone_count }
- Event: goal_created { method: template|scratch|scratch_ai }

A11y
- Sections as headings; error summaries navigable

---

## SCR-GOAL-DETAIL — Goal Detail

Purpose
- Central hub for a specific goal: overview, analytics, plans/milestones, routines linkages, and actions (edit, add, AI assist).

Primary Data
- Goal entity; Plans/Milestones; linked Routines; progress metrics (FR-5, FR-15)

Entry
- From Dashboard card, Goal List, post-create redirect

Layout (Tabbed or Sectioned)
- App Bar: Goal Title
  - Actions: Edit Goal, More (…) [Archive, Delete (confirm)]
- Header Summary
  - Milestone % ring; next upcoming milestone; tags; domain; priority
  - Buttons: Add Milestone, Add Routine (pre-linked), Get AI Suggestions
- Section: Plans
  - Accordion per Plan
  - Inside: Milestone list
    - Row: title, target date (optional), status toggle (Pending/Completed), overflow (Edit → SCR-MILESTONE-EDITOR, Delete confirm)
    - Quick Add row (SCR-MILESTONE-QUICK-ADD inline)
- Section: Linked Routines (chips with streaks; tap to open routine editor list)
- Footer (mobile): CTA shortcuts (Add Milestone / Get AI Suggestions)

States
- Empty (no milestones): “Add your first milestone” CTA
- Offline: AI button disabled

Key Actions
- Add/Edit/Delete milestones; mark complete; add routine; invoke AI (UC-2, UC-7)
- Navigate to analytics tab (UC-6 link)

Validation/Constraints
- Delete milestone → confirm; remove reminders

Instrumentation
- Event: goal_detail_viewed { milestone_pct, milestone_count }
- Event: milestone_action { type: add|edit|delete|complete }

A11y
- Milestone toggle accessible with clear state; analytics ring has text equivalents

---

## SCR-PLAN-EDITOR — Plan Editor

Purpose
- Create or edit a plan under a goal.

Entry
- From Goal Form (Add Plan) or Goal Detail (Add/Edit Plan)

Layout
- Modal/Sheet: “Plan”
  - Fields: Title (required), Description (optional)
  - Save / Cancel

States
- Validation: Title required
- Offline: fully supported

Instrumentation
- Event: plan_saved { is_new }

A11y
- Proper focus trap; labels bound to inputs

---

## SCR-MILESTONE-EDITOR — Milestone Editor

Purpose
- Create/edit milestone details, target dates, and success criteria; mark complete.

Entry
- From Milestone row “Edit” or new from Quick Add continuation

Layout
- Modal/Sheet: “Milestone”
  - Fields:
    - Title (required)
    - Description (optional)
    - Target Date (Date picker; optional)
    - Success Criteria (optional text or checklist seed)
    - Status: Pending/Completed
  - Save / Cancel
  - Secondary: Delete (confirm)

States
- Validation: Title required; invalid date corrected with message
- Offline: fully supported

Key Actions
- Save updates; if Completed, record timestamp and update progress

Instrumentation
- Event: milestone_saved { has_target, status }

A11y
- Field errors announced; toggle state readable

---

## SCR-MILESTONE-QUICK-ADD — Milestone Quick Add

Purpose
- Fast inline creation of milestones within a plan section.

Entry
- Inline row in Goal Detail (per Plan) or Goal Form

Layout
- Single-line input (Title) + “Add” button
- On Add: row appears in list; optional toast “Added — set target date?”

States
- Validation: non-empty title
- Offline: supported

Instrumentation
- Event: milestone_quick_add

A11y
- Input labeled; button accessible; confirmation announced

---

Cross-Screen Error & Offline Handling
- Offline Banner: “You’re offline — AI and cloud shares are unavailable. All local features work.”
- Storage Error Modal: “Couldn’t save. Try again.” with Retry and copy diagnostics (PII-redacted)
- Long Operations: show progress with cancel and keep UI responsive

Security & Privacy Notes (applies to all screens)
- No AI/network activity without explicit user action
- No sensitive content in telemetry; IDs hashed/anonymized
- All exports/sharing initiated from Progress/Export flows retain redaction-by-default

Acceptance Mapping (Selected)
- AC-1: Template application via SCR-TEMPLATES → SCR-GOAL-REVIEW → SCR-GOAL-DETAIL
- AC-5: Milestone % and streaks visible on Dashboard goal cards and Goal Detail
- AC-6: AI suggestions path via SCR-GOAL-AI-SUGGEST with accept/edit before save
- AC-3 (context): Dashboard prompts to Morning Review; Checklist shows due/soon
- AC-2 (context): Routine completion flows accessible from Checklist/Notifications

Open Questions / TBD
- Should Goal Detail use tabs (Overview/Analytics/Activity) on mobile, or sections in a single scroll?
- Minimum enforced milestones at create time vs. soft guidance only?
- Default target-date offsets on template apply (unset vs. heuristic)?
- Per-goal color/tag accents on cards to improve scannability (ensure AA contrast)