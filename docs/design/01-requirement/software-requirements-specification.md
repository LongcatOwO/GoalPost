# Software Requirements Specification (SRS) — GoalPost

Document Status: Draft v0.1  
Source: Derived from `docs/design/01-requirement/overview.md`  
Owner: Phuwasate Lutchanont
Audience: Engineering, Product (self)

## 0) Summary (For Quick Reading)

- Problem we solve (top-level):
  - Turn personal goals into concrete, trackable plans.
  - Maintain consistent habits with routines, schedules, and reminders.
  - Understand why goals stall using AI diagnostics and reflective journaling.
  - Stay motivated via progress views and gamified rewards.
  - Start fast using domain templates (fitness, finance, education, diet, beauty, health).
  - Capture and optionally share achievements to reinforce commitment and social support.

- Primary users: Individuals focused on self-improvement (students, professionals, hobbyists, general wellness seekers).
- Secondary users: power users who want structured analytics and AI coaching.

- MVP must-have features (MoSCoW “Must”):
  - Create/edit/archive goals, plans, and milestones.
  - Track routines/habits and daily activities.
  - Scheduling and reminders (in-app notifications; OS/push optional for later).
  - Morning/night review flows.
  - Progress tracking (completion rates, streaks, milestone status).
  - Journaling of failures/insights.
  - AI assistance: suggest steps; analyze failures and blockers (via external LLM API).
  - Basic templates (at least 3 categories).
  - Capture success evidence (notes/photos) and optional basic sharing (export/share link or local export).

- Key quality needs:
  - Simple, fast, low-friction UI; clear daily workflow.
  - Privacy-first defaults; user controls sharing.
  - Reliable local persistence with backup/export.
  - Clear audit/logs around AI suggestions and decision rationale.


- Architectural guardrails:

  - Start simple: single-user app, minimal infra, local-first data with export/backup.
  
  - Target platforms: Mobile (iOS/Android) and Web; offline-first on mobile; responsive/PWA web experience where possible.
  
  - AI access via an app-managed thin proxy (no user-visible API keys). Software-side controls: per-user quotas and rate limits, daily/monthly cost ceilings, max tokens per call, debounce/batching, exponential backoff with circuit breaker, feature flag to disable/pause AI, local caching of AI artifacts, graceful offline fallback, and in-app usage metering with no background AI calls without user intent.
  
  - Extensible data model for templates and future modules.



- Out-of-scope (MVP):
  - Team/multi-user collaboration, real-time sync across accounts.
  - Marketplace of public templates.
  - Advanced social feeds; comments/likes.
  - Wearables and deep third-party integrations (calendar/health) — post-MVP.

---

## 1) Purpose

Define functional and non-functional requirements for GoalPost, a personal roadmap manager that helps individuals set goals, build plans, track routines, understand blockers (with AI assistance), and stay motivated through progress visibility, rewards, and sharing.

## 2) Scope

- In-scope: goal planning, milestones, routines, scheduling, reminders, reviews, progress analytics, journaling, AI diagnostics and guidance, domain templates/modules, capture/share achievements.
- Platforms: Mobile (iOS/Android) and Web; offline-first on mobile; responsive/PWA web experience where possible.
- Single-user mode for MVP; no server-side multi-tenant requirements initially.

## 3) Users and Personas

- Self-improver: wants clarity, structure, and steady progress.
- Busy professional: time-constrained; needs lightweight scheduling/reminders and concise daily reviews.
- Analyst/optimizer: wants feedback loops, diagnostics, and data-backed insights.
- Wellness seeker: benefits from curated templates (fitness, health, diet) and progress reinforcement.


Accessibility note: users may have limited time; prioritize mobile-first flows on iOS/Android and responsive web UX; clear, concise UI with strong defaults.


## 4) Problem Statement

- Routine inconsistency and weak habit formation.
- Lack of visibility into progress → low motivation.
- Poor time scheduling and missed reminders.
- Unclear why plans fail; no systematic reflection.
- High friction starting new goals in complex domains without expert structure.
- Weak reinforcement: successes not captured or celebrated; little social accountability.

