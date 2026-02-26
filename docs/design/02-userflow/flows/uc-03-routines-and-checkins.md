# UC-3 — Define Routines and Complete Daily Check-ins

ID: UC-3  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user)  
Level: User goal  
Frequency: High (daily use)

Traceability (SRS):
- Core Use Case: UC-3
- Functional Requirements: FR-6..8 (Routine/Habit Tracking), FR-9..11 (Scheduling/Reminders), FR-12..14 (Reviews, context), FR-15..17 (Progress/Analytics), FR-18..19 (Gamification/Achievements, indirect)
- Acceptance: FR-6..8 Acceptance (“Mark a routine complete; streak increments accordingly”)

Related Screens:
- SCR-ROUTINE-LIST: All routines with today status and streaks
- SCR-ROUTINE-QUICK-ADD: Inline quick-add routine
- SCR-ROUTINE-EDITOR: Create/Edit routine (cadence, schedule)
- SCR-CHECKIN-PANEL: Today’s check-ins (Mark Complete / Snooze / Dismiss)
- SCR-SCHEDULE-PICKER: Date/time and cadence options
- SCR-NOTIFICATION-CENTER: Upcoming/overdue reminders
- SCR-REVIEW-MORNING: Morning review (lists today’s routines/tasks)
- SCR-REVIEW-NIGHT: Night review (capture completions/blockers)
- SCR-ERROR: Inline validation/errors
- SCR-OFFLINE: Offline banner/modal

Non-Functional Reminders:
- NFR-1 Usability: Complete routine check-ins ≤ 2 minutes with ≤ 5 taps
- NFR-2 Performance: <150ms interactions; lists and streak updates <1s
- NFR-4 Availability: Fully functional offline (in-app reminders); degrade gracefully when push/OS notifications aren’t available
- NFR-6 Privacy: All data private by default
- NFR-7 Accessibility: Clear labels, keyboard navigation, WCAG AA contrasts
- NFR-9 Observability: Log metadata-only events (no PII)

---

## Goal/Benefit
Let the user define recurring routines with cadences (daily/weekly/custom) and make frictionless daily check-ins that maintain streaks, drive consistency, and feed progress views and achievements.

Outcome: User has one or more routines with schedule/cadence; check-ins are easy to complete; streaks update; reminders and reviews surface what matters today.

---

## Preconditions
- User has at least one Goal or Plan (optional for general routines; recommended to link routines to goals/plans for traceability).
- Local storage available. Online not required.
- OS/push notifications are optional post-MVP; in-app reminders are available for MVP (FR-10..11).

## Triggers
- From SCR-GOAL-DETAIL: “Add Routine” under a Goal/Plan
- From SCR-ROUTINE-LIST: “+ New Routine”
- From SCR-REVIEW-MORNING: Prompt to add a routine if none exist
- From due reminder in SCR-NOTIFICATION-CENTER or SCR-CHECKIN-PANEL: complete/snooze/dismiss

## Postconditions (Success)
- Routines exist with cadence and schedule (FR-6).
- Check-ins recorded; streaks update (FR-7).
- Today/Upcoming reminders reflect state (FR-9..11).
- Progress views show streak metrics (FR-15).
- Achievements may be awarded for streak thresholds (FR-18..19).

## Postconditions (Failure/Abort)
- If user cancels before saving a new routine, no persistence occurs.
- For check-in actions, if canceled, no change to streak or completion state.
- Optional: Auto-save drafts for routine creation if the user navigates away unexpectedly.

---

## Primary Flow A — Create a Routine

1) Entry — SCR-ROUTINE-LIST or SCR-GOAL-DETAIL
   - User taps “+ New Routine” or “Add Routine”.

2) Quick Add — SCR-ROUTINE-QUICK-ADD
   - Inputs:
     - Title (required)
     - Link to: Goal or Plan (optional but recommended)
   - CTA: “Next: Cadence & Time” or “Save (Default: Daily 9am)”
   - Default: Daily at user’s preferred morning time from Settings (FR-33).

3) Configure Cadence & Schedule — SCR-ROUTINE-EDITOR
   - Cadence options:
     - Daily, Weekly (pick days), Custom (e.g., every N days)
   - Time options:
     - One time per day; optional second time; or time window (MVP: single time)
   - Additional options:
     - Start date (default today), Optional end date
     - Notifications: in-app reminder enabled (default on)
   - Validation:
     - Title required
     - At least one cadence selection
   - CTA: “Create Routine”

