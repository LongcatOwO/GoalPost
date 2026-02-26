# UC-7 — Ask AI to Suggest Steps; Accept/Edit Outputs

ID: UC-7
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)
Scope: GoalPost app (Mobile + Web, single-user)
Level: User goal
Frequency: Medium–High (on new goals or when progress stalls)

Traceability (SRS):
- Core Use Case: UC-7
- Functional Requirements: FR-23 (AI suggests steps/milestones), FR-24 (analyze failures and propose corrective actions), FR-25 (diagnostics synthesis), FR-26 (user acceptance required)
- Supporting: FR-1..5 (Goals/Plans/Milestones), FR-6..8 (Routines), FR-12..14 (Reviews) for context
- Acceptance: AC-6 (AI suggests ≥3 milestone candidates with rationale; user can accept/edit and save)

Related Screens:
- SCR-GOAL-DETAIL: Goal context and CTA to “Get AI Suggestions”
- SCR-AI-SUGGEST: Suggestions UI for milestones/steps with rationale and confidence
- SCR-AI-REFINE: Prompt refinement and constraints (scope, tone, count, time horizon)
- SCR-PLAN-MAP: Map accepted suggestions to Plan/Milestone hierarchy
- SCR-PREVIEW-CHANGES: Review & Confirm to persist drafts/edits
- SCR-OFFLINE: Offline banner/modal (AI disabled)
- SCR-ERROR: Inline validation/recovery
- SCR-USAGE: AI usage/cost panel (metadata, daily limits)
- Alias (where already used): SCR-GOAL-AI-SUGGEST

Non-Functional Reminders:
- NFR-1 Usability: Typical interaction ≤ 2 minutes; ≤ 5 taps to accept a set
- NFR-2 Performance: Results aim to display within 6–8s; UI remains responsive
- NFR-4 Availability: Offline disables AI; manual flows always available
- NFR-5 Security & NFR-6 Privacy: No PII in prompts without consent; redact logs; show disclaimers
- NFR-9 Observability: Log metadata only (prompt length, tokens, duration, errors); no raw content in telemetry
- NFR-12 Cost: Show usage; rate-limit; no background calls without intent; cancel in-flight

---

## Goal/Benefit
Provide fast, structured assistance to break a goal into actionable milestones or steps. The AI proposes candidates with rationale and confidence; the user edits or accepts selectively, then maps them into plans/milestones. This accelerates planning while keeping full human control.

Outcome: Accepted suggestions are saved as editable drafts under a selected Plan/Milestone; an AI Artifact is stored with rationale, confidence, and assumptions.

---

## Preconditions
- A Goal exists (title required; description/domain/tags recommended).
- AI feature is enabled and within usage/rate limits; connectivity available.
- If policy requires API configuration or a managed proxy, it is available; otherwise AI is disabled and UI falls back to manual.

## Triggers
- From SCR-GOAL-DETAIL: Tap “Get AI Suggestions”
- From UC-1/UC-2 flows: “Get AI Suggestions” while creating/editing
- From Reviews (UC-4/UC-5): “Help me plan steps today/tomorrow” (context-limited suggestions)

## Postconditions (Success)
- AI Artifact persisted (prompt metadata, suggestions, rationale, confidence)
- Accepted items exist as drafts under chosen Plan/Milestone
- User returns to context view with “Edit/Refine” affordances

## Postconditions (Failure/Abort)
- If canceled or no items accepted, no changes to entities (Artifact may still be recorded with “no_accept” outcome if configured)
- Draft mapping selections are discarded unless explicitly saved

---

## Primary Flow A — Suggest Milestones for a Goal
1) Entry — SCR-GOAL-DETAIL
   - CTA: “Get AI Suggestions”
   - Inline hint: “AI proposes 3–5 milestones with rationale. You choose what to keep.”

2) Configure Scope — SCR-AI-REFINE
   - Inputs (pre-filled from goal context; all editable):
     - Goal title (required), description, domain/tags
     - Time horizon (e.g., 3 months)
     - Count: 3 (default), range 3–5
     - Output type: “Milestones”
     - Tone: Practical/Concise (default)
   - CTA: “Generate”
   - Show usage panel (today count, estimated tokens); disclaimer link