## 5) Definitions and Glossary

- Goal: a desired outcome with time horizon.
- Plan: breakdown of a goal into tasks/strategies.
- Milestone: measurable checkpoint within a plan/goal.
- Activity/Task: actionable unit to progress a milestone/goal.
- Routine/Habit: recurring activity with cadence.
- Reminder/Notification: prompt associated with schedule or routine.
- Review (Morning/Night): quick plan/reflect cycle for a day.
- Progress Report: aggregated status and metrics (completion, streaks).
- Gamification/Reward: badges, points, streaks, achievements.
- Journal: reflective entries, especially on failures and learnings.
- AI Assistant: capability to suggest steps and diagnose blockers.
- Template/Module: pre-defined plan structures for a domain (e.g., fitness).
- Success Capture: notes/photos documenting achievements.
- Share: export/share artifacts with others (privacy-controlled).

## 6) Assumptions, Constraints, Dependencies

- Single user per installation/account for MVP.
- AI assistance uses an external LLM API accessed via an app-managed proxy; no user-supplied API key required; API keys are stored securely by the app; offline suggestions limited or none.
- Notifications: guaranteed in-app; OS notifications and push are optional later.
- Data: local-first storage with export; cloud sync is post-MVP.
- Legal/Compliance: privacy-first; user controls what is shared; no sensitive health or financial advice beyond templated guidance (disclaimers required).
- Performance: snappy interactions on commodity hardware.

## 7) Out of Scope

- Multi-user collaboration, shared workspaces.
- Real-time social features (likes/comments/follows).
- Deep third-party integrations (Google Calendar, Apple Health, banking APIs) in MVP.
- AI autonomy (no auto-execution of changes without user confirmations).

---

## 8) Functional Requirements (FR)

### 8.1 Goal, Plan, and Milestone Management
- FR-1 Create, read, update, archive/delete goals.
- FR-2 Associate each goal with one or more plans.
- FR-3 Define milestones under plans with target dates and success criteria.
- FR-4 Tag goals/plans/milestones (e.g., domain, priority).
- FR-5 View a goal detail page showing plan, milestones, progress, upcoming tasks.

Acceptance (MVP):
- Create one goal with at least one plan and two milestones.
- Edit milestone target date and mark as completed.

### 8.2 Routine and Habit Tracking
- FR-6 Define routines with cadence (daily/weekly/custom).
- FR-7 Track routine completions; maintain streaks.
- FR-8 Suggest daily routines based on configured cadences.

Acceptance:
- Mark a routine complete; streak increments accordingly.

### 8.3 Scheduling and Reminders
- FR-9 Schedule activities and milestones with due dates/times.
- FR-10 In-app reminders list with snooze/dismiss.
- FR-11 Notification center view for overdue and upcoming items.

Acceptance:
- A scheduled task due within the next hour appears in reminders.

### 8.4 Morning/Night Reviews
- FR-12 Morning review: show today’s priorities (goals, tasks, routines).
- FR-13 Night review: capture completions, blockers, and reflections; list missed items.
- FR-14 Persist summaries for historical review.

Acceptance:
- Completing a night review creates a journal entry linked to the day.

### 8.5 Progress Tracking and Analytics
- FR-15 Progress views: goal status, milestone completion %, routine streaks.
- FR-16 Trend reports: weekly completion rate and streak trends.
- FR-17 Export progress snapshot (CSV/JSON or image/PDF summary).

Acceptance:
- User can see milestone completion percentage and a streak count.

### 8.6 Gamification and Rewards
- FR-18 Achievements for streak thresholds, milestone completions, consistency.
- FR-19 Reward notifications in-app; badge gallery.

Acceptance:
- Completing first milestone awards a “First Milestone” badge.

### 8.7 Journaling and Failure Reflection
- FR-20 Create journal entries, especially on missed goals or tasks.
- FR-21 Categorize reasons for failure (customizable taxonomy).
- FR-22 Link journal entries to goals/milestones/routines.

Acceptance:
- Missed task can be logged with a reason and linked to the task’s goal.

