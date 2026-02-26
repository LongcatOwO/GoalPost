# UC-5 — Night Review (Completions, Blockers, and Journaling)

ID: UC-5  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user)  
Level: User goal  
Frequency: Daily (within user-configured evening window)

Traceability (SRS):
- Core Use Case: UC-5 (Night review: record completions, blockers, and journal reflections)
- Functional Requirements: FR-12..14 (Morning/Night Reviews), FR-6..8 (Routines), FR-9..11 (Scheduling/Reminders), FR-15..17 (Progress/Analytics), FR-18..19 (Gamification/Achievements), FR-20..22 (Journaling & Failure Reflection), FR-23..26 (AI Assistance – diagnostics), FR-30..32 (Success Capture/Sharing – optional)
- Acceptance Criteria: AC-4 (Night review persists a journal entry with missed items and reasons)

Related Screens:
- SCR-REVIEW-NIGHT: Night Review (primary surface)
- SCR-MISSED-ITEMS: Missed items selector with reason taxonomy
- SCR-JOURNAL-ENTRY: Reflection editor (text, tags, links to entities)
- SCR-AI-DIAGNOSE: AI diagnostic summary for blockers (optional)
- SCR-SUCCESS-CAPTURE: Add notes/photos to completed items (optional)
- SCR-PROGRESS-VIEW: Summary of today’s progress (streaks, completion %)
- SCR-ACHIEVEMENT-TOAST: Earned badges/achievements UI
- SCR-NOTIFICATION-CENTER: Overdue/upcoming reminders overview (context)
- SCR-OFFLINE: Offline banner/modal
- SCR-ERROR: Inline validation/recovery

Non-Functional Reminders:
- NFR-1 Usability: Complete in ≤ 2 minutes with ≤ 5 taps (typical case)
- NFR-2 Performance: <150ms interactions; summaries render <1s
- NFR-4 Availability: Works offline; AI degrades gracefully
- NFR-6 Privacy: Private by default; redaction on any optional export
- NFR-7 Accessibility: Keyboard/touch-friendly; WCAG AA contrast
- NFR-9 Observability: Metadata-only logs; redact PII

---

## Goal/Benefit
Help the user close the day with clarity:
- Capture what got done and celebrate small wins.
- Identify what didn’t get done and why (blockers/contexts).
- Reflect and learn via journaling to improve tomorrow.
- Optionally request AI diagnostics to spot patterns and propose corrective actions.

Outcome: A Night Review snapshot is saved with completions, missed items and reasons, and at least one Journal Entry linked to the day; progress and achievements update accordingly.

---

## Preconditions
- User has routines/milestones/tasks planned or in progress.
- Local data available; connectivity not required (AI optional).
- Night review window configured in Settings (FR-33) or default (e.g., 19:00–23:59).

## Triggers
- Automatic prompt within evening window: “Start Night Review?”
- User opens SCR-REVIEW-NIGHT from SCR-DASH or reminder card.
- Completing last scheduled item prompts a “Wrap up your day?” CTA.

## Postconditions (Success)
- A Night Review record for the current date is persisted (FR-14).
- Completions and misses are recorded; streaks and completion metrics update (FR-7, FR-15).
- At least one Journal Entry exists if any items were missed or the user added reflections (FR-20..22; AC-4).
- Optional AI diagnostic artifact saved if invoked and accepted (FR-24..26).
- Optional success captures (notes/photos) stored and linked (FR-30).

## Postconditions (Failure/Abort)
- If canceled before saving, no changes persist (optional draft).
- If partial edits occurred, roll back or keep a draft (if enabled) for later completion.

---

## Primary Flow — Happy Path

1) Entry — SCR-REVIEW-NIGHT
   - Header: “How did today go?”
   - Cards:
     - Today’s Summary: count completed vs. planned; routine streaks; milestone progress.
     - Quick Wins: items completed today (tap to add note/photo).
     - Missed Today: items due/selected this morning not completed (tap to add reasons).

2) Mark Completions (Quick Wins)
   - List shows items already completed; user can:
     - Add note or attach photo (optional success capture).
     - Mark any late completions (if not yet marked done) with a single tap.
   - Saving updates progress immediately; possible achievement toast if thresholds met (FR-18/19).

