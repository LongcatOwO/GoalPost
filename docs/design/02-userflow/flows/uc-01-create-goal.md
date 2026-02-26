# UC-1 — Create a New Goal (from Scratch or Template)

ID: UC-1  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user)  
Level: User goal  
Frequency: High (onboarding and ongoing)  

Traceability (SRS):
- Core Use Case: UC-1
- Functional Requirements: FR-1..5 (Goals/Plans/Milestones), FR-27..29 (Templates), FR-23..26 (AI assist, optional), FR-33..36 (Settings/Data, indirect)
- Acceptance Criteria: AC-1 (template creates pre-defined structures)

Related Screens (IDs to be defined in screens specs):
- SCR-DASH: Dashboard/Home
- SCR-GOAL-NEW-ENTRY: New Goal Entry (scratch vs template)
- SCR-TEMPLATES: Template Library
- SCR-GOAL-FORM: Create Goal (Scratch) Form
- SCR-GOAL-AI-SUGGEST: AI Suggestions for Milestones (Optional)
- SCR-GOAL-REVIEW: Goal Preview & Confirm
- SCR-GOAL-DETAIL: Goal Detail
- SCR-ERROR: Inline Validation/Error & Recovery
- SCR-OFFLINE: Offline Notification Banner/Modal

Non-Functional Reminders:
- NFR-1 Usability: ≤ 2 minutes, ≤ 5 taps typical when using template
- NFR-2 Performance: <150ms interactions, <1s render
- NFR-4 Availability: Works offline (no AI/template fetch beyond local cache)
- NFR-6 Privacy: Private by default

---

## Goal/Benefit
Enable the user to rapidly create a well-structured goal, either:
- from scratch, with optional AI guidance to draft milestones, or
- from a curated template that pre-populates plans, milestones, and routines.

Outcome: A new Goal exists with at least one Plan and Milestones (and optionally Routines), ready to track and schedule.

---

## Preconditions
- App installed and running; user at least at SCR-DASH.
- Local storage available; no requirement for cloud connectivity.
- For AI suggestions path: AI feature is enabled and usage within limits.
- For Template path: At least one template installed locally (MVP requires ≥3 categories).

## Triggers
- User taps “New Goal” (FAB or button) on SCR-DASH.
- OR User selects “Create from Template” CTA in SCR-DASH quick actions.

## Postconditions (Success)
- A Goal entity is created with:
  - At least 1 Plan (FR-2)
  - At least 2 Milestones (FR-3) or as per template
- If Template was used, Plan/Milestones/Routines are pre-filled (FR-27..29; AC-1).
- User lands on SCR-GOAL-DETAIL with progress and upcoming items visible (FR-5).

## Postconditions (Failure/Abort)
- No persistent changes are saved if user cancels before confirmation.
- Draft state may be cached locally for recovery (optional enhancement).

---

## Primary Flow (Happy Path A — Create from Template)
1. Entry — SCR-GOAL-NEW-ENTRY
   - System shows two primary options: “From Template” and “From Scratch”.
   - Default highlights “From Template” (fast path per MVP usability).

2. Choose Template — SCR-TEMPLATES
   - User browses categories: Fitness, Finance, Education, Diet, Beauty, Health (≥3 guaranteed in MVP).
   - User selects a Template (e.g., “Fitness: Couch to 5K Beginner”).

3. Template Preview (Inline on SCR-TEMPLATES or dedicated section)
   - System shows template summary: example goal title, 1 plan, 2 routines, 3 milestones (example).
   - Actions: “Use Template”, “Customize First”.

4. Confirm Use — SCR-GOAL-REVIEW (Template-applied Draft)
   - System creates a draft Goal pre-filled:
     - Goal: title, domain, default priority/tags (editable)
     - Plan: at least 1 with default description
     - Milestones: target dates unset or suggested ranges
     - Routines: cadence set (daily/weekly/custom)
   - User may:
     - Edit title/description/tags/priority
     - Adjust milestone target dates and success criteria
     - Tweak routine cadence/time

5. Validate & Confirm
   - System validates required fields:
     - Goal.title non-empty
     - Milestones >= 2 with titles (MVP example), target_date optional
   - User taps “Create Goal”.

6. Persist & Route — SCR-GOAL-DETAIL
   - System persists Goal, Plan(s), Milestones, Routine(s).
   - System shows success toast “Goal created”.
   - System shows progress cards: milestone %, routines/streak (initially 0).

Result: AC-1 satisfied if template applied created pre-defined structures.

---

## Primary Flow (Happy Path B — Create from Scratch with Optional AI Assist)
1. Entry — SCR-GOAL-NEW-ENTRY
   - User chooses “From Scratch”.

2. Goal Basics — SCR-GOAL-FORM
   - Fields:
     - Title (required)
     - Description (optional)
     - Domain (select/tag)
     - Priority (Low/Med/High)
     - Tags (optional)
   - CTAs:
     - “Next: Milestones”
     - Secondary: “Get AI Suggestions” (if AI available), “Cancel”

3. Optional AI Suggestions — SCR-GOAL-AI-SUGGEST
   - Precondition: AI enabled and online.
   - System sends goal title/description and domain context to AI.
   - System shows up to 3 milestone suggestions with rationale and confidence (FR-23, FR-26).
   - User can:
     - Accept All → convert to editable milestones
     - Accept Some → pick subset
     - Edit a suggestion inline
     - Skip AI → go manual

   Offline/AI-disabled handling:
   - If AI not available, system shows inline info: “AI suggestions unavailable; proceed manually.”

4. Milestones & (Optional) Plan — SCR-GOAL-FORM (Milestone Section)
   - User ensures at least 1 Plan exists (auto-create default plan “Main Plan” if none).
   - User adds Milestones (≥2 recommended in MVP), with optional target dates and success criteria.
   - CTA: “Review”

