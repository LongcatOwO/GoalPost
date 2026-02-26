# UC-8 — Ask AI Why Progress Is Stalled (Diagnostics & Proposed Fixes)

ID: UC-8  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user)  
Level: User goal  
Frequency: Medium (when momentum drops; weekly cadence recommended)

Traceability (SRS):
- Core Use Case: UC-8 (Ask AI why progress is stalled; review diagnostic and proposed fixes)
- Functional Requirements: FR-24 (analyze failures and propose corrective actions), FR-25 (“why system not working” diagnostic), FR-26 (confidence/assumptions + user acceptance before saving)
- Supporting: FR-6..8 (Routines/streaks), FR-9..11 (Scheduling/reminders), FR-12..14 (Reviews & summaries), FR-15..17 (Progress/analytics), FR-20..22 (Journaling & reasons), FR-33..36 (Settings/Data)
- Acceptance (related): AC-6 (AI suggests candidates with rationale; user can accept/edit); UC-5 night review integration

Related Screens:
- SCR-AI-DIAGNOSE: Diagnostics entry, summary, and CTA
- SCR-DIAGNOSE-REFINE: Scope, consent, and signal selection
- SCR-DIAGNOSE-RESULT: Patterns, hypotheses (with confidence/assumptions), contributing factors
- SCR-PROPOSED-FIXES: Actionable recommendations with edit/accept controls
- SCR-PREVIEW-FIXES: Review & confirm changes (no auto-apply without consent)
- SCR-GOAL-DETAIL: Goal context (analytics tab)
- SCR-PROGRESS-HUB: Portfolio context and entry
- SCR-REVIEW-NIGHT: Integration point for end-of-day diagnostics
- SCR-USAGE: AI usage/cost panel (metadata only)
- SCR-OFFLINE: Offline banner/modal
- SCR-ERROR: Inline validation/recovery

Non-Functional Reminders:
- NFR-1 Usability: Typical run ≤ 2 minutes; ≤ 5 taps to accept common fixes
- NFR-2 Performance: Target 6–8s for results; UI responsive with cancel/timeout
- NFR-4 Availability: Offline disables AI; show manual guidance; never block core flows
- NFR-5 Security & NFR-6 Privacy: Redact PII; show disclaimers; no background calls without user intent; consent gates for signal use
- NFR-9 Observability: Metadata-only logs (durations, counts); never log raw journal text
- NFR-12 Cost: Visible usage/limits; graceful rate-limit handling

---

## Goal/Benefit
Provide a fast, privacy-respecting diagnostic to explain why progress has slowed (e.g., missed routines, overdue milestones, repeated blockers) and offer concrete, user-controlled corrective actions (rescope, reschedule, split work, cadence tweaks, reminders, reflection prompts). User remains in full control; no changes occur without explicit acceptance.

Outcome: A diagnostic artifact (observations, hypotheses with confidence/assumptions) is saved. The user may accept zero or more proposed fixes, each mapped to concrete edits (e.g., reschedule X, reduce cadence Y, create micro-milestones), all reviewed before persisting.

---

## Preconditions
- At least one Goal or Routine with recent activity exists.
- Local data available; connectivity available for AI (offline shows manual guidance).
- AI access configured and within usage/rate limits (policy-appropriate proxy).
- User consented to include specific signals (journals/reasons, streaks, overdue stats). Defaults conservative; can widen scope in SCR-DIAGNOSE-REFINE.

## Triggers
- From SCR-GOAL-DETAIL: “Why am I stuck?” CTA
- From SCR-PROGRESS-HUB: “Diagnose stall” (portfolio-level)
- From SCR-REVIEW-NIGHT: “Why stuck?” after logging misses
- From repeated misses banner (“We’ve noticed repeated misses on Routine X”)

## Postconditions (Success)
- AI Artifact stored with observations, hypotheses (with confidence/assumptions), proposed fixes.
- If user accepts fixes, mapped changes are persisted (no partial silent edits).
- User returns to context with visible updates (schedules, cadences, new micro-milestones) and an audit note of accepted changes.