3) Capture Missed Items & Reasons — SCR-MISSED-ITEMS
   - For each missed item, user selects one or more reason tags from a taxonomy (FR-21), e.g.:
     - Time constraints, Low energy, Overplanned, Distraction, External event, Unclear next step, Other (free text)
   - Optional “Retry plan” toggle: auto-carry to tomorrow’s morning review.
   - CTA: “Save Reasons”.

4) Journal Reflection — SCR-JOURNAL-ENTRY
   - Editor with prompt suggestions:
     - “What helped today?”
     - “What blocked you?”
     - “One improvement for tomorrow?”
   - Auto-links to missed and/or significant completed entities (FR-22).
   - CTA: “Save Journal”.
   - Upon save, at least one Journal Entry is created for the day (AC-4).

5) Optional AI Diagnostics — SCR-AI-DIAGNOSE
   - CTA: “Why am I getting stuck?”
   - If enabled/online:
     - System summarizes non-PII metadata: streak trends, missed reason tags, overdue counts, last 7-day completion rate (FR-25).
     - AI returns a short diagnostic:
       - Patterns observed (e.g., frequent “unclear next step” on weekdays)
       - Hypotheses and confidence
       - 2–3 corrective actions (e.g., reduce daily load, define smaller milestone steps, schedule earlier)
     - User can Accept (convert into tomorrow suggestions or journal addendum) or Edit.
     - All AI outputs require user acceptance before saving (FR-26).

6) Confirm & Save — SCR-REVIEW-NIGHT
   - System validates:
     - If any missed items exist, at least one Journal Entry or reason-tag capture recorded.
   - Persist Night Review summary, journal(s), updates to item statuses, achievements, and optional AI artifact.
   - Toast: “Night review saved. Well done.”

7) Route
   - Offer direct route to:
     - SCR-PROGRESS-VIEW (see today’s charts)
     - Or “Set up tomorrow now?” → deep-link to Morning Review pre-draft (optional)

Result: AC-4 satisfied — a journal entry linked to the day is created, missed items have reasons, and progress is updated.

---

## Alternate Flows

A1) No Items Today
- Empty-state coaching:
  - “Capture a quick reflection anyway?”
  - “Set one small intention for tomorrow.”
- User can create a standalone Journal Entry and save review.

A2) Bulk Actions for Missed Items
- “Apply same reason to all” (e.g., “Time constraints today”).
- “Carry all to tomorrow morning plan.”

A3) Minimal Check-in (30s Mode)
- Single screen:
  - “Completed: X / Planned: Y”
  - Toggle 1–2 misses with reason tag
  - Quick reflection text area
  - Save → Done

A4) Late-Night Completions
- If user logs late completions past midnight within grace window:
  - Attribute to the intended day and adjust streaks accordingly.

A5) Success Capture Later
- User skips adding photos/notes now.
- Items remain editable; a “Add evidence” reminder appears next time in-progress view opens.

---

## Exception Flows

E1) Validation Errors
- If missed items exist but neither reasons nor a journal entry recorded:
  - Inline prompt: “Add at least one reason or a short reflection.”
- If journal text too long (if constrained):
  - Show counter and block save until within limit or allow auto-truncation (opt-in).

E2) Offline Mode
- Entire flow works.
- AI button disabled with banner: “AI diagnostics unavailable offline.”
- Save locally; sync/export later if applicable.

E3) Storage Error
- Modal: “Couldn’t save your night review. Try again.”
- Options: Retry, Copy details (redacted logs), Cancel (retain draft if possible).

E4) Clock/Time Zone Change
- Light banner: “We adjusted day boundaries due to time change.”
- Link to details and allow manual correction for item attribution (optional).

---

## Data Created/Updated

- NightReviewSummary (for today)
  - id, date, planned_count, completed_count, missed_count
  - missed_entities: [{ entity_ref, reason_tags[], notes? }]
  - achievements_awarded?: [achievement_id]
  - created_at, updated_at

- Journal Entry (at least one when there are misses or user adds reflection)
  - id, date, text, reason_tags?, linked_entities: [goal_id|plan_id|milestone_id|routine_id]
  - created_at, updated_at

