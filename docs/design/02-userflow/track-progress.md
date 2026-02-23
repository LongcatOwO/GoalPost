# User Flow: Track Progress & Daily Review

Summary
-------
This user flow defines how users view progress on their goals, perform a daily review, and take follow-up actions (adjust plans, log reflections, skip or re-schedule tasks). It includes the happy path, alternative flows, edge cases, data needs, acceptance criteria, and implementation notes.

Actors
------
- User: signed-in individual who owns or follows goals.
- System: GoalPost backend, UI, analytics, and AI assistant.
- Notifications/Calendar: external systems for reminders and calendar sync.

Preconditions
-------------
- User has at least one active goal or draft with milestones/tasks.
- User is signed in and has the app open or receives a daily review notification.
- Progress data is being recorded (task completions, habit logs, milestone updates).

Entry Points
------------
- Dashboard -> `Today` or `Progress` pane.
- Goal detail -> `Progress` tab.
- Daily Review push/notification or scheduled in-app reminder.
- Calendar integration (user taps quick link from calendar event).

Primary (Happy) Flow — Daily Review & Track Progress
---------------------------------------------------
1. Entry / Summary View
   - Trigger: User opens the app or receives a `Daily Review` notification and taps it.
   - Screen: `Daily Review` modal or `Today` view summarizing:
     - Today's scheduled habits/tasks
     - Progress summary (goals with percent complete, streaks)
     - Overdue items and upcoming milestones
     - Quick actions (Mark done, Snooze, Add journal entry)
   - Data needed: user's timezone, today's scheduled tasks, completion status, goal progress metrics.

2. Inspect Progress for a Goal
   - User taps a goal from the summary to open `Goal Overview -> Progress`.
   - Screen shows:
     - Overall completion percentage
     - Recent activity timeline (last 7/30 days)
     - Milestones with statuses (not started / in progress / completed)
     - Habit streaks and habit completion rates
     - Visual charts (progress over time, weekly effort)
   - UI elements: Drill-down on milestone, activity feed, filters for timeframe.

3. Log or Confirm Today's Work
   - User marks today's tasks/habits as complete or logs partial completion.
   - System updates goal progress and streak status in real-time.
   - If the user completes a milestone, the system surfaces a `Milestone Achieved` confirmation with options to celebrate/share.

4. Reflect (Journal)
   - Screen: `Daily Reflection` prompt with short, guided questions:
     - What went well today?
     - What blocked progress?
     - One step to take tomorrow.
   - User writes or uses AI-assisted suggestions to populate the journal entry.
   - System stores the journal entry linked to the day and optionally to the goal(s).

5. AI Insights & Recommendations (optional)
   - User can tap `Analyze` to let AI summarize recent progress and suggest adjustments:
     - Suggested milestone rescheduling
     - Habit frequency changes
     - Tips for improving consistency
   - AI shows rationale and estimated impact; user can accept all, some, or none.

6. Take Action
   - From the review, user can:
     - Reschedule a milestone or task
     - Convert a missed task into a habit
     - Add follow-up sub-tasks
     - Share progress (social or accountability buddy)
   - System persists changes and recalculates progress projections.

7. Exit
   - User finishes and exits the daily review. System logs analytics events:
     - `daily_review_opened`, `daily_reflection_saved`, `task_marked_complete`, `ai_insights_applied`.

Alternative Flows
-----------------
A1 — Quick Check-In (Minimal Interaction)
  - User quickly taps `Mark Today's Tasks Done` from notification without opening full review.
  - System records completion and shows a brief toast with next steps.

A2 — Missed Day / Catch-up Flow
  - If user missed several days, the `Daily Review` shows a catch-up summary and a `Catch Up` workflow to:
    - Bulk mark skipped days
    - Re-schedule missed milestones
    - Create a `Recovery Plan` suggested by AI to regain momentum

A3 — Multi-Goal Consolidated Review
  - When multiple active goals have activity, the `Daily Review` aggregates changes and allows switching focus between goals.
  - User can perform actions per-goal or apply a bulk action (e.g., `Snooze all low priority tasks`).

