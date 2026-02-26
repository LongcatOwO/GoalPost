# UC-9 — Capture Success Evidence and Export a Shareable Summary

ID: UC-9  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user, local-first)  
Level: User goal  
Frequency: Medium (on milestone completion; weekly/monthly for exports)

Traceability (SRS):
- Core Use Case: UC-9 (Capture photos/notes for completed milestones and export a shareable summary)
- Functional Requirements: FR-30 (Attach notes/photos), FR-31 (Generate shareable artifact/export), FR-32 (Privacy controls with redaction by default)
- Supporting: FR-15..17 (Progress metrics and export), FR-12..14 (Reviews integration), FR-33..36 (Settings/Data)
- Acceptance Criteria: AC-7 (Export full dataset JSON), AC-8 (Exports hide photos by default unless explicitly included)

Related Screens:
- SCR-GOAL-DETAIL: Goal detail (milestones/attachments area)
- SCR-MILESTONE-DETAIL: Milestone detail (attachments, completion context)
- SCR-SUCCESS-CAPTURE: Quick success capture panel (notes/photos)
- SCR-ATTACHMENT-EDITOR: Attachment details (caption, tags, redact flags)
- SCR-MEDIA-PICKER: Camera/Gallery picker (mobile), File picker (web)
- SCR-PROGRESS-HUB: Overview of progress (entry to export)
- SCR-EXPORT-OPTIONS: Scope, format, redaction toggles
- SCR-EXPORT-PREVIEW: Preview redactions and included items
- SCR-EXPORT-COMPLETE: Success screen with file location
- SCR-OFFLINE: Offline banner/modal
- SCR-ERROR: Inline validation/recovery

Non-Functional Reminders:
- NFR-1 Usability: Add a note/photo in ≤ 30s; export in ≤ 1 min
- NFR-2 Performance: <150ms UI interactions; preview render <1s for ≤ 200 attachments
- NFR-4 Availability: Fully works offline (local files); no cloud dependency
- NFR-5 Security: Strip sensitive metadata (EXIF) from exported images; secure local storage where platform supports
- NFR-6 Privacy: Redaction ON by default; explicit opt-in to include photos/notes
- NFR-7 Accessibility: Alt text for images; keyboard/touch friendly; WCAG AA
- NFR-9 Observability: Metadata-only logs; never log photo content or note text

---

## Goal/Benefit
- Capture concrete evidence of success (notes/photos) to reinforce motivation and build a portfolio of achievements.
- Export a privacy-safe progress summary (JSON/CSV and/or static summary page/image/PDF) to archive or share selectively.

Outcome: Attachments are linked to completed milestones (and optionally routines); an export artifact is created with redaction by default and explicit opt-in needed to include sensitive content.

---

## Preconditions
- At least one Milestone exists; completion can be logged from Goal/Milestone detail or during Night Review.
- Local file system available for storing attachments and export files.
- Permissions granted as needed (camera/photos library on mobile; file access on web/desktop).
- App privacy defaults configured (redaction ON).

## Triggers
- From SCR-MILESTONE-DETAIL after marking “Completed”: “Add Photo/Note”
- From SCR-REVIEW-NIGHT (“Quick Wins” card): “Add Evidence”
- From SCR-PROGRESS-HUB or SCR-GOAL-DETAIL: “Export Summary”

## Postconditions (Success)
- One or more attachments saved and linked to the completion (FR-30).
- An export file is created with chosen scope/format, respecting redaction defaults (FR-31..32).
- User can locate/open the export and optionally share it externally.

## Postconditions (Failure/Abort)
- If user cancels before save, no attachment/export is persisted.
- If permissions denied or storage error occurs, user sees recoverable errors with retry options.

---

## Primary Flow A — Attach Notes/Photos to a Completed Milestone

1) Entry — SCR-MILESTONE-DETAIL or SCR-REVIEW-NIGHT
   - User taps “Add Evidence” or “Add Photo/Note”.

2) Choose Evidence Type — SCR-SUCCESS-CAPTURE
   - Options: “Add Note”, “Add Photo”
   - Inline hint: “Private by default. Visible only to you.”

3) Add Note — SCR-ATTACHMENT-EDITOR
   - Inputs: Text (required), Optional Tags (e.g., “PR”, “PB”, “Team”)
   - Redaction: Toggle “Include in exports” default OFF
   - CTA: “Save Note”