### 8.8 AI Assistance (Guidance and Diagnostics)
- FR-23 AI suggests steps/milestones given a goal statement.
- FR-24 AI analyzes failure patterns and proposes corrective actions.
- FR-25 AI “why system not working” diagnostic: synthesize signals from streaks, delays, journals.
- FR-26 All AI outputs include confidence/assumptions and allow user acceptance/edits before saving.

Acceptance:
- User submits a goal; AI returns 3 suggested milestones and a short rationale; user can accept/edit.

Dependencies:
- External LLM API configured via user-supplied key; handle absence gracefully with manual-only flows.

### 8.9 Templates and Domain Modules
- FR-27 Curated templates: Fitness, Financial, Education, Diet, Beauty, Health (at least 3 in MVP).
- FR-28 Apply a template to a new goal (pre-populates plans/routines/milestones).
- FR-29 Customize templates post-apply without mutating the original template library.

Acceptance:
- Creating a “Fitness” goal from template pre-fills 1 plan, 2 routines, 3 milestones.

### 8.10 Success Capture and Sharing
- FR-30 Attach notes and photos to milestones/completions.
- FR-31 Generate a shareable artifact (export file or static page) with redaction options.
- FR-32 Privacy controls: default private; explicit opt-in per export/share.

Acceptance:
- Export a progress summary with selected photos redacted by default unless included.

### 8.11 Settings, Privacy, and Data Management
- FR-33 Configure notification preferences, time zone, review times.
- FR-34 Manage AI settings and API key storage.
- FR-35 Data export/import (JSON/CSV) and local backup.
- FR-36 Data deletion: delete goals, related data, and AI artifacts.

Acceptance:
- Export full dataset to a single file and re-import into a fresh instance.

---

## 9) Non-Functional Requirements (NFR)

- NFR-1 Usability: Morning/night reviews complete in ≤ 2 minutes with ≤ 5 clicks/taps each.
- NFR-2 Performance: UI actions respond in < 150ms; analytics views render in < 1s for ≤ 5k records.
- NFR-3 Reliability: No data loss across restarts; autosave edits; export succeeds reliably.
- NFR-4 Availability (local-first): app functions offline except AI calls and external sharing; degrade gracefully.
- NFR-5 Security: Local storage encrypted at rest where platform supports; API keys stored securely; no keys in logs.
- NFR-6 Privacy: Opt-in sharing; redact personal data by default on exports; clear consent prompts.
- NFR-7 Accessibility: Contrasts meet WCAG AA; keyboard navigation; labels for assistive tech.
- NFR-8 Maintainability: Modular architecture; feature modules not tightly coupled; 80% of core logic covered by unit tests.
- NFR-9 Observability: Structured logs for AI prompts/responses (metadata-only, redact PII), errors, and critical flows.
- NFR-10 Portability: Minimal platform-specific dependencies; data exports portable via open formats (JSON/CSV).
- NFR-11 Localization: Text externalized for future i18n; MVP in English.
- NFR-12 Cost: AI usage controllable via settings; show token/usage summary; no background AI calls without user intent.
- NFR-13 Ethics: AI provides suggestions, not prescriptions; show disclaimers for fitness/financial/health templates.

---

## 10) Data Model (High-Level)

Entities (minimum):
- User Settings
- Goal (id, title, description, domain, priority, status, dates, tags)
- Plan (id, goal_id, title, description)
- Milestone (id, plan_id, title, description, target_date, status, success_criteria, attachments)
- Routine (id, goal_id or plan_id, title, cadence, schedule, streak, last_completed)
- Task/Activity (optional for MVP; can be represented as milestone steps)
- Reminder (id, entity_ref, due_at, status)
- Journal Entry (id, date, text, reason_tags, linked_entities)
- Achievement (id, type, awarded_at, entity_ref)
- Template (id, domain, prefilled structures)
- Attachment (photo/note refs)
- AI Artifact (prompt metadata, suggestion text, rationale, links)

Relationships:
- Goal 1—* Plan; Plan 1—* Milestone; Goal/Plan 1—* Routine; Entities *—* Journal via links.