- Entity updates
  - Routine: last_completed, streak updates (if completion logged)
  - Milestone: status (Completed/Pending), completion timestamp (if completed)
  - Optional attachments (notes/photos) for success capture

- AI Artifact (optional)
  - prompt_metadata (counts/tags only), diagnostic_text, rationale, confidence, accepted_actions[], accepted_at

- Progress Cache (derived)
  - daily_completion_rate, rolling_7d, streak_changes

Note: All logs/AI metadata redact PII per NFR-9.

---

## Business Rules & Constraints

- BR-1: If any item was missed, user must either tag at least one reason or create a journal entry before final save (AC-4).
- BR-2: Streak logic matches routine cadence; late completions within grace window count toward intended day.
- BR-3: Achievements can trigger on consistency thresholds (e.g., 3/5/7-day streak, first milestone complete).
- BR-4: AI outputs are suggestions only; require explicit acceptance and are recorded with confidence/assumptions (FR-26).
- BR-5: Privacy default is private; success captures are not shared unless user later exports/shares explicitly (FR-31..32).
- BR-6: Night Review is one per day; re-opening updates/overwrites that day’s summary (with history retained internally if implemented).

---

## Analytics & Telemetry (Privacy-Respecting)

- Event: night_review_started { local_time, had_planned: boolean }
- Event: night_review_completed { duration_ms, completed_count, missed_count, journal_created: boolean, used_ai: boolean }
- Event: missed_reasons_selected { counts_by_tag }
- Event: success_capture_added { count, has_photos: boolean }
- Event: ai_diagnostic_invoked { success: boolean, duration_ms }
- Event: achievements_awarded { types[] }

Note: Redact IDs or hash; no free-text content in analytics (NFR-9).

---

## Acceptance Criteria (Derived)

- AC-5.1: Starting night review shows today’s completed and missed items with a quick summary (FR-12..14).
- AC-5.2: User can add reasons for missed items from a taxonomy and/or write a reflection (FR-21).
- AC-5.3: Saving night review persists at least one Journal Entry when there are misses or when the user enters a reflection, linked to the day and relevant entities (FR-20..22; AC-4).
- AC-5.4: Completing items during review updates routine streaks and milestone completion status immediately (FR-7, FR-15).
- AC-5.5: Optional AI diagnostic suggests patterns and corrective actions; user can accept/edit before saving (FR-24..26).

---

## Open Questions / TBD

- Should reasons taxonomy be editable by users in MVP, or fixed with “Other” free text?
- What is the default grace window for late-night completions crossing midnight?
- Do we prompt automatic carryover of all missed items to tomorrow by default, or ask per item?
- Should “energy/mood” quick sliders be included to enrich diagnostics (post-MVP)?
- Do we allow multiple journal entries in a single night review, or consolidate into one with sections?

---

## UI State Map

- ST-UC5-01: SCR-REVIEW-NIGHT (summary + actions)
- ST-UC5-02: SCR-MISSED-ITEMS (reasons tagging)
- ST-UC5-03: SCR-JOURNAL-ENTRY (reflection)
- ST-UC5-04: SCR-AI-DIAGNOSE (optional diagnostics)
- ST-UC5-05: SCR-SUCCESS-CAPTURE (optional notes/photos)
- ST-UC5-06: SCR-PROGRESS-VIEW (post-save optional)
- ST-UC5-ERR: SCR-ERROR (validation), SCR-OFFLINE (banner)

---

## Wireflow (Textual)

- App open (evening window) → Prompt → SCR-REVIEW-NIGHT
  - Review “Today’s Summary”
  - Tap “Quick Wins” → mark/annotate completions (optional success capture) → back
  - Tap “Missed Today” → SCR-MISSED-ITEMS → select reasons (bulk or per-item) → Save → back
  - Tap “Reflect” → SCR-JOURNAL-ENTRY → write reflection (links auto) → Save → back
  - Optional: “Why stuck?” → SCR-AI-DIAGNOSE → Accept/Edit → back
  - “Save Night Review” → Persist summary, journal(s), updates, achievements → Toast
  - Optional route: “See Today’s Progress” → SCR-PROGRESS-VIEW