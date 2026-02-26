# Design Appendix — Glossary of Common Terms and IDs

Purpose
- Quick reference for keywords, acronyms, and ID conventions used across the design docs (requirements, user flows, wireframes, system, and UI specs).

---

## ID Conventions

- UC-# — Use Case
  - End-to-end user journey (e.g., UC-1 Create Goal, UC-4 Morning Review).
- SCR-XXXXX — Screen
  - Individual screen specification (e.g., SCR-GOAL-DETAIL, SCR-REVIEW-NIGHT).
- FR-# — Functional Requirement
  - Feature capability traced from the SRS (e.g., FR-6..8 Routine tracking).
- NFR-# — Non-Functional Requirement
  - Quality attributes (performance, privacy, offline, accessibility, etc.).
- AC-# — Acceptance Criteria
  - Verifiable outcomes tied to use cases and requirements.
- BR-# — Business Rule
  - Policy/constraint enforced by the app (e.g., title required).
- ST-UCx-yy — UI State Map ID
  - Named UI states for a given use case (navigation/flow anchors).

---

## Core Domain Entities

- Goal
  - A high-level objective the user wants to achieve.
- Plan
  - A container under a goal that groups milestones (e.g., “Main Plan”).
- Milestone
  - A concrete checkpoint within a plan; can have a target date and criteria.
- Routine
  - A recurring habit/check-in with a cadence and schedule; maintains streaks.
- Reminder
  - In-app scheduled prompt tied to routines/milestones (Active/Snoozed/Dismissed).
- Journal Entry
  - A reflection/notes item, often created during Night Review; can link to entities.
- Achievement / Badge
  - Recognition of progress (e.g., first milestone, streak thresholds).
- Template
  - Predefined structure for fast goal setup (plans/milestones/routines).
- Attachment
  - Note or photo captured as evidence for completed work (privacy-redacted by default).
- Export Artifact
  - A saved progress/summary dataset (JSON/CSV/Image/PDF) with redaction flags.
- AI Artifact
  - Metadata about AI suggestions/diagnostics (counts, confidence, acceptance).
- Progress Cache (derived)
  - Aggregated metrics for fast UI (milestone %, streaks, counts).

---

## Status & Fields (Common)

- Milestone.status — Pending | Completed
- Routine.streak — Number of consecutive completed cadence windows
- Routine.last_completed — Timestamp of last completion
- Reminder.status — Active | Snoozed | Dismissed
- Target Date — Optional due date for milestones; used for reminders and progress views

---

## Key Flows and Surfaces

- Morning Review (SCR-REVIEW-MORNING)
  - Select top priorities, adjust times, optional AI “plan my day,” then save Today’s Plan.
- Night Review (SCR-REVIEW-NIGHT)
  - Record completions/misses, add reasons and journal reflections; optional AI diagnostics.
- Today’s Checklist (SCR-CHECKLIST-TODAY / SCR-CHECKIN-PANEL)
  - Execute prioritized items; complete routines, mark milestones done, snooze/dismiss.
- Notification Center (SCR-NOTIFICATION-CENTER)
  - Unified overdue/today/upcoming reminders with bulk actions.
- Progress Hub (SCR-PROGRESS-HUB)
  - Overview of milestone %, streaks, overdue load; entry to Trends/Achievements/Export.
- Trends (SCR-TRENDS)
  - Weekly completion and streak trend visualizations.
- Achievements (SCR-ACHIEVEMENT-GALLERY / SCR-ACHIEVEMENT-DETAIL)
  - View earned/locked badges and criteria.
- Goal Detail (SCR-GOAL-DETAIL)
  - Manage plans/milestones, link routines, view progress; AI assist entry.
- Editors
  - Plan Editor (SCR-PLAN-EDITOR), Milestone Editor (SCR-MILESTONE-EDITOR), Milestone Quick Add (SCR-MILESTONE-QUICK-ADD), Routine Quick Add/Editor (SCR-ROUTINE-QUICK-ADD / SCR-ROUTINE-EDITOR).

---

## AI Terms (Assist & Diagnostics)

- AI Assist (UC-7)
  - Suggests milestones/steps with rationale and confidence; user must accept/edit before saving.
- AI Diagnostics (UC-8)
  - Explains stalled progress via observations, hypotheses (with confidence/assumptions), and proposed fixes.
- Confidence
  - Qualitative band (Low/Med/High) on AI outputs.
- Assumptions / Rationale
  - Short notes explaining why a suggestion or hypothesis was made.
- Proposed Fixes
  - Safe, actionable changes (split milestones, reschedule, adjust cadence, add planning/reflective prompts).
- Plan Map / Preview Changes
  - Steps to map accepted AI items to plans/milestones and confirm before persisting.
- Usage/Limits / Cooldowns
  - Controls and transparency for AI request frequency/cost; no background calls.

Privacy defaults for AI:
- No PII or journal free-text is sent unless the user includes it intentionally.
- AI outputs are suggestions; nothing is saved without explicit acceptance.

