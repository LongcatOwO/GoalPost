# GoalPost — Routines, Reviews, Notifications, and Progress Screen Specifications

File: docs/design/02-userflow/screens/routine-review-progress-screens.md
Scope: Routine management and check-ins, Morning/Night Reviews, Notification Center, and Progress/Analytics (including Achievements and Export)
Traceability:
- SRS Functional: FR-6..11 (Routines & Reminders), FR-12..14 (Reviews), FR-15..17 (Progress/Analytics/Export), FR-18..19 (Gamification), FR-20..22 (Journaling integration), FR-23..26 (AI assistance, optional in reviews), FR-30..32 (Success capture and privacy-redacted sharing), FR-33..36 (Settings/Data, indirectly)
- SRS Non-Functional: NFR-1..7, 9..12
- Core Use Cases: UC-3 (Routines & check-ins), UC-4 (Morning review), UC-5 (Night review), UC-6 (Progress & Achievements), UC-7/UC-8 (AI assist/diagnostics, optional), UC-9 (Success capture & export links), UC-10 (Backup/Restore, export overlap)

Conventions
- Mobile-first; responsive web adapts via multi-column layouts, persistent side-panels where feasible.
- Offline-first: core routine and review flows fully offline; AI and external sharing disabled gracefully.
- Privacy-first: all sharing/export opt-in with redaction ON by default.
- Destructive actions require confirmation. Long operations are cancelable and non-blocking.

Global Components
- App Bar, Banner (offline/info/warn/error), Card, List Row, Segmented Tabs, Modal/Bottom Sheet, Date/Time Picker, Number/Input Controls, Toast, Skeleton.
- Checklist Row Pattern: Leading icon (type), Title, Subtext (due time/streak), Trailing actions.

Performance Targets (NFR)
- UI interactions <150ms; lists/charts render <1s for typical sizes (≤5k records aggregated).
- AI ops: show loader, cancelable; timeout target 6–8s with manual fallback.
- Autosave local edits; atomic writes; no partial states on errors.

Accessibility (A11y)
- WCAG AA contrast; 44dp hit targets; keyboard navigation; aria roles/labels; live regions for async toasts and updates.
- Alt text required for images added by user; focus order logical and preserved on state changes.

---

## SCR-ROUTINE-LIST — Routines Overview

Purpose
- Central hub to view, create, edit, pause, and complete routine check-ins. Surface streaks and today’s due/overdue status.

Primary Data
- Routine list: title, cadence (daily/weekly/custom), schedule (time, DOW), streak, last_completed, due state (due now/today/overdue), pause state.

Entry
- From Dashboard “Routines” shortcut; from Goal Detail (pre-filtered to linked routines); from Morning/Night Reviews context.

Layout (Mobile)
- App Bar: “Routines”
  - Actions: Filter (linked goal, cadence), Sort (title, next due, streak desc)
- Summary Chips: Due Now | Due Today | Overdue | Active | Paused (counts)
- List (grouped): Due Now, Due Today, Upcoming, Paused
  - Row: [Icon] Title — cadence/time
    - Subtext: streak chip, next due at
    - Actions: Complete (if due), Snooze, Dismiss, Edit (kebab)
- FAB: “+ New Routine” → SCR-ROUTINE-QUICK-ADD

States
- Empty: “No routines yet — add your first one.” CTA → New Routine
- Offline: banner; fully functional

Key Actions
- Quick-add, Edit cadence/time, Pause/Resume, Complete (if due), Snooze/Dismiss
- Filter by linked goal or show all

Validation/Constraints
- Title required; at least one cadence selection; valid time(s)
- Pause stops future reminders; streak behavior per Settings

Instrumentation
- Event: routine_list_viewed { total, due_now, due_today, paused }
- Event: routine_quick_complete { routine_id_hash, streak_after }
- Event: routine_pause_toggled { to: paused|active }

A11y
- Each row has labeled controls; announce streak changes and completion via live region

---

## SCR-ROUTINE-QUICK-ADD — Quick Add Routine

Purpose
- Speedy routine creation with sensible defaults and optional link to Goal/Plan.

Entry
- From SCR-ROUTINE-LIST FAB or from Goal Detail (“Add Routine” pre-linked)

