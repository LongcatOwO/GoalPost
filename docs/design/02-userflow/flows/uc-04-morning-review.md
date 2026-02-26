# UC-4 — Morning Review

ID: UC-4  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user)  
Level: User goal  
Frequency: Daily (within user-configured morning window)

Traceability (SRS):
- Core Use Case: UC-4 (Morning review: confirm priorities, planned tasks, and routines)
- Functional Requirements: FR-12..14 (Morning/Night Reviews), FR-6..8 (Routines), FR-9..11 (Scheduling/Reminders), FR-15..17 (Progress/Analytics), FR-23..26 (AI Assistance – optional)
- Acceptance Criteria: AC-3 (Morning review displays all due/soon tasks and routines for the day)

Related Screens:
- SCR-REVIEW-MORNING: Morning Review (primary flow surface)
- SCR-CHECKLIST-TODAY: Today’s Checklist (priorities and actions post-review)
- SCR-NOTIFICATION-CENTER: Overdue and upcoming items
- SCR-ROUTINE-LIST: Routines overview with streaks
- SCR-GOAL-DETAIL: Goal details (context on milestones)
- SCR-SCHEDULE-PICKER: Pick/adjust due times and reminders
- SCR-AI-ASSIST-PLAN: AI “Plan my day” suggestions (optional)
- SCR-OFFLINE: Offline banner/modal
- SCR-ERROR: Inline validation/recovery

Non-Functional Reminders:
- NFR-1 Usability: Complete morning review ≤ 2 minutes with ≤ 5 taps
- NFR-2 Performance: <150ms interactions; lists render <1s
- NFR-4 Availability: Works offline; AI calls degrade gracefully
- NFR-6 Privacy: All data private by default
- NFR-7 Accessibility: Keyboard/touch-friendly, WCAG AA contrast
- NFR-9 Observability: Log metadata-only events; redact PII

---

## Goal/Benefit
Help the user start the day with clarity and focus by surfacing what matters now:
- See all due/soon routines and milestones (plus overdue carryovers).
- Pick the top priorities for today and confirm timing.
- Optionally use AI to propose a lightweight daily plan.
Outcome: A “Today’s Plan” snapshot is saved; reminders are set; the user lands on a simple checklist to execute.

---

## Preconditions
- User has at least one Goal/Plan/Routine configured (or templates were applied previously).
- Local data available; no dependency on network for base flow.
- Morning review window is configured in Settings (FR-33) or uses default (e.g., 05:00–11:00).
- AI assist only available if enabled and within usage limits (FR-23..26).

## Triggers
- Automatic prompt within morning window when the app opens: “Start Morning Review?”
- User taps “Start Morning Review” from SCR-DASH or a persistent prompt card.
- A daily reminder (in-app) opens the review.

## Postconditions (Success)
- A “Today’s Plan” is persisted for the current date (FR-14).
- Selected priorities (e.g., top 3) and adjusted due times/reminders are saved (FR-9..11).
- Overdue items are carried and reflected; streak at-risk routines are highlighted.
- User lands on SCR-CHECKLIST-TODAY with the confirmed list.

## Postconditions (Failure/Abort)
- If user cancels, no changes are persisted (draft optional).
- If partial edits occurred before cancel, they are rolled back (or saved to draft if enabled).

---

## Primary Flow — Happy Path

1) Entry — SCR-REVIEW-MORNING
   - System shows “Today at a Glance”:
     - Due/Overdue counts (Routines, Milestones)
     - Streaks and “at risk today” routines
     - Motivational nudge (lightweight; privacy safe)

2) Consolidated List — “What’s on Today”
   - Sections:
     - Due Now / Overdue: routines, milestones with due today or earlier
     - Due Soon (today): remaining items with scheduled times
   - Inline actions per item: “Prioritize”, “Edit Time”, “Skip Today”
   - Bulk action: “Select Top 3 Priorities” (recommended cap, editable)

3) Prioritize
   - User taps to mark up to 3 “Top Priorities” (soft limit; allow >3 with nudge).
   - Optional reorder (drag) to set execution sequence.
   - Visual tokens: priority rank badges (1,2,3).

4) Adjust Timing (Optional)
   - For any selected item, user taps “Edit Time” → SCR-SCHEDULE-PICKER.
   - Set/adjust due time (today) and enable in-app reminder (default on).
   - Save returns to SCR-REVIEW-MORNING with updated schedule.

5) Optional AI Suggestion — SCR-AI-ASSIST-PLAN
   - CTA: “Suggest Plan for Today”
   - If enabled and online, the system sends:
     - today’s due/overdue items, streak risks, last 7-day completion trend (metadata only), user-set working hours block (optional setting)
   - AI returns:
     - Suggested prioritized list (≤5) with rationale and time blocks
     - Confidence and assumptions (FR-26)
   - User can Accept All, Accept Some, Reorder, or Edit.
   - If AI unavailable, show non-blocking banner and remain in manual flow.

6) Confirm & Save
   - User taps “Start My Day” / “Save Plan”.
   - System validates:
     - At least one item selected OR confirm “Start with an empty plan?”
     - Times (if set) are on today’s date
   - Persist the “Today’s Plan” snapshot and reminders (FR-14, FR-9..11).

7) Route to Execution — SCR-CHECKLIST-TODAY
   - Show prioritized checklist with quick actions:
     - Complete (for routines), Mark done (for milestones), Snooze, Dismiss
   - Toast: “Today’s plan ready.”

Result: AC-3 satisfied — all due/soon routines and tasks are visible; user has a clear prioritized plan for the day.

---

## Alternate Flows