3) Generate & Display — SCR-AI-SUGGEST
   - Non-blocking loader with cancel; target 6–8s timeout then offer Retry/Manual
   - Show cards per suggestion:
     - Title, optional description
     - Rationale and assumptions
     - Confidence (Low/Med/High)
   - Controls per card:
     - Accept, Edit, Reject
   - Bulk controls:
     - Accept All, Reject All
   - “Regenerate” (new set), “Refine Prompt” (adjust constraints)

4) Edit & Curate
   - On Edit: inline title/description edits; optional success criteria seed
   - De-duplication hint if suggestion overlaps existing milestone
   - Maintain selection state across reorders/edits

5) Map to Plan — SCR-PLAN-MAP
   - Pick existing Plan or create new (“Main Plan” default)
   - Optional: set target date offsets (e.g., +4w, +8w) to pre-fill dates
   - Validate at least one accepted item

6) Review & Confirm — SCR-PREVIEW-CHANGES
   - Summary of new milestones with optional target dates
   - CTA: “Create Milestones”
   - Persist:
     - Milestones as Pending
     - AI Artifact (suggestions, rationale, confidence, accepted flags)
   - Toast: “Milestones created”

7) Route
   - Return to SCR-GOAL-DETAIL (analytics tab updates %; upcoming list populated)

Result: FR-23 and FR-26 satisfied; AC-6 aligns (≥3 candidates shown; user can accept/edit then save).

---

## Primary Flow B — Suggest Sub-steps (Tasks) for a Milestone
1) Entry — From a milestone in SCR-GOAL-DETAIL; action: “Suggest Sub-steps”
2) Scope — SCR-AI-REFINE
   - Inputs: Milestone title/description; desired granularity (S, M, L)
   - Output type: “Steps/Tasks”; Count: 3–7 default 5
3) Suggestions — SCR-AI-SUGGEST
   - Same affordances as Flow A (Accept/Edit/Reject; rationale; confidence)
4) Map & Persist — SCR-PLAN-MAP → SCR-PREVIEW-CHANGES
   - Map accepted steps under the selected Milestone
   - Persist drafts (Task/Activity or Milestone notes per MVP data model)
5) Route — Back to Milestone detail with new sub-steps listed

Result: Accelerated breakdown beneath a milestone with user-controlled acceptance.

---

## Primary Flow C — Prompt Refinement Loop
1) From SCR-AI-SUGGEST tap “Refine Prompt”
2) Adjust: constraints (time horizon, realism), exclude topics, emphasize constraints
3) “Generate” → display new set; previous set remains viewable via tabs (“v1”, “v2”)
4) Accept from any version; dedupe on persist

Result: Iterative co-creation without losing prior candidates.

---

## Alternate Flows
A1) Use With Templates
- If a template already provided a draft, AI focuses on gaps or personalization
- UI tag: “Template-aware suggestions”

A2) Morning/Night Context
- Morning: “Plan my day” proposes 2–3 near-term steps mapped to Today’s Plan
- Night: “Break blockers” proposes smaller next-actions for repeatedly missed items

A3) Low Confidence Suggestions
- If overall confidence Low, show a banner:
  “These may require more context. Consider refining constraints.”

A4) Partial Accept with Manual Add
- Accept some, then add manual items inline before confirming

A5) Multiple Plans
- Allow mapping accepted items across different plans in one session (advanced toggle)

---

## Exception Flows
E1) Offline / AI Disabled
- Show offline banner; “AI suggestions unavailable. Add steps manually.”
- Provide one-click manual quick-add workflow

E2) Timeout / Rate Limit / Quota Exceeded
- Banner with retry cooldown; suggest narrowing scope or trying later
- Never lose user edits; keep refined prompt and selections

E3) Safety/Content Filter
- If unsafe/unhelpful output flagged, hide suggestion and show message:
  “This item was filtered. Refine your request for safer guidance.”

E4) Duplicate/Overlap Detection
- If accepted item duplicates an existing milestone/step (fuzzy title match), warn and allow:
  - Merge (append notes) or Create Anyway

E5) Storage Error on Save
- Modal: “Couldn’t save accepted items. Try again.”
- Options: Retry, Copy diagnostics (redacted), Cancel (keep selections for later)

---

## Data Created/Updated
- AI Artifact:
  - { id, goal_id, milestone_id?, created_at, prompt_metadata: { fields_used, tokens_estimate? }, suggestions: [{ id_local, text/title, description?, rationale, confidence, accepted: boolean, edited: boolean }], assumptions?, version_tag, cost_estimate?, duration_ms, status: success|timeout|error }