## Postconditions (Failure/Abort)
- If user cancels or accepts no fixes, no entity changes occur (artifact may still be saved as “no_accept” for audit if enabled).
- All in-progress edits remain discardable until confirmation.

---

## Primary Flow A — Diagnose a Single Goal

1) Entry — SCR-AI-DIAGNOSE
   - Context: Selected Goal summary (milestone %, overdue, recent streaks, last review date).
   - CTA: “Run Diagnostic”.

2) Scope & Consent — SCR-DIAGNOSE-REFINE
   - Scope: [This Goal], Time window: [Last 2/4/8 weeks], Signals to include (toggle):
     - Milestone delays/overdues (counts/ages)
     - Routine streak breaks and adherence
     - Reason tags from missed items (aggregated counts only)
     - Morning/Night review adherence (count of reviews run)
   - Privacy note: “No journal free-text will be sent. Only counts/tags/metrics.”
   - CTA: “Generate” (shows usage/cost metadata and disclaimer).

3) Generate — SCR-AI-DIAGNOSE (loading)
   - Non-blocking loader; cancel available.
   - Timeout target 8s; on timeout offer Retry or Manual Tips.

4) Results — SCR-DIAGNOSE-RESULT
   - Sections:
     - Observations: “3 milestones overdue >10 days; 4/7 routines missed on weekdays; morning review run 1/7 days.”
     - Hypotheses (with Confidence Low/Med/High) and Assumptions:
       - Example: “Scope too large for weekdays” (High), Assumption: “Workdays are time-constrained”
       - Example: “Unclear next steps for Milestone B” (Med), Assumption: “No sub-steps defined”
     - Contributing Factors: top missed reason tags (counts), streak break patterns, overdue age distribution.
     - Data Used (transparent): list included signals and time window.

5) Proposed Fixes — SCR-PROPOSED-FIXES
   - 4–6 actionable items (edit/view per fix):
     - Split: “Split Milestone B into 3 micro-milestones (2–3h each)”
     - Reschedule: “Move overdue Milestone A to Sat 10:00; add 30m buffer”
     - Cadence: “Reduce Routine X to M/W/F; shift to earlier 07:00”
     - Priorities: “Limit ‘Top Priorities’ to 2 on weekdays for 2 weeks”
     - Planning: “Add Weekly Planning routine (Sun 18:00, 15m)”
     - Reflection: “Add ‘Define next step’ prompt to Night Review for this goal”
   - Per-fix controls: Accept, Edit (tweak time/cadence/count), Dismiss
   - Bulk: Accept None/All (Still requires preview before persist)

6) Review & Confirm — SCR-PREVIEW-FIXES
   - Show diff of pending changes:
     - Create micro-milestones: titles, optional target offsets
     - Update routine cadence/times
     - Reschedule due dates (+ reminders)
     - Add weekly planning routine
     - Add review prompt rule (non-destructive UI config)
   - Validation: Conflicting times, invalid dates, duplicate titles highlighted inline.
   - CTA: “Apply Changes” (or Cancel).

7) Persist & Route
   - Persist accepted changes in a single transaction.
   - Save AI Artifact with accepted flags and edited params.
   - Toast: “Applied 4 fixes. You can undo in History.” (if undo supported)
   - Route back to SCR-GOAL-DETAIL (analytics updates immediately).

Result: FR-25 and FR-26 satisfied with clear confidence/assumptions and explicit acceptance before changes.

---

## Primary Flow B — Portfolio-Level Diagnostic (All Goals)

1) Entry — SCR-PROGRESS-HUB → “Diagnose stall”
2) Scope — SCR-DIAGNOSE-REFINE
   - Scope: All Goals, Time window: 4/8 weeks, Signals same as above (aggregated).
3) Results — SCR-DIAGNOSE-RESULT
   - Portfolio Observations: goals with highest overdue load, routines with repeated streak breaks.
   - Hypotheses and Factors ranked across goals.
4) Proposed Fixes — SCR-PROPOSED-FIXES
   - Batched recommendations grouped by goal; user can expand per goal.
   - Mapping pane to select targets for bulk reschedules or routine cadence changes.
5) Preview & Confirm — SCR-PREVIEW-FIXES
   - Show grouped diffs; allow partial acceptance per goal.