Layout
- Sheet/Modal: “New Routine”
  - Title (required)
  - Link to: Goal/Plan (optional; pre-filled if launched from Goal)
  - Primary CTA: “Save (Daily at [default morning time])”
  - Secondary: “Cadence & Time” → SCR-ROUTINE-EDITOR

Validation
- Title non-empty; if Save used, default cadence/time applied

Instrumentation
- Event: routine_quick_add_saved { linked_goal: boolean }

A11y
- Labels bound to inputs; focus trapped in modal; escape closes with confirmation if dirty

---

## SCR-ROUTINE-EDITOR — Routine Editor

Purpose
- Create/edit full routine details: cadence, schedule, pause/resume, and preferences.

Entry
- From quick-add “Cadence & Time”, from row “Edit”, or via Goal Detail routine list.

Layout
- Modal/Screen: “Routine”
  - Title (required)
  - Link to (Goal/Plan)
  - Cadence
    - Daily
    - Weekly (select days of week)
    - Custom: Every N days (spinner)
  - Time: HH:MM (local); add second time (post-MVP?). MVP: single time
  - Start Date (default today); Optional End Date
  - Notifications: In-app reminders [ON] (default)
  - Pause toggle (with optional pause_until date)
  - Save / Cancel; Delete (confirm)

Validation
- Title required; at least one cadence/time selection; dates sensible (start ≤ end)

Key Actions
- Save updates and reschedule reminders; pause/resume; delete

Instrumentation
- Event: routine_saved { cadence_type, time_changed: boolean }
- Event: routine_deleted

A11y
- Controls accessible; validation messages with aria-describedby

---

## SCR-CHECKIN-PANEL — Today’s Check-ins

Purpose
- Focused space to complete routine check-ins (and time-bound milestone actions), with streak updates and minimal friction.

Primary Data
- Due Now / Due Today / Overdue routine instances; milestone actions due soon (display only; milestone completion handled in respective flows).

Entry
- From Dashboard “Today’s Checklist” or Notification Center; post-Morning Review.

Layout
- App Bar: “Today”
  - Action: Edit selection (toggle to show all due)
- Sections: Due Now, Due Today, Overdue
  - Routine Row: Title, due time, streak chip
    - Actions: Complete, Snooze, Dismiss
  - Milestone Row (read-only or quick action if allowed by product policy): Title, due time, “Mark Done” → persists and opens small confirm

States
- Empty (no due): “You’re all set — check back later.”

Key Actions
- Complete routine (FR-7), Snooze/Dismiss (FR-10), Mark milestone done (FR-15; optional inline)

Validation
- Debounce double taps; offer Undo for 5s

Instrumentation
- Event: checkin_panel_viewed { counts_by_section }
- Event: routine_completed { routine_id_hash, streak_after }
- Event: routine_snoozed { duration }

A11y
- Rows fully keyboard-operable; completion announced via live region

---

## SCR-REVIEW-MORNING — Morning Review

Purpose
- Quickly select today’s top priorities, confirm times/reminders, and optionally get AI help planning the day.

Primary Data
- Today’s due/overdue routines and milestones; streak risk indicators; last 7-day completion trend metadata.

Entry
- Morning window prompt; Dashboard CTA.

Layout
- App Bar: “Morning Review”
- Banner (if applicable): offline, out-of-window note
- “Today at a Glance” cards: Due, Overdue, Streak at risk
- List: What’s on Today (selectable)
  - Per-item actions: Prioritize (toggle), Edit Time, Skip Today
- AI Suggestion (optional): “Suggest Plan” (if enabled)
- Footer: “Start My Day” (save snapshot)

Key Actions
- Select up to Top 3 priorities (soft limit)
- Set/adjust times with reminders
- Invoke AI suggestions with user acceptance (FR-26)

Validation
- Require at least one selection or confirm empty plan

Post-Save
- Route to SCR-CHECKIN-PANEL; persist DailyReviewSummary for the date (FR-14)

Instrumentation
- Event: morning_review_started { has_due, has_overdue }
- Event: morning_review_saved { priorities_count, used_ai }

A11y
- Selection state clear; time edits accessible; save confirmation announced

Acceptance Mapping
- AC-3: Displays all due/soon items; persists plan snapshot

---

## SCR-REVIEW-NIGHT — Night Review

Purpose
- Capture completions, log misses with reasons, reflect via a journal, and optionally run AI diagnostics for corrective actions.

