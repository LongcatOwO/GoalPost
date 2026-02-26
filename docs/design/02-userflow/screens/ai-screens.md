# GoalPost — AI Assist and Diagnostics Screen Specifications

File: docs/design/02-userflow/screens/ai-screens.md
Scope: AI-assisted planning (suggestions) and AI diagnostics (why progress is stalled), including usage/limits
Traceability:
- SRS Functional: FR-23 (AI suggests steps/milestones), FR-24 (analyze failures & corrective actions), FR-25 (“why system not working” diagnostics), FR-26 (confidence/assumptions + user acceptance before saving)
- SRS Non-Functional: NFR-1..7, NFR-9, NFR-12 (usability, performance, privacy, observability, usage/cost control)
- Core Use Cases: UC-7 (Ask AI to suggest steps), UC-8 (Ask AI why progress is stalled)
- References: Screens in goal-screens.md and routine-review-progress-screens.md

Conventions
- Mobile-first screen definitions; responsive web adapts to wider layouts with split panes where applicable.
- Offline-first: AI features are disabled gracefully when offline or when limits are hit; manual alternatives are always available.
- Privacy-first: No PII is sent unless explicitly provided by the user; telemetry is metadata-only (never raw prompt/response/journal text).
- Acceptance gating: All AI outputs require user review and explicit acceptance before saving (FR-26).

Global Components
- App Bar, Banner (offline/info/warning/error), Card, List Row, Modal/Bottom Sheet, Side Panel, Tabs/Pills, Date/Time Picker, Text Inputs.
- Async Controls: Non-blocking loaders with Cancel; retry affordances; visible request timeouts; never block navigation.
- Usage/Limit Surface: Inline meter and limits panel.

Performance Targets (NFR)
- UI interactions <150ms; initial render <1s.
- AI request: target visible results ≤ 6–8 seconds; cancelable; show timeout and retry/manual fallback.
- Debounce repeated triggers to avoid duplicate calls.

Accessibility (A11y)
- WCAG AA contrast; fully keyboard operable; roles/labels for all controls.
- Live announcements for async state changes (loading, result received, errors).
- Focus preserved across modal/sheet opens/closes; clear heading structure.

Telemetry (Privacy-Respecting)
- Events capture metadata only (e.g., counts, durations, result status, confidence bands).
- No storage of raw AI prompts or responses; no PII or free-text analytics.

---

## SCR-AI-REFINE — Scope & Prompt Refinement

Purpose
- Configure the AI request using context from the selected Goal/Milestone, set result count, time horizon, and tone. Ensures safe, controlled inputs.

Entry
- From SCR-GOAL-DETAIL (Get AI Suggestions)
- From SCR-GOAL-FORM (Scratch path → “Get AI Suggestions”)
- From Milestone detail (Suggest sub-steps)

Primary Data
- Pre-filled context: Goal title (required), description, domain/tags
- Options: Output type (Milestones vs. Steps), Count (3 default; 3–5 for milestones; 3–7 for steps), Time horizon (e.g., 1–3 months), Tone (Practical/Concise default)

Layout
- App Bar: “AI Assist — Configure”
- Section: Context
  - Goal Title (required, editable)
  - Description (optional)
  - Domain/Tags (chips)
- Section: Output
  - Type: Milestones | Steps (radio)
  - Count: Stepper (default per type)
  - Time horizon: Dropdown (2w, 1m, 3m, custom)
  - Tone: Dropdown (Practical, Supportive)
- Section: Usage & Safety
  - “No PII will be sent unless you add it.” link to disclaimer
  - Usage meter snippet: requests left today / soft cap
- Footer: “Generate” (primary), “Cancel”

States
- Offline: Banner “AI unavailable offline.” Generate disabled; link to manual add.
- Limits: Banner “Daily limit reached.” Generate disabled; learn-more link.

Key Actions
- Adjust context/constraints, Generate, Cancel

Validation/Constraints
- Title is required; Count within allowed range; Time horizon valid
- Debounce subsequent Generate clicks; single in-flight request allowed

Instrumentation
- Event: ai_refine_opened { context: goal|milestone }
- Event: ai_generate_clicked { type, count, horizon }

A11y
- Inputs labeled; error messages bound via aria-describedby; focus moves to next screen on Generate

Privacy
- Show short disclaimer with link to full policy; no raw text analytics

---

## SCR-AI-SUGGEST — Suggestions (Accept/Edit/Reject)

Purpose
- Present AI-proposed milestones/steps with rationale and confidence; allow per-item Accept/Edit/Reject and bulk operations.

Entry
- From SCR-AI-REFINE after Generate

Primary Data
- Suggestion list: [{ title, description?, rationale, confidence: Low|Med|High }]
- Selection state: accepted/rejected/edited flags
- In-flight request state: loading/canceled/timeout/error

