# UC-6 — View Progress Analytics and Achievements

ID: UC-6  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user)  
Level: User goal  
Frequency: Daily/Weekly

Traceability (SRS):
- Core Use Case: UC-6 (View progress analytics and earned achievements)
- Functional Requirements: FR-15..17 (Progress & Analytics, Export), FR-18..19 (Gamification & Rewards), FR-30..32 (Success Capture/Sharing — optional context)
- Acceptance Criteria: AC-5 (Progress view shows milestone % and routine streaks), FR-17 export snapshot

Related Screens:
- SCR-PROGRESS-HUB: Overall progress dashboard (goals, milestones, routines, trends)
- SCR-GOAL-DETAIL: Goal-specific progress view (plan/milestone metrics, upcoming items)
- SCR-TRENDS: Trends and reports (weekly completion rates, streak trends)
- SCR-ACHIEVEMENT-GALLERY: Earned achievements/badges list
- SCR-ACHIEVEMENT-DETAIL: Badge details and unlock criteria
- SCR-PROGRESS-EXPORT: Export snapshot (CSV/JSON/image/PDF summary)
- SCR-NOTIFICATION-CENTER: Overdue/upcoming items (contextual)
- SCR-OFFLINE: Offline banner/modal
- SCR-ERROR: Inline validation/recovery

Non-Functional Reminders:
- NFR-1 Usability: Locate key progress metrics in ≤ 2 taps
- NFR-2 Performance: <150ms UI interactions; analytics render <1s for ≤ 5k records
- NFR-4 Availability: Works offline; export and share degrade gracefully
- NFR-6 Privacy: Redaction by default on exports; opt-in per sharing
- NFR-7 Accessibility: WCAG AA contrast; keyboard/touch accessible charts
- NFR-9 Observability: Structured, metadata-only analytics logs (no PII)

---

## Goal/Benefit
Enable the user to quickly understand how they are progressing towards their goals, identify momentum or stall points, and reinforce motivation through achievements and streaks. Provide optional export to share or archive progress while protecting privacy by default.

Outcome:
- Clear visibility into milestone completion %, routine streaks, and weekly completion trends.
- View and celebrate earned achievements.
- Optionally export a progress snapshot (CSV/JSON or summary image/PDF) with redaction defaults.

---

## Preconditions
- User has at least one Goal with plans/milestones and/or at least one Routine.
- Local data available (offline-first). Exports to file system available.
- Achievements are computed on-the-fly or have been awarded during prior actions.

## Triggers
- From Dashboard/Home, user taps “Progress”.
- From Goal Detail, user taps “View full analytics”.
- From a success toast or badge notification, user taps “View Achievements”.

## Postconditions (Success)
- User sees up-to-date progress metrics (milestone %, routine streaks).
- User can browse achievements and read unlock criteria.
- Optional export file is created according to chosen format and redaction options.

## Postconditions (Failure/Abort)
- If user cancels export, no files are written.
- If any rendering or data error occurs, user remains in current view with non-blocking error messaging.

---

## Primary Flow A — Open Progress Hub (Overview)
1) Entry — SCR-PROGRESS-HUB
   - Sections:
     - At a Glance: 
       - Total active goals
       - Average milestone completion % (weighted/unweighted toggle)
       - Longest routine streak
       - Overdue count (milestones, routines)
     - Quick Cards per Goal:
       - Goal title
       - Milestone % complete
       - Active routine streaks linked to that goal
       - Upcoming milestone(s)
   - CTA:
     - “View Trends”
     - “View Achievements”
     - “Export Snapshot”

2) Interactions
   - Tap a Goal card → SCR-GOAL-DETAIL (anchor to analytics tab).
   - Toggle weighting (weighted by milestone count vs. unweighted average) updates “At a Glance”.

3) Result
   - AC-5 satisfied at overview level: milestone % and streaks are visible.

---