5. Review & Confirm — SCR-GOAL-REVIEW
   - System displays a summary:
     - Goal basics
     - Plan
     - Milestones (with ability to edit/reorder)
     - Optional: Add a Routine now? (Quick-add daily/weekly cadence)
   - Validation:
     - Title required
     - ≥1 Plan, ≥1 Milestone (MVP target: 2+ suggested)
   - User taps “Create Goal”.

6. Persist & Route — SCR-GOAL-DETAIL
   - System persists entities and shows the newly created goal detail (FR-5).
   - Toast: “Goal created”.

---

## Alternate Flows
A1. Cancel at Any Time
- User taps “Cancel”.
- System discards draft and returns to SCR-DASH. If a draft exists, prompt: “Save draft?” (optional).

A2. Template Edit Before Apply
- From Template Preview, user taps “Customize First”.
- System opens SCR-GOAL-REVIEW in “template draft” mode with all fields editable before persist.

A3. Routine Deferral
- On Review, user declines to add routine.
- System proceeds; goal created without routines.

A4. Multiple Plans
- User adds multiple plans in Review.
- Validation still only requires ≥1 plan; proceed if valid.

A5. Timeboxing
- If Review > 2 minutes, show tip: “You can edit details later.” No blocking.

---

## Exception Flows
E1. Validation Errors (Goal Basics or Review)
- Missing Title → inline error under Title; disable “Create Goal”.
- Milestone with empty title → inline error; highlight row.

E2. Template Not Found/Corrupted
- On “Use Template” fail → show error toast, remain on SCR-TEMPLATES, allow retry or pick another template.

E3. AI Request Failure/Timeout
- Show non-blocking banner: “AI suggestions unavailable now; continue manually.”
- Keep user on SCR-GOAL-FORM with manual milestone inputs.

E4. Offline Mode
- If offline:
  - From Template path: use locally cached templates only.
  - AI path disabled with info banner.
- Creation still works; all data saved locally (NFR-4).

E5. Storage Error (Rare)
- Show modal: “Couldn’t save your goal. Try again.”
- Offer “Retry” and “Report Issue” (logs redacted per NFR-5/6/9).

---

## Data Created/Updated
- Goal {
  id, title, description, domain, priority, status: “Active”, dates: {created_at, updated_at}, tags
}
- Plan { id, goal_id, title, description }
- Milestone { id, plan_id, title, description?, target_date?, status: “Pending”, success_criteria?, attachments: [] }
- Routine? { id, goal_id|plan_id, title, cadence, schedule, streak: 0, last_completed: null }
- AI Artifact? { prompt_metadata, suggestions[], rationale, confidence } (only if AI used)
- Reminder? Not required at creation; can be added later in scheduling flows (FR-9..11)

---

## Business Rules & Constraints
- BR-1: Goal title is required.
- BR-2: At least one Plan must exist (auto-create if needed).
- BR-3: Milestones optional per entity model, but MVP UX nudges to ≥2 for structure.
- BR-4: Templates are applied as copies; original template remains immutable (FR-29).
- BR-5: AI outputs require user acceptance before saving (FR-26).
- BR-6: Privacy default = Private; nothing shared externally by default.

---

## Analytics & Telemetry (Privacy-Respecting)
- Event: goal_create_initiated { method: “template|scratch” }
- Event: goal_create_ai_invoked { success: boolean, duration_ms }
- Event: goal_create_validation_error { field, count }
- Event: goal_created { method, has_routines: boolean, milestone_count, duration_ms }
- Event: template_applied { template_id, category }

Note: Log metadata only; redact PII (NFR-9).

---

## Acceptance Criteria (Derived)
- AC-1: Creating a goal from a template creates pre-defined plan(s), routine(s), and milestone(s).
- AC-1a: User can review and edit template-applied items before persisting.
- AC-1b: Post-create, SCR-GOAL-DETAIL reflects pre-filled structures.
- Scratch Path: User can create a goal with plan and milestones without template; if AI is available, milestone suggestions can be accepted/edited before save.

---

## Open Questions / TBD
- Should the MVP enforce a minimum of 2 milestones at save time or only nudge?
- Default routine creation on template apply: always include, or ask?
- Should we timebox AI responses with a fixed timeout (e.g., 6–8s) before offering manual fallback?
- Default target dates from template: use offsets vs. unset?
- Naming convention for default plan (“Main Plan”) — configurable per template or global default?

---

## UI State Map (for Screen Specs Reference)
- ST-UC1-01: SCR-GOAL-NEW-ENTRY (method selection)
- ST-UC1-02: SCR-TEMPLATES (list + preview)
- ST-UC1-03: SCR-GOAL-FORM (scratch basics + milestone editor)
- ST-UC1-04: SCR-GOAL-AI-SUGGEST (optional)
- ST-UC1-05: SCR-GOAL-REVIEW (template or scratch draft)
- ST-UC1-06: SCR-GOAL-DETAIL (post-create)
- ST-UC1-ERR: SCR-ERROR (inline), SCR-OFFLINE (banner)

---

## Wireflow (Textual)
- From SCR-DASH → SCR-GOAL-NEW-ENTRY
  - If “From Template” → SCR-TEMPLATES → (Preview) → SCR-GOAL-REVIEW → (Create) → SCR-GOAL-DETAIL
  - If “From Scratch” → SCR-GOAL-FORM → (optional) SCR-GOAL-AI-SUGGEST → SCR-GOAL-FORM (milestones) → SCR-GOAL-REVIEW → (Create) → SCR-GOAL-DETAIL
  - Cancel returns to SCR-DASH at any point before Create