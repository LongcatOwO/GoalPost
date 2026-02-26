# UC-2 — Break Down a Goal into Milestones and Set Targets

ID: UC-2  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user)  
Level: User goal  
Frequency: High (immediately after creating a goal; recurring while refining)

Traceability (SRS):
- Core Use Case: UC-2
- Functional Requirements: FR-1..5 (Goals/Plans/Milestones), FR-9..11 (Scheduling/Reminders), FR-15..17 (Progress/Analytics), FR-18..19 (Achievements, indirect), FR-23..26 (AI assist, optional)
- Acceptance: 8.1 (Create goal with plan and 2 milestones, edit milestone target, mark complete), AC-5 (Progress shows milestone %)

Related Screens:
- SCR-GOAL-DETAIL: Goal Detail (overview of plan/milestones, progress)
- SCR-PLAN-EDITOR: Plan Editor (create/edit one or more plans)
- SCR-MILESTONE-EDITOR: Milestone Editor (title, description, target date, success criteria)
- SCR-MILESTONE-QUICK-ADD: Inline quick add in plan/milestone list
- SCR-SCHEDULE-PICKER: Date/Time picker and reminder options
- SCR-AI-SUGGEST: AI suggestions for milestones (optional)
- SCR-NOTIFICATION-CENTER: Upcoming/overdue reminders
- SCR-ERROR: Inline validation and error states
- SCR-OFFLINE: Offline banner/modal

Non-Functional Reminders:
- NFR-1 Usability: Keep flow ≤ 2 minutes, ≤ 5 taps for adding basic milestones
- NFR-2 Performance: <150ms interactions; list updates and progress recalculation <1s
- NFR-4 Availability: Fully functional offline (no AI); reminders listed in-app
- NFR-6 Privacy: Private by default; no external sharing implied by this flow
- NFR-7 Accessibility: Keyboard/touch-friendly, clear labels, WCAG AA contrast
- NFR-9 Observability: Log metadata (redacted), including state transitions and validation issues

---

## Goal/Benefit
Enable the user to decompose a goal into actionable milestones, each with clear targets (dates and success criteria), so that progress can be tracked, reminders scheduled, and achievements recognized.

Outcome: One or more Plans with Milestones exist under the Goal; each Milestone can have a target date and success criteria set, enabling progress computation and scheduling.

---

## Preconditions
- The user has at least one existing Goal (created via UC-1).
- The Goal may already have a default Plan; if not, the user can create one during this flow.
- Local storage is available. Online connectivity is optional; AI assist requires it (if enabled).

## Triggers
- From SCR-GOAL-DETAIL, the user selects:
  - “Add Milestone” inside an existing Plan, or
  - “Add Plan” then adds milestones to that plan, or
  - “Get AI Suggestions” for milestones (if available).
- From a reminder to define targets (prompt when milestones lack dates).

## Postconditions (Success)
- Plan(s) exist with one or more Milestones.
- Milestones have titles; targets (dates, success criteria) may be set or left unset.
- Optional reminders are created for target dates.
- Progress indicators update (milestone % complete on Goal/Plan).
- Overdue/upcoming items appear in SCR-NOTIFICATION-CENTER.

## Postconditions (Failure/Abort)
- If user cancels before saving, no changes are persisted.
- Draft changes may be cached locally for recovery (optional enhancement).

---

## Primary Flow A — Add Milestones to an Existing Plan
1) Entry — SCR-GOAL-DETAIL
   - User expands a Plan section and taps “Add Milestone”.

2) Create Milestone — SCR-MILESTONE-QUICK-ADD
   - Inputs: Title (required), optional Description.
   - CTAs: “Save & Add Another”, “Save”, “Cancel”.

3) Refine Targets — SCR-MILESTONE-EDITOR (on Save)
   - Optional: Set Target Date (via SCR-SCHEDULE-PICKER).
   - Optional: Define Success Criteria (textual, checkbox list, or numeric threshold).
   - Optional: Add Tags.
   - CTA: “Done”.

4) Persist & Return — SCR-GOAL-DETAIL
   - System saves milestone.
   - Progress recalculates at Plan/Goal level (FR-15).
   - If Target Date set, an in-app reminder is created (FR-9..11).