A4 — No Activity / Empty State
  - If there's no activity or configured tasks:
    - Show `No progress yet` with CTAs: `Add first habit`, `Import template`, `Set daily routine`.
    - Offer onboarding tips and quick goal linking.

Error Flows
-----------
E1 — Sync Failure (offline or network)
  - If the system can't sync updates:
    - Save changes locally and show `Pending sync` status.
    - Retry sync automatically and present `Retry now` ability.
    - If conflict occurs, present `Resolve conflict` dialog with options: `Keep local`, `Use server`, `Merge`.

E2 — Analytics / Chart Load Error
  - If progress charts fail to render:
    - Show simplified textual summary and `Try again` action.
    - Log error for debugging.

E3 — AI Unavailable or Rate-limited
  - If AI endpoint is unavailable:
    - Hide `Analyze` or show `AI unavailable` message with expected retry time.
    - Allow manual insights entry as fallback.

UI Wireframes / Screen Map
--------------------------
- Dashboard
  - Today / Progress summary card
    - CTA: `Open Daily Review` -> `Daily Review` modal
- Daily Review Screen
  - Header: Date, quick-stats (completed / planned / streak)
  - Sections:
    - Scheduled tasks/habits (with toggle complete)
    - Goals summary (list)
    - Overdue & upcoming milestones
    - Reflection card (open to expand)
    - AI Insights card (optional)
  - Footer actions: `Save Reflection`, `Apply Suggestions`, `Share`, `Close`
- Goal Overview -> Progress Tab
  - Progress % | Timeline charts | Milestones list | Activity feed
  - Actions: `Edit plan`, `Reschedule`, `Add note`

Data Model Notes
----------------
- ProgressEntry
  - id, user_id, goal_id (nullable), date, type (habit|task|milestone), status, metadata
- JournalEntry
  - id, user_id, goal_ids[], date, title, body, ai_summary_id (nullable)
- GoalProgressSnapshot
  - goal_id, date, percent_complete, streaks, estimated_effort
- SyncQueue
  - local_id, payload, status, last_attempt_at
- AISuggestion
  - id, user_id, goal_id, suggestions[], accepted[], created_at

Acceptance Criteria
-------------------
- The `Daily Review` shows the user's scheduled items and accurate completion state.
- Users can mark items complete and the system updates goal progress immediately.
- Users can write a reflection that gets stored and associated with the day and relevant goals.
- AI suggestions are optional; users can accept, partially accept, or dismiss them.
- Offline changes are saved locally and synced when online, with conflict resolution options.
- Overdue items and upcoming milestones are clearly surfaced and actionable.
- Analytics events are emitted for key interactions.

Edge Cases & Considerations
---------------------------
- Timezone changes: daily review boundaries must respect current timezone and historical logs should remain tied to the timezone they were recorded in.
- Large activity histories: pagination or condensed summaries to prevent long loads.
- Privacy: journal entries default to private; sharing must be explicit.
- Accessibility: keyboard navigation for marking tasks and entering reflections; ensure screen reader compatibility.
- Multiple devices: last-writer-wins conflicts should prompt user instead of silently overwriting significant reflections or plan edits.
- Habit tracking variance: support partial completions (e.g., `50% done`) and map them to progress heuristics.
- Notifications noise: allow users to tune daily review frequency, time, and channels (push, email).

Implementation Notes
--------------------
- Use reactive state to show immediate feedback when tasks are marked complete; queue server updates for durability.
- Keep charts client-side-rendered using lightweight libraries to avoid heavy server computation.
- Store daily reflections in a dedicated journal store and index by date for fast retrieval.
- AI suggestions should be computed on demand and cached for a short TTL; track usage to manage rate limits.
- Provide a `compact` daily review layout for quick check-ins and an `expanded` layout for deep dives.
- Track analytics keys: `daily_review_open`, `task_toggled`, `reflection_saved`, `ai_insight_requested`, `suggestion_applied`.

Related Design Files
--------------------
- `docs/design/01-draft/overview.md` — project goals, templates, and high-level problems to solve.
- Place other user flows in `docs/design/02-userflow/` for consistent documentation.

Revision History
----------------
- 2026-02-12 — Initial draft for tracking progress and daily review flow.