Layout
- App Bar: “AI Suggestions”
  - Actions: “Refine Prompt” (side panel), “Regenerate”
- Results Area
  - While loading: non-blocking loader with Cancel
  - Suggestion Cards (list):
    - Title (editable on “Edit”)
    - Optional description (inline editor)
    - Rationale (collapsed summary with “More”)
    - Confidence chip: Low|Med|High with tooltip
    - Controls: Accept, Edit, Reject
  - Empty-state on failure: Retry + “Add manually” link
- Footer
  - “Next: Map to Plan” (enabled when ≥1 accepted)
  - Secondary: “Reject All”, “Cancel”

States
- Timeout: Banner “No response in time. Try again or refine.”
- Offline/Limit discovered post-trigger: Non-blocking error; disable new calls

Key Actions
- Accept/Edit/Reject per item; Accept All/Reject All; Regenerate; Refine Prompt; Next

Validation/Constraints
- At least one accepted to proceed; edits persist in-memory across regenerations (versions tab optional in advanced mode)

Performance
- Truncate rationale to keep UI responsive; expand on demand

Instrumentation
- Event: ai_suggest_viewed { suggestions_count, avg_confidence }
- Event: ai_suggest_selection { accepted, rejected, edited }
- Event: ai_regenerate_clicked

A11y
- Card headings are semantic; action buttons keyboard focusable; selection state announced

Privacy
- No raw content in telemetry; caution tip “Review suggestions before saving”

Acceptance Mapping
- UC-7, FR-23, FR-26, AC-6 (≥3 milestone candidates possible; user can accept/edit and save later)

---

## SCR-PLAN-MAP — Map Accepted Items to Plan/Milestone

Purpose
- Map accepted suggestions into the plan hierarchy (choose existing Plan or create a new one). Optional date offsets for milestones.

Entry
- From SCR-AI-SUGGEST “Next: Map to Plan”

Primary Data
- Accepted items list
- Available Plans for selected Goal
- Optional target date offsets (for milestones)

Layout
- App Bar: “Map to Plan”
- Section: Target
  - Plan: Dropdown of existing plans; “+ New Plan” (modal title/description)
  - For Steps mode (Milestone context): show selected Milestone name (read-only)
- Section: Apply Options
  - Target date offsets (for milestones): None | +2w | +4w | +8w | Custom
  - Apply offset: All | Per-item toggle
- Preview List
  - Accepted items with indicators for mapping and date offsets
- Footer
  - “Review & Confirm” (primary)
  - “Back”, “Cancel”

States
- No plans yet: Prompt to create “Main Plan” inline
- Invalid mapping: Inline error until Plan selected

Validation/Constraints
- Requires a Plan for milestones; Steps require parent Milestone context

Instrumentation
- Event: ai_plan_map_opened { items_count }
- Event: ai_plan_map_confirmed { plan_selected, items_count, offset_used }

A11y
- Form controls labeled; previews readable; errors announced

Privacy
- No external calls; local-only mapping

---

## SCR-PREVIEW-CHANGES — Review & Confirm (Create from Suggestions)

Purpose
- Final review of created items before persisting as user-owned drafts. Requires explicit acceptance.

Entry
- From SCR-PLAN-MAP

Primary Data
- List of items to be created with titles, descriptions, target dates (if any), target Plan/Milestone
- AI Artifact metadata placeholder (will be saved upon confirm)

Layout
- App Bar: “Review Changes”
- Summary
  - Count of items by type (Milestones/Steps)
  - Target Plan/Milestone labels
- Change List
  - Each item row: Title, optional description, target date (if set), edit inline if needed
- Footer
  - “Create Items” (primary)
  - “Back”, “Cancel”

States
- Storage error: Modal “Couldn’t save. Try again.” with Retry

Validation/Constraints
- Title non-empty; duplicates detected (warn: Merge or Create anyway)

Instrumentation
- Event: ai_preview_opened { items_count }
- Event: ai_preview_confirmed { created_milestones, created_steps }

A11y
- Changes are listed under headings; confirm button focusable; live announcement on success

Privacy
- Saves AI Artifact metadata (counts, confidence, acceptance) without raw bodies

Acceptance Mapping
- UC-7, FR-26 (user acceptance), FR-23 (created items)

---

## SCR-AI-DIAGNOSE — Diagnostic Entry

Purpose
- Provide an entry point to run a diagnostic on a Goal (or portfolio) when progress stalls. Routes to refine signals and consent.

Entry
- From SCR-GOAL-DETAIL: “Why am I stuck?”
- From SCR-PROGRESS-HUB: “Diagnose stall”
- From SCR-REVIEW-NIGHT: “Why stuck?” CTA