5) Repeat
   - User adds at least a second milestone to meet MVP guidance.
   - Result: Plan now has 2+ milestones (as recommended).

Result: Goal has a Plan with multiple Milestones and optional targets set.

---

## Primary Flow B — Create a Plan, Then Milestones
1) Entry — SCR-GOAL-DETAIL
   - No plan present or user wants multiple plans. User taps “Add Plan”.

2) Create Plan — SCR-PLAN-EDITOR
   - Inputs: Title (required), Description (optional).
   - CTA: “Create Plan”.

3) Add Milestones
   - On successful plan creation, system focuses milestone list for the new plan.
   - Continue with Primary Flow A steps (Add Milestone, set targets).

Result: New Plan with milestones under the existing Goal.

---

## Primary Flow C — AI-Assisted Milestone Suggestions (Optional)
1) Entry — SCR-GOAL-DETAIL (Plan section) or SCR-PLAN-EDITOR
   - User taps “Get AI Suggestions”.

2) Generate Suggestions — SCR-AI-SUGGEST
   - System sends goal/plan context (title, description, domain/tags) to AI (FR-23).
   - System displays up to 3 milestone suggestions, each with:
     - Title, optional description
     - Rationale and confidence (FR-26)

3) User Selection and Edits
   - User accepts all/some, and may edit titles and descriptions inline.
   - CTA: “Apply to Plan”.
   - If AI unavailable/offline: show banner, allow manual continue.

4) Persist & Targeting
   - On Apply, milestones are added as drafts to the plan.
   - System prompts to set target dates/success criteria (bulk or per-milestone).
   - On confirm, system saves and returns to SCR-GOAL-DETAIL with updated progress.

Result: Drafted milestones accelerated by AI; user-reviewed before saving (FR-26).

---

## Primary Flow D — Edit Target Dates and Mark Milestone Complete
1) Entry — SCR-GOAL-DETAIL
   - User taps an existing milestone.

2) Edit Targets — SCR-MILESTONE-EDITOR
   - Adjust Target Date (via SCR-SCHEDULE-PICKER).
   - Refine Success Criteria.
   - CTA: “Save”.

3) Mark Complete — SCR-MILESTONE-EDITOR or Inline Toggle
   - User toggles Status to “Completed”.
   - Optionally add completion note/photo (FR-30, optional in this flow).

4) Persist & Update
   - System saves changes.
   - Progress updates (e.g., milestone % complete).
   - Achievement check (e.g., “First Milestone” badge per FR-18/19, if applicable).

Result: Acceptance from 8.1 satisfied (edit milestone target date and mark completed).

---

## Alternate Flows
A1) Quick Add Without Targets
- User adds milestones with only titles, no dates/criteria.
- System allows save; progress bases on completion status only.
- Gentle nudge: “Set targets later to schedule reminders.”

A2) Bulk Add
- User enters multiple milestone titles in a multi-line input or repeated quick-add.
- System creates all, then offers batch targeting (date offsets).

A3) Reorder Milestones
- User long-press/drags to reorder within a plan.
- System persists order; no impact on dates/criteria.

A4) Convert or Split
- User duplicates a milestone or splits it into sub-milestones (optional enhancement).
- System maintains references; progress aggregation unaffected.

A5) Delete Milestone
- User chooses “Delete” on a milestone.
- Confirm dialog; if confirmed:
  - System removes milestone and related reminders.
  - Recompute progress.

---

## Exception Flows
E1) Validation Errors
- Missing Title → Inline error; disable Save.
- Invalid Date (e.g., malformed) → Reset field with message.
- Target Date before Goal Start (if goal start modeled) → Warn and allow fix or continue per rule.

E2) Scheduling Conflicts
- If overlapping reminder exists, show non-blocking info; allow overwrite or keep both.

E3) AI Failure/Timeout
- Show banner: “AI suggestions unavailable; proceed manually.”
- Keep user on editor; no data loss.

E4) Offline Mode
- AI disabled; templates still usable if cached.
- All local edits persist; reminders recorded in-app.

