# User Flow: Create a New Goal (with Template Selection)

Summary
-------
This user flow describes the primary and alternative paths for creating a new goal in GoalPost, with a focus on selecting and using a template. It includes screens, decision points, edge cases, data needed, and acceptance criteria.

Actors
------
- User: a signed-in individual using GoalPost.
- System: GoalPost backend, UI, and AI assistant.

Preconditions
-------------
- The user has an active account and is signed in.
- The user is on the app's dashboard, goals list, or a context where `Create Goal` is available.
- Template library contains at least one template (system or user-created) — if empty, the UI must show a clear `No templates` state.

Primary (Happy) Flow — Create a New Goal using a Template
--------------------------------------------------------
1. Entry
   - Trigger: User taps `Create Goal` CTA on the dashboard or goals list.
   - Screen: `Create Goal Start` modal/screen with two primary options:
     - `Start from Template`
     - `Create Custom Goal`

2. Choose Template
   - User selects `Start from Template`.
   - Screen: `Template Gallery` (list/grid) showing templates grouped by category (`Fitness`, `Financial`, `Education`, etc.).
   - UI elements:
     - Search bar
     - Category filters
     - Card for each template with preview (name, short description, difficulty, estimated duration)
     - `Preview` and `Select` actions on each card

3. Preview Template (optional)
   - User taps `Preview` on a template.
   - Screen: `Template Preview` shows:
     - Full description
     - Typical milestones and tasks
     - Estimated timeline
     - Example schedule
     - `Use This Template` and `Back` buttons

4. Select Template
   - User taps `Select` (or `Use This Template` from preview).
   - System loads the template into a `Create Goal` form.

5. Customize Goal Details
   - Screen: `Create Goal` form pre-filled with template data:
     - Title (editable)
     - Description (editable)
     - Target date / Duration
     - Priority / Difficulty
     - Milestones (editable list)
     - Habit / daily tasks suggestions
     - Visibility (private / shareable)
     - Notifications and reminders
   - User customizes fields as desired.
   - UI provides inline help and recommended defaults.

6. AI Enhancement (optional but recommended)
   - UI offers `Let AI optimize this plan` toggle/button.
   - If enabled:
     - System analyzes the template + user inputs and suggests:
       - Adjusted milestones
       - Smarter scheduling (based on user's timezone, typical routine)
       - Estimated weekly effort
       - Possible bottlenecks & tips
     - UI displays changes with `Apply` or `Dismiss` options.

7. Confirm & Create
   - User reviews final plan and taps `Create Goal`.
   - System validates the input (required fields, sensible dates).
   - On success:
     - Goal is created
     - System navigates to `Goal Overview` (new goal page)
     - Shows `Goal Created` toast and recommended first actions (e.g., `Start First Milestone`, `Add Habit`, `Share`)

8. Post-creation suggestions
   - System suggests:
     - Add to daily routine / calendar
     - Invite accountability buddy
     - Enable milestone reminders
     - Capture a success photo template for later use

Alternative Flows
-----------------
A1 — Start from Custom (no template)
  - From step 1, user selects `Create Custom Goal`.
  - System presents blank `Create Goal` form (same as step 5 but empty).
  - Continue with steps 6–8.

A2 — Template Not Found / No Templates
  - If the template library is empty or the search yields no results:
    - Show `No templates found` view with:
      - `Create Custom Goal` CTA
      - `Request Template` feedback link
      - Option to `Import Template` (for power users)
  - User can switch to custom flow.

A3 — Cancel or Exit During Creation
  - If user cancels:
    - If there is unsaved data, show `Save as Draft` / `Discard` / `Continue Editing` options.
    - If `Save as Draft`: create a draft accessible in `My Drafts`.
    - If `Discard`: delete temporary data and return to prior screen.

A4 — AI Suggestions Rejected
  - User reviews AI suggestions and taps `Dismiss` or selectively rejects some suggestions.
  - System applies only the accepted suggestions and proceeds to step 7.

Error Flows
-----------
E1 — Validation Error
  - Missing required fields or invalid date:
    - Show inline validation messages and block creation until fixed.

E2 — Backend Failure
  - On submit, network or server error:
    - Show `Something went wrong` with `Retry` and `Save as Draft` options.
    - Log error and surface friendly message.

E3 — Conflicting Schedule
  - AI detects conflicts with existing goals/events:
    - Show conflict warning with recommended resolution options (`Reschedule`, `Shorten milestones`, `Ignore`).

UI Wireframe / Screen Map (high-level)
-------------------------------------
- Dashboard
  - CTA: `Create Goal` -> `Create Goal Start` modal
    - Option: `Start from Template` -> `Template Gallery` -> `Template Preview` -> `Create Goal (prefilled)`
    - Option: `Create Custom Goal` -> `Create Goal (blank)`
- Create Goal (form)
  - Sections: Basic Info | Timeline | Milestones | Routine / Habits | Notifications | AI Optimize
  - Actions: `Create` | `Save as Draft` | `Cancel`

Data Model Notes
----------------
- Goal
  - id, user_id, title, description, category, template_id (nullable), created_at, updated_at, due_date, priority, visibility
- Milestone[]
  - id, goal_id, title, description, order, due_date, status
- Habit/Task[]
  - id, goal_id, recurrence, time_of_day, reminder_settings
- Drafts
  - store incomplete `Goal` objects with `is_draft` flag
- AI Plan Suggestions
  - store suggestion objects so user can accept/reject granular changes

Acceptance Criteria
-------------------
- User can start creating a goal from templates or from blank.
- Template gallery supports search, filter, preview, and select.
- Selecting a template pre-fills the goal creation form.
- User can edit pre-filled fields before creating the goal.
- AI optimization is optional and can be accepted or rejected.
- On successful create, the user is navigated to the new goal overview and shown confirmation.
- Unsaved changes prompt the user to save as draft or discard.
- Errors show clear, actionable messages and retry/save options.

Edge Cases & Considerations
---------------------------
- Large templates (many milestones): UI should collapse/expand milestone groups to avoid overwhelming the user.
- Accessibility: All flows must be navigable by keyboard and screen readers; `Preview` should include text summaries.
- Internationalization: Dates, time, and text must respect locale.
- Rate limits for AI: Throttle AI suggestions to avoid excessive calls.
- Privacy: Templates copied from user-shared templates must respect visibility rules and not expose private data.

Notes for Implementation
------------------------
- Use a single `CreateGoal` component that supports `mode: { template | custom | draft }`.
- Keep template preview data lightweight; fetch full template details only on preview/select.
- Persist drafts server-side and locally (for offline support) with conflict resolution on sync.
- Track analytics events: `create_goal_started`, `template_selected`, `ai_suggestions_applied`, `goal_created`.

Related Design Files
--------------------
- `docs/design/01-draft/overview.md` — project goals and templates (source for template categories and problems the system solves).
- Place new user flows in `docs/design/02-userflow/` for consistency.

Revision History
----------------
- 2026-02-12 — Initial flow draft (created).