---

## 11) External Interfaces


- AI Provider: LLM API (HTTPS) via an app-managed thin proxy (no user-visible API keys); rate limits and cost controls applied.


- Notifications: In-app for MVP; Mobile push (APNs/FCM) and Web Push post-MVP, subject to platform permissions.

- File System: For photo attachments and export/import operations.

---

## 12) Core Use Cases

- UC-1 Create a new goal from scratch or template.
- UC-2 Break down a goal into milestones; set targets.
- UC-3 Define routines and complete daily check-ins.
- UC-4 Morning review: confirm priorities, planned tasks, and routines.
- UC-5 Night review: record completions, blockers, and journal reflections.
- UC-6 View progress analytics and earned achievements.
- UC-7 Ask AI to suggest steps; accept/edit outputs.
- UC-8 Ask AI why progress is stalled; review diagnostic and proposed fixes.
- UC-9 Capture photos/notes for completed milestones and export a shareable summary.
- UC-10 Backup/export data and restore/import on a fresh install.

---

## 13) Acceptance Criteria Summary (MVP)

- AC-1 Creating a goal from a template creates pre-defined plans/milestones/routines.
- AC-2 User can complete a routine and see streak increment immediately.
- AC-3 Morning review displays all due/soon tasks and routines for the day.
- AC-4 Night review persists a journal entry with missed items and reasons.
- AC-5 Progress view shows milestone % and routine streaks.
- AC-6 AI suggests at least 3 milestone candidates with rationale; user can accept/edit and save.
- AC-7 Export produces a JSON file containing goals, plans, milestones, routines, journals, achievements.
- AC-8 Privacy: exported summary hides photos unless explicitly included.

---

## 14) Risks and Mitigations

- R-1 AI dependency/costs → Mitigate with manual workflows and visible usage controls.
- R-2 Privacy concerns with sharing → Redaction by default; explicit consent gates.
- R-3 Over-complexity for solo dev → Prioritize MVP scope; modular design; template-driven UI.
- R-4 Data loss → Autosave and regular local backups; import/export tested.
- R-5 Motivation drop if feedback unclear → Clear progress visuals, early achievements, daily reviews streamlined.

---

## 15) Roadmap and MoSCoW Prioritization

- Must (MVP): FR-1..5, FR-6..8, FR-9..11, FR-12..14, FR-15..17, FR-20..22, FR-23..26 (baseline), FR-27..29 (≥3 templates), FR-30..32 (basic sharing/export), FR-33..36; NFR-1..6, 8, 10, 12.
- Should (Post-MVP v1.1): OS notifications, richer analytics, more templates, better badge system, image annotations.
- Could: Calendar integrations, wearable/health data, financial APIs, public template marketplace.
- Won’t (now): Multi-user collaboration, social feeds with engagement mechanics.

Phasing (suggested):
- Phase 1: Goals/plans/milestones, routines, reviews, progress, export/import.
- Phase 2: AI suggestions/diagnostics (behind a flag until stable).
- Phase 3: Templates library expansion, achievements, basic sharing artifacts.
- Phase 4: Notifications beyond in-app, polishing analytics and UX.

---

## 16) Traceability (Overview → Requirements)

- Routine Activity → FR-6..8; Progression Tracking → FR-15..17; Gamification → FR-18..19; Time Scheduling → FR-9..11; Reminder → FR-9..11; Morning/Night Review → FR-12..14; Find Out Why Not Working → FR-25; Journal Failures → FR-20..22; AI Understand Failures → FR-24..25; Templates/Modules → FR-27..29; Capture/Share Success → FR-30..32.

---

## 17) Open Questions / TBD


- Mobile (iOS/Android) and Web confirmed; storage specifics per platform (e.g., on-device DB, IndexedDB, secure keychain), and cloud sync roadmap.


- Finalize notification channels: in-app (MVP), and timeline for Mobile push (APNs/FCM) and Web Push; permission UX.

- Template details and curation depth for each domain.
- AI provider selection, cost ceilings, and prompt templates.
- Visual identity and branding for achievements and exports.

End of SRS.