E5) Storage Error
- Modal: “Couldn’t save changes. Try again.”
- Options: Retry; Copy details (redacted logs), Cancel (retain draft if possible).

---

## Data Created/Updated
- Plan { id, goal_id, title, description }
- Milestone { id, plan_id, title, description?, target_date?, status: Pending|Completed, success_criteria?, attachments: [] }
- Reminder (optional) { id, entity_ref: milestone_id, due_at, status: Active|Snoozed|Dismissed }
- AI Artifact (optional) { prompt_metadata, suggestions[], rationale, confidence, accepted_at }
- Progress Cache (derived) { goal_id, plan_id, milestone_counts, completion_pct }

---

## Business Rules & Constraints
- BR-1: Milestone title is required.
- BR-2: At least one Plan must exist to host milestones (auto-create a default if needed).
- BR-3: Target Date is optional but recommended; reminders require a date/time.
- BR-4: AI outputs require user acceptance before saving (FR-26).
- BR-5: Templates and AI suggestions create editable drafts; originals remain immutable (FR-29).
- BR-6: Completing a milestone should be reversible (toggle complete/incomplete), with timestamped history (for analytics integrity).

---

## Analytics & Telemetry (Privacy-Respecting)
- Event: milestone_add_initiated { method: “manual|ai|bulk|template”, goal_id, plan_id }
- Event: milestone_saved { has_target: boolean, has_criteria: boolean }
- Event: milestone_validation_error { field, count }
- Event: milestone_completed { milestone_id, from_status, to_status }
- Event: progress_recomputed { goal_id, plan_id, milestone_total, completed_total, pct }

Note: Redact or hash IDs in logs if needed; no PII in payloads (NFR-9).

---

## Acceptance Criteria (Derived)
- AC-2.1: User can add at least two milestones under a plan (satisfying 8.1 acceptance recommendation).
- AC-2.2: User can edit a milestone’s target date and save changes.
- AC-2.3: User can mark a milestone as completed, and progress updates immediately (FR-15).
- AC-2.4: If a target date is set, an in-app reminder is created and appears in the reminder list (FR-9..11).
- AC-2.5: AI-suggested milestones (if enabled) require user review and acceptance before saving (FR-26).

---

## Open Questions / TBD
- Should the system enforce at least two milestones per plan at save, or only recommend?
- Do we support success criteria templates (checkboxes vs. free text) in MVP?
- Should reminders support multiple notifications per milestone (e.g., pre-due nudge and due)?
- How to handle time zones for target dates when traveling (auto-adjust vs. fixed local)?
- Do we show a “confidence” badge or only text in AI suggestions?

---

## UI State Map
- ST-UC2-01: SCR-GOAL-DETAIL (plan expanded)
- ST-UC2-02: SCR-MILESTONE-QUICK-ADD (inline create)
- ST-UC2-03: SCR-MILESTONE-EDITOR (full details and targets)
- ST-UC2-04: SCR-SCHEDULE-PICKER (date/time, reminder)
- ST-UC2-05: SCR-PLAN-EDITOR (create/edit plan)
- ST-UC2-06: SCR-AI-SUGGEST (milestone proposals)
- ST-UC2-ERR: SCR-ERROR (validation), SCR-OFFLINE (banner)

---

## Wireflow (Textual)
- From SCR-GOAL-DETAIL
  - If “Add Milestone” → SCR-MILESTONE-QUICK-ADD → SCR-MILESTONE-EDITOR (targets) → Save → SCR-GOAL-DETAIL (progress updates)
  - If “Add Plan” → SCR-PLAN-EDITOR → Create → (auto-focus new plan) → SCR-MILESTONE-QUICK-ADD → SCR-MILESTONE-EDITOR → Save → SCR-GOAL-DETAIL
  - If “Get AI Suggestions” → SCR-AI-SUGGEST → Accept/Edit → (Optional) SCR-MILESTONE-EDITOR (targets) → Save → SCR-GOAL-DETAIL
  - If Edit existing milestone → SCR-MILESTONE-EDITOR → Change target date → Save → SCR-GOAL-DETAIL
  - If Mark complete (inline or editor) → Persist → Recompute progress → Return to SCR-GOAL-DETAIL