4) Persist & Route — SCR-ROUTINE-LIST (or back to SCR-GOAL-DETAIL)
   - System saves Routine with streak=0, last_completed=null.
   - In-app reminders scheduled per cadence/time.
   - Toast: “Routine created”.

Result: Routine appears in Today’s check-in list when due.

---

## Primary Flow B — Complete a Daily Check-in

1) Entry — SCR-CHECKIN-PANEL or SCR-NOTIFICATION-CENTER
   - Shows “Due Now”, “Due Soon”, or “Overdue” routines with action buttons.

2) Complete — SCR-CHECKIN-PANEL
   - User taps “Mark Complete”.
   - System updates:
     - last_completed = today (with timestamp)
     - streak = streak + 1 if yesterday (or last cadence occurrence) was completed; otherwise streak resets to 1 (see rules below)
   - Toast: “Completed. Streak: X days.”

3) Optional Notes/Reflection
   - Inline quick note (optional) to link a Journal Entry (FR-20..22, helpful when missed previously).
   - CTA: “Add Note” (non-blocking).

4) Refresh Lists
   - Remove item from Today’s pending list.
   - Update streak displays across SCR-ROUTINE-LIST and progress views.

Result: FR-7 acceptance met (streak increments after completion).

---

## Primary Flow C — Snooze or Dismiss a Reminder

1) Entry — SCR-NOTIFICATION-CENTER or inline on SCR-CHECKIN-PANEL item
   - User taps “Snooze” or “Dismiss”.

2) Snooze
   - Options: +10 min, +1 hour, +Today evening (configurable shortcuts).
   - Reminder rescheduled; item stays in Today panel (snoozed state).

3) Dismiss
   - Removes current reminder occurrence from the reminder list.
   - Does not mark complete; no change to streak.

Result: User can manage timing without losing track of the routine.

---

## Primary Flow D — Edit or Pause a Routine

1) Entry — SCR-ROUTINE-LIST
   - User selects a routine, taps “Edit”.

2) Edit — SCR-ROUTINE-EDITOR
   - Change cadence, schedule, or linked goal/plan.
   - Optional: Pause routine (stop reminders) with a resume date.
   - CTA: “Save Changes”.

3) Persist & Recompute
   - System updates reminder schedule.
   - Streak unaffected by edits; pause prevents streak decay if configured (see rules).

Result: Routine remains accurate to user’s life changes.

---

## Alternate Flows

A1) Create from Template
- When creating a Goal from a Template (UC-1), routines may pre-populate.
- User can review/adjust cadence/time in SCR-ROUTINE-EDITOR before saving.
- Original template stays immutable (FR-29).

A2) Bulk Create
- User adds multiple routines via repeated quick-add.
- System supports rapid entry and later fine-tuning.

A3) Custom Cadence Examples
- Weekly on Mon/Wed/Fri at 7am.
- Every N days (e.g., every 2 days).
- System calculates “next due” and streak continuity accordingly.

A4) Time Zone Change
- On device time zone shift, system adjusts “today” calculations based on local time with guardrails to prevent double-counting or missed day (see constraints).

A5) Morning/Night Review Integration
- SCR-REVIEW-MORNING lists today’s routines; completing there follows Primary Flow B.
- SCR-REVIEW-NIGHT prompts to log missed routines and reasons (links to Journal).

---

## Exception Flows

E1) Validation Errors (Create/Edit)
- Missing title → inline error, disable Save.
- No cadence selected → inline error.
- Invalid time → reset to last valid with message.

E2) Scheduling Conflicts
- Overlapping routine times → non-blocking info; both remain but reminders may combine in UI.

E3) Offline Mode
- Fully supported; reminders are in-app.
- Any optional cloud sync or push is deferred; no data loss (NFR-4).

E4) Storage Error
- Modal: “Couldn’t save. Try again.”
- Options: Retry, Copy details (redacted logs), Cancel (retain draft if possible).

E5) Streak Calculation Ambiguity
- If system clock changed or time zone jumped, show light banner:
  “We adjusted your check-in windows due to time changes.” Provide link to details and allow manual correction (optional enhancement).

---

## Data Created/Updated

- Routine {
  id, goal_id|plan_id (optional), title, cadence (daily|weekly|custom),
  schedule (time(s), days-of-week or N), paused?: boolean, pause_until?: date,
  streak: number, last_completed: date|null, created_at, updated_at
}
- Reminder (derived from schedule) {
  id, entity_ref: routine_id, due_at, status: Active|Snoozed|Dismissed
}
- Journal Entry (optional, if user adds note on miss/completion) {
  id, date, text, reason_tags?, linked_entities: [routine_id]
}
- Achievement (indirect, if thresholds hit) {
  id, type: “streak_3|streak_7|...”, awarded_at, entity_ref: routine_id
}
- Progress Cache (derived) {
  routine_id, streak, completion_trend (optional aggregate), last_completed
}