## Primary Flow B — Goal Analytics Drill-Down
1) Entry — SCR-GOAL-DETAIL (Analytics Tab)
   - Metrics:
     - Milestone progress: completed/total, % complete
     - Milestone list with statuses, optional success criteria indicators
     - Linked routine streaks and completion rate
     - Upcoming/overdue reminders (contextual)
   - Visualizations:
     - Progress ring/bar for milestones
     - Streak badge or counter
   - CTA:
     - “Edit Milestones”
     - “Open Trends”
     - “Export Goal Summary”

2) Interactions
   - Tap a milestone to see details (target date, success criteria, attachments).
   - Tap routine streak to open routine history.

3) Result
   - Immediate, goal-specific insight to help prioritize actions.

---

## Primary Flow C — Trends & Reports
1) Entry — SCR-TRENDS
   - Charts:
     - Weekly Completion Rate (last 4–8 weeks)
     - Streak Trends (per routine or top 3 routines)
   - Filters:
     - Time Window: 2w/4w/8w/custom
     - Scope: All Goals vs. Selected Goal
     - Routine Filter: All vs. selected routine(s)

2) Interactions
   - Tap a chart point to show tooltip (week’s completion %; streak highs/lows).
   - Toggle moving average overlay.
   - Compare “This Week vs. Last Week” quick pill.

3) Result
   - FR-16 satisfied; user identifies trends and momentum.

---

## Primary Flow D — Achievements Gallery
1) Entry — SCR-ACHIEVEMENT-GALLERY
   - Views:
     - Earned: Grid/list of badges with awarded dates
     - Locked: Grayed cards showing criteria (optional concealment until near-unlock)
   - Sorting/Filters:
     - Recent, By Type (streaks, milestones, consistency), By Goal

2) Details — SCR-ACHIEVEMENT-DETAIL
   - Shows:
     - Badge name, description
     - Unlock criteria (e.g., “First Milestone”, “Streak 7”)
     - Awarded_at (if earned)
     - Linked entity (routine/goal) if applicable
   - CTA:
     - “View Related Goal/Routine”
     - “Export Progress Snapshot” (optional share context)

3) Result
   - FR-18..19 satisfied: user can view earned achievements and understand how to earn more.

---

## Primary Flow E — Export Progress Snapshot
1) Entry — SCR-PROGRESS-EXPORT
   - Options:
     - Scope: All Data vs. Selected Goal
     - Format: CSV, JSON, Summary Image/PDF (MVP at least CSV/JSON)
     - Redaction: ON by default (hide photos, redact notes; user can opt-in to include)
     - Time Range: All vs. last 4/8/12 weeks (for summary export)
   - Privacy Banner:
     - “Your export is private by default. Carefully review before sharing.”

2) Export
   - User taps “Export”.
   - System generates file; shows path or “Open file” button.

3) Result
   - FR-17 satisfied; AC-5 remains in effect since user arrived via progress views.

---

## Alternate Flows
A1) Empty State (No Data Yet)
- Show coaching cards:
  - “Create your first goal or routine to begin tracking progress.”
  - “Apply a template to get started faster.”
- Provide links to UC-1 and UC-3 flows.

A2) Lightweight Compare
- Toggle “Compare to last period” on overview cards to show +/− deltas.

A3) Print-Friendly / Readable Summary
- When selecting Summary Image/PDF, show preview; allow branding toggle (on/off).

A4) Per-Goal Export
- From SCR-GOAL-DETAIL, “Export Goal Summary” pre-fills scope filters.

A5) Share Later
- Save an export preset; re-run quickly next week with same settings.

---

## Exception Flows
E1) Offline Mode
- All progress and achievements views work.
- Export works to local file system; cloud share links disabled.

E2) Large Dataset / Performance
- If rendering exceeds NFR budget:
  - Show skeleton/loading states.
  - Defer heavy charts; progressively load or paginate metrics.

E3) Storage/Export Error
- Error banner: “Export failed. Try again.”
- Provide retry; optionally show diagnostics link with redacted logs.

E4) Privacy Risk Warning
- If user opts-in to include photos/notes:
  - Confirmation modal: “You’re including personal content. Proceed?”