Primary Data
- Context snapshot: milestone %, overdue counts, recent streaks, last review run

Layout
- App Bar: “AI Diagnostics”
- Context Card: Goal/Portfolio snapshot
- CTA: “Run Diagnostic”
- Secondary: “Manual Tips” (offline or if user prefers no AI)

States
- Offline: Banner “AI diagnostics unavailable offline.” Only Manual Tips enabled
- Limits reached: Banner and disabled CTA; learn-more link

Instrumentation
- Event: ai_diagnose_opened { scope: goal|portfolio }

A11y
- Snapshot metrics accessible with text values; CTA clearly labeled

Privacy
- Shows consent notice for signals selection next

---

## SCR-DIAGNOSE-REFINE — Select Signals & Consent

Purpose
- Specify diagnostic scope, time window, and which signals to include. Reinforces privacy controls (aggregated data only; no journal free-text).

Entry
- From SCR-AI-DIAGNOSE “Run Diagnostic”

Primary Data
- Scope: This Goal | All Goals (portfolio)
- Time Window: last 2/4/8 weeks (custom)
- Signals (toggles):
  - Milestone overdues/delays (counts/ages)
  - Routine streak breaks/adherence
  - Missed reason tags (aggregated counts only)
  - Morning/Night review adherence counts
- Consent note: “No journal free-text will be sent.”

Layout
- App Bar: “Select Signals”
- Scope/Window selectors
- Signal toggles with helper text
- Usage meter snippet and disclaimer link
- Footer: “Generate” (primary), “Back”

States
- If insufficient data likely: subtle tip “Results may be less precise with limited data”

Validation/Constraints
- At least one signal selected; window valid

Instrumentation
- Event: ai_diagnose_refine_opened { default_scope }
- Event: ai_diagnose_generate_clicked { window_weeks, signals_count }

A11y
- Toggles accessible and labeled; consent text programmatically associated

Privacy
- Privacy note reiterated; link to policy

---

## SCR-DIAGNOSE-RESULT — Observations & Hypotheses

Purpose
- Present AI observations (facts), hypotheses (with confidence/assumptions), and contributing factors based on selected signals.

Entry
- From SCR-DIAGNOSE-REFINE “Generate”

Primary Data
- Observations (facts from aggregated signals)
- Hypotheses [{ text, confidence Low|Med|High, assumptions[] }]
- Contributing factors: top reason tags, overdue age distribution, streak patterns
- Data Used: echo back included signals and window

Layout
- App Bar: “Diagnostic Results”
  - Action: “See Proposed Fixes”
- Sections
  - Observations: bullet list; concise
  - Hypotheses: list with confidence badges; collapsible assumptions
  - Contributing Factors: chips with counts; small charts (optional)
  - Data Used: plain list
- Footer
  - “See Proposed Fixes” (primary)
  - “Back”, “Close”

States
- Timeout/error: Banner with Retry or offer Manual Tips

Instrumentation
- Event: ai_diagnose_result_viewed { hypotheses_count, avg_confidence }

A11y
- Confidence conveyed via text labels and not only color; lists are structured

Privacy
- No raw journal text; only aggregates shown

Acceptance Mapping
- UC-8, FR-25, FR-26 (confidence/assumptions)

---

## SCR-PROPOSED-FIXES — Actionable Recommendations

Purpose
- Offer concrete, safe-to-apply recommended changes derived from the diagnostic; user can accept, edit parameters, or dismiss.

Entry
- From SCR-DIAGNOSE-RESULT “See Proposed Fixes”

Primary Data
- Proposed fixes [{ type, params, rationale, editable_fields[], confidence }]
  - Examples: Split Milestone, Reschedule Milestone, Adjust Routine Cadence/Time, Add Weekly Planning routine, Add Night Review prompt

Layout
- App Bar: “Proposed Fixes”
  - Actions: Accept None | Accept All (both open preview)
- Fix List
  - Card per fix:
    - Title (e.g., “Split Milestone B into 3 micro-milestones”)
    - Short rationale; confidence chip
    - Inline editable params (e.g., new date/time, cadence days)
    - Controls: Accept, Edit (enabled by default), Dismiss
- Footer
  - “Review & Confirm” (enabled if any accepted)
  - “Back”, “Close”

States
- Conflicting params (e.g., overlapping times): inline warnings

Validation/Constraints
- At least one fix accepted to continue
- Param ranges enforced (times, counts)

Instrumentation
- Event: ai_diagnose_fixes_viewed { proposed }
- Event: ai_diagnose_fixes_selection { accepted, edited, dismissed }

A11y
- Controls labeled and reachable; warnings announced

Privacy
- No external calls here; local parameter edits only

---