Primary Data
- Today’s completions/misses; reason tags taxonomy; journal editor; (optional) AI diagnostic.

Entry
- Evening window prompt; Dashboard CTA; or after last completion.

Layout
- App Bar: “Night Review”
- Summary: Completed vs Planned; streaks; milestone % change
- Cards:
  - Quick Wins: Completed items (Add note/photo)
  - Missed Today: Select reasons (chips; multi-select) and optional notes
  - Reflection: Journal text area with suggested prompts
- AI Diagnostic (optional): “Why stuck?” → opens inline/side modal
- Footer: “Save Night Review”

Key Actions
- Mark late completions; add reasons for missed; write reflection; attach success evidence (notes/photos)

Validation
- If misses exist: require at least one reason or a journal entry before save

Post-Save
- Persist NightReviewSummary and Journal(s); update streaks/progress; optional AI artifact

Instrumentation
- Event: night_review_completed { completed_count, missed_count, journal_created, used_ai }

A11y
- Form fields labeled; chip group with clear selection; error prompts announced

Acceptance Mapping
- AC-4: Journal entry persisted for misses/reflection

---

## SCR-NOTIFICATION-CENTER — Reminders

Purpose
- Unified in-app reminder feed to see and act on overdue/today/upcoming items.

Primary Data
- Reminders: entity type (routine/milestone), due_at, status (Active/Snoozed/Dismissed).

Entry
- Dashboard CTA; navigation tab; from OS notifications (post-MVP).

Layout
- App Bar: “Reminders”
- Tabs: Overdue | Today | Upcoming
- List rows:
  - Title, entity icon, due time
  - Actions: Open, Snooze, Dismiss
- Multi-select toolbar: Snooze (preset or custom), Dismiss

Key Actions
- Open entity, Snooze/Dismiss single or bulk

Validation
- Snooze presets (+10m / +1h / tonight); custom via picker

Instrumentation
- Event: notifications_viewed { tab }
- Event: reminders_action { type, count }

A11y
- Tab changes announced; bulk select accessible

Acceptance Mapping
- FR-10..11: In-app reminders and notification center behavior

---

## SCR-PROGRESS-HUB — Progress Overview

Purpose
- Snapshot of goals’ progress: milestone completion %, longest streaks, overdue load; entry to deep analytics, achievements, and export.

Primary Data
- Aggregated progress metrics per goal and overall: milestone %, streaks, overdue counts.

Entry
- Dashboard “Progress”; via Goal Detail.

Layout
- App Bar: “Progress”
  - Actions: Export, Trends, Achievements
- At a Glance:
  - Active goals count
  - Avg milestone % (weighted/unweighted toggle)
  - Longest streak
  - Overdue items count
- Goal Cards:
  - Title, milestone %, streak chips, upcoming milestone(s)
  - CTAs: Open Goal (analytics tab), Export Goal Summary

Key Actions
- Toggle averages; open Trends/Achievements/Export; drill to Goal analytics

Instrumentation
- Event: progress_hub_viewed { goals_visible, has_overdue }

A11y
- Charts have text equivalents; toggles labeled

Acceptance Mapping
- AC-5: Milestone % and streaks visible

---

## SCR-TRENDS — Trends & Reports

Purpose
- Visualize weekly completion rates and streak trends; compare time windows and scopes.

Primary Data
- Rolling weekly completion %, streak highs/lows; filters by goal/routine/time window.

Entry
- From Progress Hub; from Goal analytics tab.

Layout
- App Bar: “Trends”
- Filters: Time Window (2w/4w/8w/custom), Scope (All/Goal), Routine filter
- Charts:
  - Weekly Completion Rate
  - Streak Trends (top routines)
- Interactions: tooltips, moving average toggle, period comparison

Performance
- Progressive loading; skeletons while computing

Instrumentation
- Event: trends_viewed { window, scope }

A11y
- Data tables accessible as alt representations; keyboard nav for chart focus

Acceptance Mapping
- FR-16: Trends reports satisfied

---

## SCR-ACHIEVEMENT-GALLERY — Achievements

Purpose
- Celebrate earned achievements; show locked criteria to motivate.

Primary Data
- Achievements: type, awarded_at, related entity, unlock criteria.

Entry
- From Progress Hub; from toasts.