E5) Time Zone / Clock Changes
- Show light banner: “Metrics adjusted due to time change.”
- Ensure consistent weekly rollups and streak calculations.

---

## Data Created/Updated
- No core entities modified by viewing analytics.
- Export Artifact (optional):
  - { id, created_at, scope, format, redaction_flags, file_path }
- Derived/Cache (optional internal):
  - Progress aggregates (per goal/plan), trend buffers
- View Preferences (local):
  - Filters, weighting toggles, last selected scope/time window

Note: Achievements are computed/awarded by other flows; gallery reads existing awards.

---

## Business Rules & Constraints
- BR-1: Milestone % = completed_milestones / total_milestones (per plan/goal). Clarify how “Archived/Deferred” milestones factor (MVP: exclude from denominator).
- BR-2: Routine streak increments only per cadence windows (see UC-3); streak display reflects current value.
- BR-3: Weekly completion rate = completed_items / planned_items for the week; define clearly whether “planned” includes carried-over items (MVP: count by due-in-week).
- BR-4: Exports default to redaction ON (photos hidden, notes redacted). User must explicitly opt-in to include sensitive data (FR-31..32).
- BR-5: Achievements must be idempotent on re-check (no duplicates).
- BR-6: Weighted averages count per-goal milestone totals; unweighted averages treat each goal equally.

---

## Analytics & Telemetry (Privacy-Respecting)
- Event: progress_hub_viewed { goals_visible, has_overdue }
- Event: progress_goal_viewed { goal_id_hash, milestone_pct, streak_count }
- Event: trends_viewed { window, scope }
- Event: achievements_opened { earned_count, locked_visible }
- Event: export_initiated { scope, format, redaction_on: boolean }
- Event: export_completed { duration_ms, file_size_kb }

Note: Hash IDs; do not include free-text content or PII in telemetry.

---

## Acceptance Criteria (Derived)
- AC-6.1 (from AC-5): Progress view shows milestone completion percentage and routine streaks for at least one goal.
- AC-6.2: User can view weekly completion rate and streak trends in a trends view (FR-16).
- AC-6.3: User can open Achievements and see at least one earned badge when criteria met (FR-18..19).
- AC-6.4: User can export a progress snapshot (CSV/JSON or summary image/PDF) with redaction ON by default (FR-17; FR-31..32 privacy).
- AC-6.5: All analytics views render within target performance budgets for typical data sizes (NFR-2).

---

## Open Questions / TBD
- Should archived/deferred milestones be excluded from analytics by default?
- Do we include routine “heatmaps” in MVP or defer to a later release?
- Preferred default for averaging (weighted vs. unweighted) on overview?
- What minimal charting library/accessibility strategy will we use to ensure WCAG AA while offline?
- Should exports include mini-charts (sparklines) in the summary PDF/image for MVP?

---

## UI State Map
- ST-UC6-01: SCR-PROGRESS-HUB (overview)
- ST-UC6-02: SCR-GOAL-DETAIL (analytics tab)
- ST-UC6-03: SCR-TRENDS (weekly completion and streak trends)
- ST-UC6-04: SCR-ACHIEVEMENT-GALLERY (earned/locked)
- ST-UC6-05: SCR-ACHIEVEMENT-DETAIL
- ST-UC6-06: SCR-PROGRESS-EXPORT (export options)
- ST-UC6-ERR: SCR-ERROR (validation), SCR-OFFLINE (banner)

---

## Wireflow (Textual)
- From Dashboard/Home → “Progress” → SCR-PROGRESS-HUB
  - Tap a Goal card → SCR-GOAL-DETAIL (analytics)
    - Tap “View Trends” → SCR-TRENDS
    - Back → SCR-GOAL-DETAIL
  - Tap “View Achievements” → SCR-ACHIEVEMENT-GALLERY → (select) SCR-ACHIEVEMENT-DETAIL → Back
  - Tap “Export Snapshot” → SCR-PROGRESS-EXPORT → choose options → Export → Success toast → Return