## SCR-PREVIEW-FIXES — Review & Confirm (Apply Fixes)

Purpose
- Summarize accepted fixes as diffs; user confirms to apply. All changes are applied transactionally; if any failure occurs, none are applied.

Entry
- From SCR-PROPOSED-FIXES “Review & Confirm” (or Accept All/None paths)

Primary Data
- Diffs: Entity changes (reschedules, cadence updates), new micro-milestones, new routine(s), review-rule additions
- Conflicts and validations

Layout
- App Bar: “Apply Fixes”
- Summary: N changes across M entities
- Diff List: grouped by entity (Goal/Plan/Milestone/Routine)
  - Each diff line shows before → after
- Footer
  - “Apply Changes” (primary)
  - “Back”, “Cancel”

States
- Apply progress; on error show “Rolled back due to error. Try again.”

Validation/Constraints
- Block on invalid params; show conflicts with inline resolve tips

Instrumentation
- Event: ai_diagnose_preview_opened { changes_total }
- Event: ai_diagnose_applied { changes_total, types[] }

A11y
- Diffs presented as readable text; confirm button focusable; success announced

Privacy
- Saves AI Artifact with accepted flags and edited params; no free-text stored

Acceptance Mapping
- UC-8, FR-24..26 (edits and acceptance gating)

---

## SCR-USAGE — AI Usage & Cost

Purpose
- Provide transparency into AI usage (requests/tokens/cost estimates, if applicable), limits, and controls (disable AI, confirm prompts).

Entry
- Via overflow “AI Usage” in AI screens; from Settings link; or inline meter “Details”

Primary Data
- Today’s requests; soft/hard limits; cooldown info
- Cost/usage estimates (if surfaced)
- Controls: Disable AI (feature flag), Clear cached artifacts (local)

Layout
- App Bar: “AI Usage”
- Sections
  - Usage Today/This Month (counts)
  - Limits & Cooldowns (descriptions)
  - Controls: Disable AI (toggle), Clear Cache (button)
  - Links: Privacy policy, Safety guidelines
- Footer: Close

States
- If disabled: show banner and link to enable

Instrumentation
- Event: ai_usage_viewed
- Event: ai_usage_toggle_changed { enabled }

A11y
- Switch/toggle accessible and labeled

Privacy
- Reinforce no background calls without user intent; metadata-only telemetry

---

## Cross-Screen Handling

Offline
- Show persistent banner: “You’re offline — AI features are unavailable.” Provide manual alternatives (quick add milestones/steps, manual tips for diagnostics).

Timeouts & Errors
- Non-blocking loaders with Cancel; on timeout (~8s target), show Retry & Manual fallback.
- If rate-limited or quota exceeded, show clear cooldown/limit message; disable Generate/Run; manual paths remain.

Duplicates & Conflicts
- During Create/Apply, detect duplicates (suggest merge or continue); detect time conflicts (non-blocking warnings; allow override with confirm).

Storage Failures
- Show actionable modal with Retry; never persist partial changes; roll back atomically on fix application.

Privacy & Safety
- Clear disclaimers; no journal free-text or PII included without explicit user input.
- All AI outputs are suggestions; the user remains in control (FR-26).
- No background AI calls; all initiated by explicit user actions (NFR-12).

Accessibility
- Announce async events; ensure headings and roles; maintain focus order across panel transitions.

---

## Acceptance Criteria Mapping (Summary)

- AC-6: AI suggestions present ≥3 milestone candidates with rationale/assumptions; user can accept/edit; persistence only after confirm (SCR-AI-REFINE → SCR-AI-SUGGEST → SCR-PLAN-MAP → SCR-PREVIEW-CHANGES).
- UC-7: Full assist flow from request to persistence; usage/limits surfaced; offline/manual fallback.
- UC-8: Diagnostics show observations/hypotheses with confidence/assumptions; proposed fixes editable; apply transactionally only after acceptance (SCR-AI-DIAGNOSE → SCR-DIAGNOSE-REFINE → SCR-DIAGNOSE-RESULT → SCR-PROPOSED-FIXES → SCR-PREVIEW-FIXES).
- FR-26: All AI outputs require user acceptance; confidence/assumptions displayed; no auto-apply changes.
- NFR-12: Visible usage/cost controls; no background AI calls; rate limit/cooldown handling.

---

## Open Questions / TBD

- Should we expose a “Creativity vs. Practicality” slider in SCR-AI-REFINE for MVP or stick to fixed tone?
- Default date-offset heuristics for milestones on acceptance vs. unset by default?
- Degree of deduplication: warn only vs. offer merge flows?
- Portfolio diagnostics: Should we emphasize “most impactful single change” at the top?
- Undo/History: If supported, surface “Undo last applied fixes” on success toast.