4) Add Photo — SCR-MEDIA-PICKER → SCR-ATTACHMENT-EDITOR
   - Source: Camera or Gallery (mobile); File picker (web)
   - Inputs: Caption (optional), Alt text (required for accessibility)
   - Redaction: Toggles
     - “Include photo in exports” (default OFF)
     - “Strip location & metadata (EXIF)” (default ON)
   - CTA: “Save Photo”

5) Persist & Link
   - System saves attachment(s) linked to the milestone completion with timestamps.
   - Toast: “Evidence saved.” Attachment thumbnails appear on milestone detail.

Result: FR-30 satisfied. Evidence stored locally, private by default.

---

## Primary Flow B — Export a Progress Summary

1) Entry — SCR-PROGRESS-HUB or SCR-GOAL-DETAIL
   - User taps “Export Summary”.

2) Configure Export — SCR-EXPORT-OPTIONS
   - Scope:
     - “All Data” (Goals, Plans, Milestones, Routines, Journals, Achievements)
     - “Selected Goal”
   - Format:
     - JSON (full data, AC-7)
     - CSV (core tables)
     - Summary (Image/PDF or static HTML page) — MVP supports at least JSON; CSV or summary optional if feasible
   - Redaction (default ON):
     - Include Photos: OFF (AC-8)
     - Include Notes Text: OFF (export redacted placeholders)
     - Include Journal Text: OFF (export counts/tags only)
     - Strip metadata (EXIF) from any included images: ON
   - Time Range (optional for summary): All / last 4/8/12 weeks
   - CTA: “Next: Preview”

3) Preview — SCR-EXPORT-PREVIEW
   - Show items included with redaction badges:
     - Photos: “Hidden (redacted)” unless user toggles ON
     - Notes/Journal: “Hidden (redacted)” unless toggled ON
   - Warning when toggling ON:
     - “You are including personal content. Proceed?”
   - CTA: “Export”

4) Generate & Complete — SCR-EXPORT-COMPLETE
   - System writes export file to local storage (choose path/name).
   - Show file path and “Open File” button.
   - Toast: “Export created. Photos hidden by default.”

Result: FR-31..32 satisfied; AC-7 and AC-8 validated.

---

## Primary Flow C — Quick Share of a Redacted Summary (Local-Only)
1) Entry — SCR-EXPORT-OPTIONS (Summary format selected)
2) Defaults: Photos OFF, Notes/Journal OFF, Strip metadata ON
3) Export creates a local static summary (e.g., HTML or image/PDF)
4) System shows OS “Open with…” or “Copy file path” (no cloud dependency)

Result: User can share locally saved summary through OS mechanisms while preserving defaults.

---

## Alternate Flows

A1) Bulk Attach After-the-Fact
- From SCR-GOAL-DETAIL → “Manage Evidence”
- User selects multiple images (Gallery/File picker)
- Attach to selected milestone(s) with bulk caption/tag and redaction defaults

A2) Capture During Night Review
- In “Quick Wins”, user taps “Add Photo” inline
- Same editor as Primary Flow A; returns to Night Review on save

A3) Quick Export from Achievement Toast
- After awarding a badge, CTA “Export Summary”
- Pre-fills “Selected Goal” scope; redaction defaults remain ON

A4) Export Presets
- User saves export settings as a preset (scope, format, redactions)
- Next time, “Use Last Preset” → Export directly

---

## Exception Flows

E1) Permission Denied (Camera/Photos)
- Show modal with succinct rationale and “Open Settings”
- Offer “Add Note” as fallback

E2) Storage Error / Low Disk Space
- Error banner: “Couldn’t save file. Free up space or choose another location.”
- CTA: “Retry” or “Change Location”

E3) Large Media
- If image > size threshold, prompt to compress (non-destructive)
- Provide progress indicator; never block UI

E4) Offline Mode
- Fully supported; local export path only
- Any “Share link” cloud option remains disabled (post-MVP)

E5) Privacy Risk Warning
- If user enables photo/note inclusion, confirm modal:
  - “You are including personal content. Review carefully before sharing.”

---

## Data Created/Updated

- Attachment
  - { id, entity_ref: milestone_id|routine_id, type: “photo|note”, uri/path, caption?, alt_text?, tags?, created_at, updated_at, export_include: boolean (default false), exif_stripped_on_export: boolean (default true) }

- Export Artifact
  - { id, created_at, scope: “all|goal:<id>”, format: “json|csv|summary”, redaction_flags: { include_photos: boolean, include_notes: boolean, include_journal_text: boolean, strip_exif: boolean }, time_range?, file_path }