6) Persist & Route
   - Save changes; artifact links to impacted goal IDs.
   - Return to SCR-PROGRESS-HUB with refreshed cards.

Result: Cross-goal insight with safe, selective application.

---

## Primary Flow C — Night Review Integration

1) From SCR-REVIEW-NIGHT after tagging misses → “Why stuck?”
2) Scope defaults: Today + last 7 days; signals: missed reason tags, streak breaks, overdues.
3) Results emphasize near-term corrective actions:
   - 2–3 small next-steps for tomorrow morning plan.
   - 1 cadence tweak or 1 reschedule suggestion.
4) Accept/Edit fixes → Preview → Apply.
5) Option: “Auto-carry suggested next-steps to tomorrow’s Morning Review draft.”

Result: Lightweight, end-of-day corrective loop that feeds tomorrow.

---

## Alternate Flows

A1) Privacy Narrowing
- User deselects journals/reason tags; diagnostic relies on non-journal signals.
- Banner: “With fewer signals, hypotheses may be less precise.”

A2) Dry Run (No Apply)
- User views diagnostic and closes without accepting any fixes (artifact saved, no changes).

A3) Template-Aware Suggestions
- If goal was created from a template, show tag “Template-aware” and propose fixes consistent with template phases (e.g., “complete Foundation phase before adding more routines”).

A4) Gentle Mode
- Reduce scope of proposed changes (e.g., only reschedule and add one micro-step). Toggle in refine screen.

A5) Undo Path
- If the system supports undo/history, show “Undo last diagnostic changes” in success toast.

---

## Exception Flows

E1) Offline / AI Unavailable
- Disable “Generate”; show “AI diagnostics unavailable offline.”
- Offer Manual Tips card: 
  - “Reduce scope for top overdue milestone”
  - “Add Weekly Planning routine”
  - “Limit to Top 2 priorities tomorrow”
- User can open related editors directly.

E2) Timeout / Rate Limit / Quota Exceeded
- Banner with cooldown; preserve refine inputs and partial UI state.
- Offer narrower time window or fewer signals.

E3) Insufficient Data
- If signals too sparse, show:
  - “Not enough recent data to diagnose confidently.”
  - Offer: add sub-steps to key milestones; turn on morning/night reviews; set routine cadences.

E4) Safety/Content Filter
- If any output flagged, hide item and show:
  - “One suggestion was filtered. Adjust scope or refine request.”

E5) Storage Error on Apply
- Modal: “Couldn’t apply changes. Try again.”
- Options: Retry, Copy diagnostics (redacted), Cancel (keep selections).

---

## Data Created/Updated

- AI Artifact (diagnostic):
  - { id, scope: goal_id|portfolio, created_at, window, signals_used[], observations[], hypotheses: [{text, confidence, assumptions[]}], proposed_fixes: [{fix_id_local, type, params, accepted:boolean, edited:boolean}], duration_ms, status: success|timeout|error, cost_metadata? }

- Entity Updates (if user accepts fixes):
  - Milestone: { title/description edits, target_date updates, status unchanged unless explicitly set }
  - New Micro-Milestones (optional): { plan_id, title, target_offset?, status: Pending }
  - Routine: { cadence (daily/weekly/custom), schedule time, paused? false }
  - Reminder: { due_at updates; new reminders for rescheduled milestones }
  - Review Rules (local UI config): { goal_id, night_prompt_additions: ["Define next step"] }
  - New “Weekly Planning” Routine (optional): { title, cadence: weekly, schedule: Sun 18:00 }

- Audit/History (optional):
  - { artifact_id, changes_applied[], applied_at, undo_ref? }

Note: No journal free-text is transmitted in prompts. Only aggregated counts/tags/metrics are used.

---

## Business Rules & Constraints