- Milestone (Primary Flow A):
  - { id, plan_id, title, description?, target_date?, status: Pending, success_criteria? }
- Step/Task (Primary Flow B; or milestone notes per MVP):
  - { id, milestone_id, title, description?, status: Pending }
- View preferences (local):
  - last_scope, last_count, tone, defaults

Note: Log/telemetry stores metadata only; redact free-text suggestion bodies.

---

## Business Rules & Constraints
- BR-1: AI outputs require explicit user acceptance before persisting (FR-26)
- BR-2: Default suggestion count is 3 for milestones; cap at 5
- BR-3: Enforce a max processing time per request (e.g., 8s); prompt user to refine or continue manually
- BR-4: Respect daily usage limits; visible meter; block when exceeded with clear messaging (NFR-12)
- BR-5: Preserve user edits to suggestions across regenerations; don’t discard without confirmation
- BR-6: Templates remain immutable; accepted AI items are user-owned drafts (FR-29)
- BR-7: No background AI calls; only on explicit user intent (NFR-12)
- BR-8: Privacy-first; sensitive content not sent unless user includes it intentionally; show disclaimer link (NFR-6)

---

## Analytics & Telemetry (Privacy-Respecting)
- Event: ai_suggest_invoked { scope: “goal|milestone”, count_requested, duration_ms, success, timeout }
- Event: ai_suggest_result_viewed { suggestions_count, avg_confidence }
- Event: ai_suggest_selection { accepted: n, rejected: m, edited: k }
- Event: ai_suggest_persisted { created_milestones: n, created_steps: m }
- Event: ai_usage_limit_reached { limit_type }
- Event: ai_error { type: “timeout|rate_limit|safety|other” }

Note: Hash or omit entity IDs; do not include suggestion text; redact PII.

---

## Acceptance Criteria (Derived)
- AC-7.1: User can request AI suggestions for a goal and receive at least 3 milestone candidates with rationale and confidence (FR-23, FR-26)
- AC-7.2: User can accept, edit, or reject suggestions individually; accepted items map to a chosen plan (FR-26)
- AC-7.3: On confirm, accepted items are persisted as drafts; user returns to context with success feedback
- AC-7.4: If AI is unavailable (offline/limits), user is guided to a manual add flow without data loss (NFR-4, NFR-12)
- AC-7.5: Usage/cost metadata is visible; no background calls without user action (NFR-12)

---

## Open Questions / TBD
- Should we expose a “creativity vs. practicality” slider or keep a fixed prompt style in MVP?
- Do we show token/cost estimates in all builds or only in a diagnostics panel?
- How aggressively should we deduplicate suggestions vs. allow near-duplicates for user judgment?
- Should accepted suggestions auto-schedule draft target dates using offsets, or leave unset by default?
- Where to persist user prompt presets (per-goal vs. global defaults)?

---

## UI State Map
- ST-UC7-01: SCR-GOAL-DETAIL (CTA entry)
- ST-UC7-02: SCR-AI-REFINE (scope & constraints)
- ST-UC7-03: SCR-AI-SUGGEST (results with Accept/Edit/Reject, regenerate, refine)
- ST-UC7-04: SCR-PLAN-MAP (map accepted items to plan/milestone)
- ST-UC7-05: SCR-PREVIEW-CHANGES (review & confirm)
- ST-UC7-ERR: SCR-OFFLINE (banner), SCR-ERROR (inline/modals), SCR-USAGE (limits)

---

## Wireflow (Textual)
- From SCR-GOAL-DETAIL → “Get AI Suggestions”
  - SCR-AI-REFINE → “Generate”
    - SCR-AI-SUGGEST
      - Accept/Edit/Reject items
      - Optional: “Refine Prompt” → SCR-AI-REFINE → Generate (versioned sets)
      - Optional: “Regenerate” (new set)
      - “Next”
        - SCR-PLAN-MAP (select plan or create new; optional date offsets)
        - SCR-PREVIEW-CHANGES → “Create”
          - Persist milestones/steps + AI Artifact → Toast
          - Return to SCR-GOAL-DETAIL (updated analytics and lists)
- If Offline/Limit/Timeout:
  - Show banner → Offer “Add manually” (quick-add) → Return on save