- Milestone (reference only; status may already be Completed)
  - { id, completed_at?, attachments[] }

- Journal/Notes (if linked)
  - { id, date, text, reason_tags?, linked_entities[] } — text export default redacted

Note: For summary formats, derived metrics (e.g., milestone %, streaks) are computed at export time.

---

## Business Rules & Constraints

- BR-1: Privacy-first defaults — photos/notes/journal text excluded from exports unless explicitly opted-in (FR-32; AC-8).
- BR-2: EXIF and similar metadata are stripped from included images by default (NFR-5/6).
- BR-3: Attachments are stored locally and remain private unless exported with opt-in.
- BR-4: Attachment redaction flags are evaluated at export time; changing flags affects future exports, not past ones.
- BR-5: File naming convention includes app name, scope, and timestamp (e.g., goalpost_summary_goal-123_2026-02-26T2015Z.json).
- BR-6: Summary export shows metrics and thumbnails only if photos are explicitly included; otherwise show placeholders or omit.
- BR-7: Accessibility: alt text required when adding a photo to support assistive technologies (NFR-7).

---

## Analytics & Telemetry (Privacy-Respecting)

- Event: evidence_add_started { type: “note|photo” }
- Event: evidence_saved { type, redaction_on: boolean }
- Event: export_started { scope, format, redaction: { photos:false|true, notes:false|true, journal:false|true, strip_exif:true|false } }
- Event: export_completed { duration_ms, file_size_kb, success: boolean }
- Event: export_validation_warning { type: “privacy_toggle_on|exif_warning|large_media” }

Note: Do not log note text, photo content, or file paths containing PII.

---

## Acceptance Criteria (Derived)

- AC-9.1: User can attach a note and/or a photo to a completed milestone; attachments appear on milestone detail (FR-30).
- AC-9.2: User can export a full dataset (JSON) containing goals, plans, milestones, routines, journals, achievements (AC-7).
- AC-9.3: By default, exported summaries hide photos and redact note/journal text; user can explicitly opt-in to include them (FR-31..32; AC-8).
- AC-9.4: If images are included, EXIF and similar metadata are stripped by default (NFR-5/6).
- AC-9.5: Export completes successfully to a user-selected location and indicates where the file was saved (FR-17 indirect).

---

## Open Questions / TBD

- Should summary export (HTML/PDF/image) be in MVP or follow JSON-only first?
- Do we support basic image compression and/or face blurring at MVP, or defer?
- How should we handle duplicate attachments on multi-select bulk attach?
- Should alt text be required or “strongly recommended” with a reminder?
- Do we allow per-attachment overrides of global redaction defaults at export time?

---

## UI State Map

- ST-UC9-01: SCR-MILESTONE-DETAIL (attachments panel)
- ST-UC9-02: SCR-SUCCESS-CAPTURE (choose note/photo)
- ST-UC9-03: SCR-MEDIA-PICKER (camera/gallery/file)
- ST-UC9-04: SCR-ATTACHMENT-EDITOR (caption, alt, tags, redaction)
- ST-UC9-05: SCR-PROGRESS-HUB → “Export Summary”
- ST-UC9-06: SCR-EXPORT-OPTIONS (scope, format, redaction)
- ST-UC9-07: SCR-EXPORT-PREVIEW (review redactions)
- ST-UC9-08: SCR-EXPORT-COMPLETE (file path, open)
- ST-UC9-ERR: SCR-ERROR (validation/storage), SCR-OFFLINE (banner)

---

## Wireflow (Textual)

- Attach Evidence
  - SCR-MILESTONE-DETAIL → “Add Evidence”
    - SCR-SUCCESS-CAPTURE → “Add Photo”
      - SCR-MEDIA-PICKER → select/capture
        - SCR-ATTACHMENT-EDITOR → set caption, alt, redaction defaults → “Save Photo”
          - Back to SCR-MILESTONE-DETAIL (thumbnail appears)
    - or “Add Note” → SCR-ATTACHMENT-EDITOR → write note (redaction OFF by default) → “Save Note” → Back

- Export Summary
  - SCR-PROGRESS-HUB → “Export Summary”
    - SCR-EXPORT-OPTIONS → choose scope, format (JSON), confirm redaction defaults (photos OFF, notes OFF, journal OFF, strip EXIF ON) → “Next”
      - SCR-EXPORT-PREVIEW (show placeholders for redacted content) → “Export”
        - SCR-EXPORT-COMPLETE (show file path; “Open File”)