---

## Scheduling & Cadence

- Cadence
  - Daily, Weekly (select days), or Custom (every N days).
- Snooze / Dismiss
  - Temporarily delay or clear a reminder occurrence without marking completion.
- Streak / Streak at Risk
  - Consecutive on-time completions; items likely to break the streak highlighted.
- Grace Window
  - Sensible leeway around day boundaries/time zones to avoid unfair streak loss.

---

## Data, Export, Backup

- Export (Progress Snapshot)
  - JSON (full dataset), CSV (tables), Summary (image/PDF/HTML). Redaction ON by default.
- Redaction
  - Photos/notes/journal text are excluded from exports unless explicitly enabled.
- EXIF Stripping
  - Removes image metadata on export for privacy.
- Backup (UC-10)
  - Private archive (JSON + media) with a versioned manifest and checksums; optional encryption.
- Restore / Import
  - Validated, transactional replace (or advanced merge) from backup; includes migrations if needed.
- Manifest
  - Archive descriptor with schema/app version, included tables, media index, checksums.

---

## UX Patterns & Non-Functional Principles

- Offline-first / Local-first
  - All core flows work offline; AI gracefully disables with manual fallbacks.
- Privacy-first
  - Redaction defaults; explicit consent for sensitive content; metadata-only telemetry.
- Performance Targets
  - <150ms interactions; <1s screen renders; AI responses visible in ~6–8s.
- Accessibility (A11y)
  - WCAG AA contrast; keyboard operability; labeled controls; live regions for async updates.
- Resilience
  - Cancelable loaders; retries; atomic writes; transactional apply; idempotent achievements.
- Guidance vs. Enforcement
  - “Top 3” priorities and “≥2 milestones” are nudges (soft limits) unless otherwise stated.

---

## Error, Validation, and Offline Handling

- Validation Errors
  - Inline, descriptive, and block save only when required fields are missing/invalid.
- Storage/Save Errors
  - Clear recovery path with Retry; no partial persists.
- Offline Banners
  - Indicate limitations (e.g., AI disabled, cloud shares unavailable) while keeping local features working.
- Conflicts & Duplicates
  - Non-blocking warnings with options to merge or proceed.

---

## Telemetry & Observability (Privacy-Respecting)

- Event Naming Examples
  - goal_created, milestone_saved, routine_completed, morning_review_saved, night_review_completed,
    progress_hub_viewed, export_completed, ai_suggest_invoked, ai_diagnose_applied.
- Data Discipline
  - Hash or redact IDs; never log free-text from journals/notes or raw AI content.

---

## Common Acronyms & UI Terms

- CTA — Call to Action (primary/secondary buttons).
- PII — Personally Identifiable Information (never sent by default).
- A11y — Accessibility.
- EXIF — Exchangeable Image File format metadata (stripped on export).
- MVP — Minimum Viable Product (initial scope/behavior).
- FAB — Floating Action Button (e.g., “New Goal” on Dashboard).
- DOW — Day(s) of Week (used in weekly cadence selection).

---

## Examples (Cheat Sheet)

- Use Case IDs
  - UC-1 Create Goal, UC-2 Plan & Milestones, UC-3 Routines & Check-ins, UC-4 Morning Review,
    UC-5 Night Review, UC-6 Progress & Achievements, UC-7 AI Suggest Steps,
    UC-8 AI Diagnostics, UC-9 Success Capture & Export, UC-10 Backup & Restore.
- Screen IDs
  - SCR-DASH, SCR-CHECKLIST-TODAY, SCR-NOTIFICATION-CENTER, SCR-GOAL-LIST, SCR-GOAL-NEW-ENTRY,
    SCR-TEMPLATES, SCR-GOAL-FORM, SCR-GOAL-AI-SUGGEST, SCR-GOAL-REVIEW, SCR-GOAL-DETAIL,
    SCR-PLAN-EDITOR, SCR-MILESTONE-EDITOR, SCR-MILESTONE-QUICK-ADD,
    SCR-ROUTINE-LIST, SCR-ROUTINE-QUICK-ADD, SCR-ROUTINE-EDITOR,
    SCR-REVIEW-MORNING, SCR-REVIEW-NIGHT,
    SCR-PROGRESS-HUB, SCR-TRENDS, SCR-ACHIEVEMENT-GALLERY, SCR-ACHIEVEMENT-DETAIL,
    SCR-PROGRESS-EXPORT,
    SCR-AI-REFINE, SCR-AI-SUGGEST, SCR-PLAN-MAP, SCR-PREVIEW-CHANGES,
    SCR-AI-DIAGNOSE, SCR-DIAGNOSE-REFINE, SCR-DIAGNOSE-RESULT, SCR-PROPOSED-FIXES, SCR-PREVIEW-FIXES, SCR-USAGE.

---

Need something added here?
- If you spot unfamiliar IDs or terms in the design docs, add them to this appendix to keep the glossary up to date.