Note: MVP can avoid separate completion-event rows; update streak/last_completed directly, optionally maintaining a lightweight internal history for trend reports (FR-16).

---

## Business Rules & Constraints

- BR-1: Title is required; cadence must be set.
- BR-2: Streak increments when a check-in is completed within the current cadence window, and the previous cadence window was also completed (consecutive).
- BR-3: Missing a cadence window resets streak to 0 (or 1 at next completion).
- BR-4: Paused routines neither increment nor reset streak during pause if “protect streak during pause” is enabled; otherwise, streak behaves normally (choose default in Settings).
- BR-5: Time zone changes should not unfairly penalize streaks; apply sensible grace windows on the day of change.
- BR-6: Routines linked to a Goal/Plan contribute to that Goal’s progress context (display only; routines don’t change milestone completion unless explicitly linked).

---

## Analytics & Telemetry (Privacy-Respecting)

- Event: routine_create_initiated { source: “goal_detail|routine_list|template” }
- Event: routine_created { cadence_type, has_linked_goal: boolean }
- Event: routine_checkin_completed { routine_id_hash, streak_after, completion_time_local }
- Event: routine_snoozed { routine_id_hash, snooze_duration }
- Event: routine_dismissed { routine_id_hash }
- Event: routine_edited { cadence_changed: boolean, time_changed: boolean }
- Event: routine_paused { routine_id_hash, pause_until }
- Event: streak_reset { routine_id_hash, reason: “missed_window|manual_reset|clock_change_adjustment” }

Note: Hash or otherwise de-identify routine IDs; no PII content (NFR-9).

---

## Acceptance Criteria (Derived)

- AC-3.1: User can create a routine with a daily or weekly cadence and a time.
- AC-3.2: A due routine appears in Today’s check-in list and/or Notification Center (FR-10..11).
- AC-3.3: Marking a routine complete updates last_completed and increments streak when consecutive windows are satisfied (FR-7).
- AC-3.4: User can snooze or dismiss a routine reminder (FR-10).
- AC-3.5: Streak count is visible on the routine and updates immediately after completion (FR-15).
- AC-3.6: Editing a routine updates its schedule; streak remains intact unless explicitly reset.

---

## Open Questions / TBD

- Should MVP include “protect streak during pause” option, and what’s the default?
- How strict are cadence windows for late-night completions (e.g., grace period past midnight)?
- Do we show visual “heatmap” of completions for each routine (post-MVP analytics)?
- Should we allow multiple times per day for a single routine in MVP, or keep it single time for simplicity?
- How are weekly custom cadences (e.g., M/W/F) represented in missed-window calculations—per selected day only?

---

## UI State Map

- ST-UC3-01: SCR-ROUTINE-LIST (overview, add button, streaks)
- ST-UC3-02: SCR-ROUTINE-QUICK-ADD (title + link to goal/plan)
- ST-UC3-03: SCR-ROUTINE-EDITOR (cadence/time, pause)
- ST-UC3-04: SCR-CHECKIN-PANEL (today due/overdue with actions)
- ST-UC3-05: SCR-NOTIFICATION-CENTER (reminders list)
- ST-UC3-ERR: SCR-ERROR (validation), SCR-OFFLINE (banner)

---

## Wireflow (Textual)

- From SCR-ROUTINE-LIST
  - “+ New Routine” → SCR-ROUTINE-QUICK-ADD → “Next” → SCR-ROUTINE-EDITOR → “Create” → SCR-ROUTINE-LIST
  - Tap a due item in Today panel → SCR-CHECKIN-PANEL → “Mark Complete” → Update streak → Return
  - Tap “Snooze” → pick duration → Return (item shows snoozed)
  - Tap “Dismiss” → Return (item removed from reminders)

- From SCR-GOAL-DETAIL
  - “Add Routine” → SCR-ROUTINE-QUICK-ADD (pre-linked to Goal/Plan) → SCR-ROUTINE-EDITOR → “Create” → Back to SCR-GOAL-DETAIL (routine visible in related section)

- From SCR-REVIEW-MORNING
  - See today’s routines → complete/snooze/dismiss inline (same logic as SCR-CHECKIN-PANEL)

- From SCR-REVIEW-NIGHT
  - See missed routines → optionally log reasons (Journal link) → Save → Close review