- BR-1: No changes are applied without explicit user acceptance (FR-26).
- BR-2: Show confidence and assumptions for each hypothesis and recommendation (FR-26).
- BR-3: Enforce request timeout (e.g., 8s); provide manual fallback.
- BR-4: Respect usage/rate limits; show usage metadata; never run in background without user intent (NFR-12).
- BR-5: Apply all accepted fixes transactionally; if any fails, roll back and show error.
- BR-6: Template structures remain immutable; fixes create user-owned copies/edits (FR-29).
- BR-7: Privacy-first: journals’ free-text never sent; only opt-in aggregated signals (NFR-6).
- BR-8: Edits are reversible where history/undo is supported; otherwise, provide clear instructions to revert manually.

---

## Analytics & Telemetry (Privacy-Respecting)

- Event: ai_diagnostic_invoked { scope: goal|portfolio, window_weeks, signals_count, duration_ms, success:boolean, timeout:boolean }
- Event: ai_diagnostic_result_viewed { hypotheses_count, avg_confidence }
- Event: ai_diagnostic_fixes_reviewed { proposed:n, accepted:m, edited:k }
- Event: ai_diagnostic_applied { changes_total, types:[reschedule,split,cadence,planning,review_rule] }
- Event: ai_usage_limit_reached { limit_type }
- Event: ai_error { type: timeout|rate_limit|safety|other }

Note: Hash or omit entity IDs; never include free-text or PII in telemetry (NFR-9).

---

## Acceptance Criteria (Derived)

- AC-8.1: User can run an AI diagnostic for a goal or portfolio and see observations, hypotheses, and contributing factors with confidence and assumptions (FR-25, FR-26).
- AC-8.2: System proposes at least two actionable fixes; user can accept, edit, or dismiss each before saving (FR-24, FR-26).
- AC-8.3: On confirm, only accepted fixes are persisted; no auto-apply occurs without consent (FR-26).
- AC-8.4: If AI is unavailable (offline/limits), the user is guided with manual tips and can still reach editors to make equivalent changes (NFR-4, NFR-12).
- AC-8.5: Diagnostic artifacts are stored with signals used and confidence/assumptions for audit; journals’ free-text is never transmitted (NFR-6, NFR-9).

---

## Open Questions / TBD

- Should we allow a “conservative vs. aggressive” fix strategy toggle and what’s the default?
- What is the minimal signals set to produce a “confident enough” hypothesis badge?
- Do we enable an “auto-carry accepted next-steps to tomorrow” toggle by default?
- How do we prioritize fix types when multiple are applicable (scope vs. schedule vs. cadence)?
- Should portfolio diagnostics highlight the “one most impactful change” badge?

---

## UI State Map

- ST-UC8-01: SCR-AI-DIAGNOSE (entry, CTA)
- ST-UC8-02: SCR-DIAGNOSE-REFINE (scope/signals/consent)
- ST-UC8-03: SCR-DIAGNOSE-RESULT (observations, hypotheses, factors)
- ST-UC8-04: SCR-PROPOSED-FIXES (actions with accept/edit)
- ST-UC8-05: SCR-PREVIEW-FIXES (review & confirm)
- ST-UC8-ERR: SCR-OFFLINE (banner), SCR-USAGE (limits), SCR-ERROR (validation/storage)

---

## Wireflow (Textual)

- From SCR-GOAL-DETAIL → “Why am I stuck?”
  - SCR-AI-DIAGNOSE → “Run Diagnostic”
    - SCR-DIAGNOSE-REFINE (select signals, window, consent) → “Generate”
      - (Loading; cancel/timeout available)
      - SCR-DIAGNOSE-RESULT (observations, hypotheses with confidence/assumptions)
        - “See Proposed Fixes” → SCR-PROPOSED-FIXES (accept/edit/dismiss per fix; bulk accept none/all)
          - “Review & Confirm” → SCR-PREVIEW-FIXES (diff, validate conflicts)
            - “Apply Changes”
              - Persist entities + AI Artifact → Toast → Return to SCR-GOAL-DETAIL (analytics refresh)

- From SCR-PROGRESS-HUB → “Diagnose stall”
  - Same flow, scope = portfolio; mapping fixes by goal during preview.

- From SCR-REVIEW-NIGHT → “Why stuck?”
  - Defaults to short window; proposes 2–3 near-term actions
  - Accept/edit → Preview → Apply → Option to seed tomorrow’s Morning Review.