A1) Nothing Due Today
- System shows empty-state coaching:
  - “Keep momentum: pick a maintenance routine” or
  - “Advance a milestone proactively”
- Allow adding 1–3 optional focus items from all lists or templates.

A2) All Overdue
- Highlight overdue first with an “Overdue Recovery” toggle:
  - Suggest “pick one quick win” for momentum.
  - Option to reschedule the rest in bulk (tonight/tomorrow).

A3) Quick Review (30s Mode)
- User taps “Quick Review”:
  - Auto-prioritize top 3 by due time and streak risk.
  - Single confirm to save and go to SCR-CHECKLIST-TODAY.

A4) Skip Morning Review
- User taps “Skip”:
  - System dismisses prompt for today (still accessible from menu).
  - No plan snapshot saved.

A5) Pre-Selected Favorites
- If user enabled “Always include these routines on weekdays,” pre-check them on entry.
- User can unselect before confirmation.

---

## Exception Flows

E1) Validation Errors
- Invalid time (e.g., past time outside grace) → inline fix or auto-adjust to next available slot.
- Conflicting times → non-blocking info: “Two items at 7:30. Keep both?”

E2) Offline Mode
- Entire core flow works.
- AI button disabled with banner: “AI suggestions unavailable offline.”

E3) Storage Error
- Modal: “Couldn’t save today’s plan. Try again.”
- Options: Retry, Copy details (redacted logs), Cancel (retain in-memory state if possible).

E4) Outside Morning Window
- If user starts outside configured window:
  - Light banner: “You’re starting late; that’s okay!” Continue without restriction.

E5) Excess Priorities
- If >3 selected, show nudge:
  - “Fewer priorities increase focus. Proceed anyway?” Continue allowed.

---

## Data Created/Updated

- DailyReviewSummary (for today)
  - id, date, priorities: [entity_ref, order, planned_time?], created_at, updated_at
  - metadata: { started_at, duration_sec, used_ai?: boolean, item_counts: { due, overdue, soon } }
- Reminder updates (for prioritized items with scheduled times)
  - id, entity_ref, due_at (today), status: Active|Snoozed|Dismissed
- AI Artifact (if used)
  - prompt_metadata (counts/tags only), suggestions[], rationale, confidence, accepted_at
- Derived Progress cache refresh (optional)
  - today_focus_count, streak_risk_items

Note: Journal entries are not created in morning review (night review handles reflection); user can still add notes ad-hoc elsewhere.

---

## Business Rules & Constraints

- BR-1: Only one “official” morning plan snapshot per day; subsequent runs update/overwrite today’s summary.
- BR-2: “Top 3” is a recommendation, not a hard cap.
- BR-3: Items without explicit due times remain on the checklist without time-bound reminders.
- BR-4: AI outputs require user acceptance before persisting (FR-26).
- BR-5: Routines and milestones maintain their own status; morning plan is a non-destructive overlay for today’s execution.
- BR-6: Privacy default is private; no external sharing implied by this flow.

---

## Analytics & Telemetry (Privacy-Respecting)

- Event: morning_review_started { local_time, has_due, has_overdue, has_soon }
- Event: morning_review_ai_invoked { success: boolean, duration_ms }
- Event: morning_review_priorities_selected { count, count_with_times }
- Event: morning_review_saved { duration_ms, priorities_count, used_ai }
- Event: morning_review_skipped { reason?: “no_items|time|user_choice” }

Note: Redact or hash entity IDs; no PII or free-text content in logs.

---

## Acceptance Criteria (Derived)

- AC-4.1: Opening morning review displays all due/soon routines and milestones for the day, plus overdue items (AC-3).
- AC-4.2: User can select priorities and optionally set times and reminders; save persists a “Today’s Plan” snapshot (FR-14, FR-9..11).
- AC-4.3: If AI is enabled, user can receive a suggested prioritized list with rationale and accept/edit before saving (FR-23..26).
- AC-4.4: Completing morning review routes to a checklist for execution (SCR-CHECKLIST-TODAY).
- AC-4.5: Flow completes in ≤ 2 minutes with ≤ 5 taps in a typical case (NFR-1).

---

## Open Questions / TBD

- Should the default cap be Top 3 or Top 5 for certain personas?
- Grace period rules for “past time” scheduling during morning review (e.g., auto-shift 10–15 minutes ahead)?
- Allow “focus blocks” (Pomodoro-style) at MVP, or post-MVP?
- How should “streak at risk” be computed for weekly/custom cadences in the morning summary?
- Do we snapshot “time-on-task” intent for analytics, or postpone to execution tracking?

---

## UI State Map

- ST-UC4-01: SCR-REVIEW-MORNING (overview + selection)
- ST-UC4-02: SCR-SCHEDULE-PICKER (optional time selection)
- ST-UC4-03: SCR-AI-ASSIST-PLAN (optional suggestions)
- ST-UC4-04: SCR-CHECKLIST-TODAY (post-confirm execution)
- ST-UC4-ERR: SCR-ERROR (validation), SCR-OFFLINE (banner)

---

## Wireflow (Textual)

- App open (morning window) → Prompt → SCR-REVIEW-MORNING
  - View “Today at a Glance”
  - Select priorities (tap to choose, optional reorder)
  - Optional “Edit Time” → SCR-SCHEDULE-PICKER → Save → back
  - Optional “Suggest Plan” → SCR-AI-ASSIST-PLAN → Accept/Edit → back
  - “Start My Day” → Save snapshot → SCR-CHECKLIST-TODAY
- Alternate: “Skip” → Dismiss prompt → Return to previous screen (no snapshot)