Layout
- App Bar: “Achievements”
- Tabs: Earned | Locked
- Grid/List Cards:
  - Icon/Badge, name, awarded_at (if earned), brief criteria
- Details: tap → SCR-ACHIEVEMENT-DETAIL

Key Actions
- View criteria; navigate to related goal/routine; export from detail

Instrumentation
- Event: achievements_opened { earned_count }

A11y
- Card content announced with status (earned/locked); details page has headings

Acceptance Mapping
- FR-18..19: Badges and unlock visibility

---

## SCR-ACHIEVEMENT-DETAIL — Achievement Details

Purpose
- Provide full context for an achievement, with links and optional export shortcut.

Primary Data
- Badge metadata, criteria, awarded_at, related entity links.

Entry
- From Achievement Gallery.

Layout
- App Bar: badge name
- Content: description, criteria, awarded date, related entity link(s)
- CTA: “View Related” and “Export Progress Snapshot”

Instrumentation
- Event: achievement_detail_viewed { type }

A11y
- Clear headings and link targets; back navigation preserved

---

## SCR-PROGRESS-EXPORT — Export Snapshot

Purpose
- Export progress data to JSON/CSV or summary image/PDF with redaction ON by default.

Primary Data
- Export options (scope, format, redaction flags), preview, generation result.

Entry
- From Progress Hub or Achievement Detail; from Goal analytics tab.

Layout
- Step 1: Options
  - Scope: All Data | Selected Goal
  - Format: JSON (AC-7), CSV (optional), Summary (image/PDF/HTML, optional)
  - Redaction (default ON):
    - Include Photos [OFF]
    - Include Notes [OFF]
    - Include Journal Text [OFF]
    - Strip EXIF [ON]
  - Time Range (for summary): All / 4w / 8w / 12w
  - CTA: “Next: Preview”
- Step 2: Preview
  - Show items and redaction badges; warn when enabling inclusion
  - CTA: “Export”
- Step 3: Complete
  - File saved path; “Open File” button
  - Toast: “Export created. Photos hidden by default.”

Validation
- Confirm modal when turning ON sensitive content

Instrumentation
- Event: export_initiated { scope, format, redaction_on: boolean }
- Event: export_completed { duration_ms, size_kb }

A11y
- Form controls labeled; preview lists accessible

Acceptance Mapping
- FR-17, FR-31..32, AC-7, AC-8: Export with redaction defaults

---

## Cross-Screen Error, Offline, and Privacy Handling

Offline
- Banner: “You’re offline — AI and cloud shares are unavailable. Local features work.”
- Morning/Night Reviews, Routines, Notifications, Progress views function offline; AI buttons disabled with info tooltip.

Storage/Save Errors
- Modal: “Couldn’t save. Try again.” with Retry and copy diagnostics (PII-redacted).
- Long operations show progress with cancel; resume/rollback protections for partial failures.

AI Privacy & Safety
- No background AI calls; only upon explicit user action.
- Prompts avoid PII unless user includes it; show disclaimers and usage metadata (limits/tokens).
- All AI outputs require user acceptance before persisting and include confidence/assumptions (FR-26).

Telemetry (Privacy-Respecting)
- Metadata-only analytics; IDs hashed; no free text from journals/notes/photos or AI prompt/response bodies.

Accessibility
- Consistent focus management, announcements for key state changes (complete, errors, saves), labeled controls throughout.

---

## Acceptance Criteria Mapping (Summary)

- AC-2: Routine completion increments streak (SCR-CHECKIN-PANEL, SCR-ROUTINE-LIST).
- AC-3: Morning Review lists due/soon items and persists plan (SCR-REVIEW-MORNING).
- AC-4: Night Review saves a journal entry for misses/reflection (SCR-REVIEW-NIGHT).
- AC-5: Progress shows milestone % and streaks (SCR-PROGRESS-HUB; drill to Trends).
- AC-6: AI assist available where enabled with acceptance gating (Morning/Night).
- AC-7/AC-8: Exports default to redaction ON, JSON dataset available (SCR-PROGRESS-EXPORT).

Open Questions / TBD
- Enforce “Top 3” in Morning Review vs. soft guidance?
- Grace period rules around midnight for streak protection defaults.
- Allow multiple times per day for routines in MVP vs. post-MVP.
- Include mini-sparklines in summary exports at MVP?
- Routine pause behavior impact on streaks (default protect vs. reset).
