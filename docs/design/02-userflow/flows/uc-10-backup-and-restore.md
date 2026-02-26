# UC-10 — Backup/Export Data and Restore/Import on a Fresh Install

ID: UC-10  
Primary Actor: User (Self-improver, Busy professional, Analyst/optimizer, Wellness seeker)  
Scope: GoalPost app (Mobile + Web, single-user, local-first)  
Level: User goal  
Frequency: Periodic (weekly/monthly backups; imports during device/app changes)

Traceability (SRS):
- Core Use Case: UC-10 (Backup/export data and restore/import on a fresh install)
- Functional Requirements: FR-33..36 (Settings/Data Mgmt incl. export/import, deletion), FR-17 (Export snapshot option; related but distinct from full backup)
- Acceptance Criteria: AC-7 (Export produces a JSON file containing goals, plans, milestones, routines, journals, achievements)
- Non-Functional: NFR-3 (Reliability), NFR-4 (Availability/offline), NFR-5 (Security), NFR-6 (Privacy), NFR-9 (Observability), NFR-10 (Portability)

Related Screens:
- SCR-SETTINGS-DATA: Data & Privacy settings hub
- SCR-EXPORT-OPTIONS: Backup/export options (scope, format, encryption)
- SCR-EXPORT-PREVIEW: Manifest preview (what will be included)
- SCR-EXPORT-COMPLETE: Success with file location / open button
- SCR-IMPORT-OPTIONS: Restore/import entry (choose file, options)
- SCR-IMPORT-VALIDATE: Integrity and compatibility check, migration plan
- SCR-IMPORT-CONFIRM: Summary of changes (replace/merge), final confirm
- SCR-IMPORT-RESULT: Success/failure report with details
- SCR-DELETE-DATA: “Factory reset” (data deletion) with confirmations
- SCR-OFFLINE: Offline banner/modal (info only; local ops work)
- SCR-ERROR: Inline validation/recovery

Non-Functional Reminders:
- NFR-3 Reliability: Atomic writes; no partial data on failure. Show clear outcomes.
- NFR-4 Availability: Fully offline-capable; all operations local-first.
- NFR-5 Security: Keys stored securely; do not export secrets by default. Support optional encrypted backups.
- NFR-6 Privacy: Redaction defaults ON for “share exports”; full backups are private archives.
- NFR-10 Portability: Open formats (JSON/CSV) and bundled media; versioned manifest for migration.

---

## Goal/Benefit
- Create a trustworthy, portable backup of all personal data for safekeeping or device migration.
- Restore a user’s data on a fresh install quickly, safely, and predictably, with compatibility checks and migrations applied.

Outcome: A versioned backup archive (manifest + JSON/CSV + media) is created and storable by the user. A restore operation validates integrity/compatibility and either replaces or merges data (per user choice) without partial failures.

---

## Definitions
- Backup (private archive): A local archive suitable for full restore (includes all entities and attached media). Secrets (e.g., API keys) excluded by default.
- Share Export (summary/report): A privacy-redacted summary (JSON/CSV or visual) intended to share. Not sufficient for full restore.
- Manifest: JSON file describing app version, schema version, checksums, included tables, media index, and options.

---

## Preconditions
- Local storage available; user has permission to read/write files.
- For restore/import: user has a previously created backup file accessible in the file system.
- Optional: An encrypted backup requires a passphrase to decrypt.

## Triggers
- From SCR-SETTINGS-DATA, user selects:
  - “Create Backup” (full private archive), or
  - “Export Data (JSON/CSV)” (portable data), or
  - “Restore/Import from Backup”.
- First-run fresh install shows a card: “Restore from backup?” (optional).

## Postconditions (Success)
- Backup: Archive file created with manifest, data, and media; user sees its path.
- Restore: Data store reflects imported content; app restarts data session and reindexes. User lands on Dashboard with their content.
- Logs contain metadata-only events (no PII), for success/failure auditing.

## Postconditions (Failure/Abort)
- On cancel: No files written/modified.
- On error: No partial data persists; original state remains intact. A clear error reason is shown with retry guidance.

---

## Primary Flow A — Create Backup (Private Archive)
1) Entry — SCR-SETTINGS-DATA
   - “Create Backup” CTA with info: “Private archive for full restore. Secrets excluded by default.”

2) Configure — SCR-EXPORT-OPTIONS (Backup Mode)
   - Scope: “Full Backup” (all entities, templates, AI artifacts metadata, attachments/media)
   - Format: 
     - “Archive (.zip/.tar) with JSON data + media” (default)
     - “JSON only (no media)” (advanced)
   - Security:
     - “Encrypt backup with password” (recommended) OFF by default
     - “Include app settings (no API keys)” ON by default
     - “Include API keys” OFF by default (requires encryption ON and explicit opt-in)
   - CTA: “Next: Preview”

3) Preview — SCR-EXPORT-PREVIEW
   - Show manifest preview:
     - app_version, schema_version, created_at
     - tables: goals, plans, milestones, routines, reminders, journals, achievements, templates, attachments index, ai_artifacts metadata
     - media_count, total_size (estimate)
     - secrets_included: false (unless toggled)
   - CTA: “Create Backup”

4) Generate — Progress
   - Steps:
     - Snapshot data to JSON files per table
     - Build attachments/media index and copy media into archive
     - Compute checksums (per file and overall)
     - Write manifest.json (with checksums, counts, versions)
     - If encryption enabled, encrypt archive with user passphrase
     - fsync/atomic rename to final filename (e.g., goalpost_backup_2026-02-26T2015Z.gpb.zip)
   - On success → SCR-EXPORT-COMPLETE with file path + “Open File”
   - Toast: “Backup created”

5) Result
   - Reliable, portable backup created locally. No cloud dependency.

---

## Primary Flow B — Restore/Import on a Fresh Install (Replace)
1) Entry — SCR-SETTINGS-DATA or First-Run Card
   - “Restore from Backup” CTA.

2) Choose File — SCR-IMPORT-OPTIONS
   - File picker (local). Accepts:
     - Encrypted/unencrypted backup archives (.gpb.zip/.zip/.tar)
     - Raw JSON export (reduced restore; warns about missing media)
   - If encrypted, prompt passphrase (masked input). 
   - Restore Mode:
     - Replace (fresh install default): replace all local data with backup
     - Merge (advanced; see Alternate Flow C)

3) Validate — SCR-IMPORT-VALIDATE
   - Steps:
     - Integrity: verify archive structure, read manifest.json
     - Checksums: verify per-file and overall hash
     - Compatibility: schema/app version check; plan migrations if needed
     - Size check: ensure local storage capacity
     - Dry-run migration diff: items to create/update; media to extract
   - If issues:
     - Incompatible version → show migration path or “Update app then retry”
     - Corrupt file → abort; show “File appears corrupted”
     - Wrong passphrase → “Incorrect password” with retry

4) Confirm — SCR-IMPORT-CONFIRM
   - Summary:
     - Mode: Replace (default for fresh install)
     - Entities: counts per table
     - Media: count and total size
     - Estimated time and space
   - Warning: “This will replace current local data” (if any)
   - CTA: “Restore Now”

5) Apply — Transactional Restore
   - Steps:
     - Create temp workspace
     - Extract archive (decrypt if needed) and verify structure
     - Apply migrations on JSON snapshot into temp DB
     - Atomic switch: replace current DB with temp DB
     - Extract media to app media store (idempotent; verify checksums)
     - Reindex derived caches
   - On success → SCR-IMPORT-RESULT with summary and “Go to Dashboard”
   - Toast: “Restore complete”

6) Result
   - All content restored. User is routed to Dashboard with their data.

---

## Alternate Flows
A1) Export Portable Data (JSON/CSV) Without Media
- From SCR-EXPORT-OPTIONS choose “JSON only” or “CSV tables”.
- Use when migrating subsets or for analytics in external tools.
- Warn: “Not sufficient for full restore unless paired with media.”

A2) Encrypted Backup with Secrets
- User enables encryption and explicitly opts to include API keys.
- Keys are decrypted only at restore time into secure storage; never logged/exported in plaintext.
- If passphrase lost → cannot restore secrets; proceed without keys.

A3) Scheduled Backup Reminder
- Optional setting: weekly/monthly reminder to create a backup.
- Creates no automatic backups without explicit user action (NFR-12).

A4) Verify Backup
- After creation, user can “Verify Backup”:
  - Re-open archive, validate manifest/hash; show “Backup verified”.

A5) Rolling Backups
- Offer naming with timestamp; optionally prune older backups after user confirmation.

A6) Import Dry-Run Only
- User runs validation/migration plan without applying changes; receives a report.

A7) Import From JSON Only
- If user selects raw JSON (no archive), restore imports data but skips media (warn and continue).

A8) Factory Reset (Delete All Data)
- From SCR-DELETE-DATA, user confirms (2-step confirm) to wipe local data (FR-36).
- Offer immediate “Restore from Backup” CTA afterward.

A9) Merge Mode (Advanced)
- Instead of Replace, Merge attempts to combine:
  - New items from backup added
  - Conflicts resolved by “prefer backup” or “prefer local”
- Default for fresh install is Replace; Merge is hidden or advanced-only.

---

## Exception Flows
E1) Permission Denied (File Access)
- Show rationale and “Open Settings” to grant storage access.
- Allow retry or cancel.

E2) Low Disk Space
- Block operation; show required vs. available space and allow “Change Location” or cancel.

E3) Corrupted Archive / Failed Checksum
- Abort with “Backup file appears corrupted or incomplete.” Provide retry/select another file.

E4) Incompatible Version, No Migration Path
- “Backup was created with a newer schema. Update the app, then retry.”
- If older schema, attempt forward migration; otherwise instruct to export to JSON from older app first (if possible).

E5) Wrong Passphrase (Encrypted Backup)
- Allow multiple retries with cool-down; never log passphrase.

E6) Partial Failure During Apply
- Use transactional/atomic replace:
  - If any step fails, roll back to prior state.
  - Show diagnostic with “Retry” and “Report Issue” (redacted logs).

E7) Interrupted Operation (App Closed / Crash)
- On next launch, detect incomplete temp workspace:
  - Offer “Resume” or “Rollback” safely.

---

## Data Created/Updated
- Export Artifact (Backup)
  - { id, created_at, format: “archive|json_only”, encrypted: boolean, include_settings: boolean, include_api_keys: boolean, manifest_ref, file_path }
- Manifest (manifest.json inside archive)
  - { app_version, schema_version, created_at, tables: [names], counts: { per_table }, media: { count, index[] }, checksums: { files[], overall }, options: { include_settings, include_api_keys } }
- Import Log (local-only)
  - { id, started_at, completed_at, source_manifest, mode: replace|merge, result: success|failure, summary_counts, errors?: [] }
- Local Data Store
  - Replaced or merged entities: goals, plans, milestones, routines, reminders, journals, achievements, templates, attachments refs, ai_artifacts (metadata only)
- Secure Settings (if included secrets with encryption)
  - Keys rehydrated to secure storage provider; never stored in plaintext files.

---

## Business Rules & Constraints
- BR-1: Backups are private archives; default excludes API keys. Including secrets requires encryption + explicit opt-in.
- BR-2: All writes are atomic; never leave the app in a partially restored state.
- BR-3: Manifests must include schema version and checksums; restore must validate both before apply.
- BR-4: Media is bundled within the archive and validated by checksum before linking.
- BR-5: Share Exports (reports/summaries) default to redaction ON; Backups default to full data for restore but remain private and local.
- BR-6: No background backups without explicit user action (NFR-12).
- BR-7: Logs contain metadata only; never include file paths with PII, secrets, or passphrases.
- BR-8: If Merge is offered, conflict policy must be explicit and visible; default remains Replace for fresh installs.

---

## Analytics & Telemetry (Privacy-Respecting)
- Event: backup_initiated { format, encrypted, include_api_keys }
- Event: backup_completed { duration_ms, size_mb, success: boolean }
- Event: backup_verified { success: boolean }
- Event: import_initiated { mode, encrypted: boolean }
- Event: import_validated { compatibility: ok|migrate|blocked, estimated_counts }
- Event: import_applied { duration_ms, result: success|failure, created: n, updated: m, media: k }
- Event: storage_error { type: permission|disk_space|corrupt|incompatible }
Note: No PII, no file paths, no secrets, no passphrases in telemetry.

---

## Acceptance Criteria (Derived)
- AC-10.1: User can create a full backup archive containing goals, plans, milestones, routines, journals, achievements, templates, attachments (media), and AI artifact metadata. A manifest with version and checksums is included.
- AC-10.2 (from AC-7): User can export a JSON data file containing goals, plans, milestones, routines, journals, achievements (JSON-only export).
- AC-10.3: Restore/import validates integrity and compatibility, applies necessary migrations, and replaces local data transactionally on a fresh install.
- AC-10.4: Encrypted backups require a passphrase to restore; secrets are not exported unless user explicitly opts in with encryption enabled.
- AC-10.5: On any error, the restore aborts without altering existing data, and shows a clear recovery path.

---

## Open Questions / TBD
- Should encryption be required when including secrets, or merely recommended with strong warnings?
- Preferred archive format for cross-platform portability (.zip vs .tar.gz)?
- Do we support delta/backups (incremental) post-MVP?
- Should “Merge” be hidden in MVP and enabled later for advanced users?
- Maximum backup size guidance and optional media compression?
- Do we include app logs (redacted) in backups for support scenarios, or keep backups user-data-only?

---

## UI State Map
- ST-UC10-01: SCR-SETTINGS-DATA (Data & Privacy hub)
- ST-UC10-02: SCR-EXPORT-OPTIONS (Backup/export options)
- ST-UC10-03: SCR-EXPORT-PREVIEW (Manifest preview)
- ST-UC10-04: SCR-EXPORT-COMPLETE (File saved, open)
- ST-UC10-05: SCR-IMPORT-OPTIONS (Pick file, mode)
- ST-UC10-06: SCR-IMPORT-VALIDATE (Checksums, compatibility, migration plan)
- ST-UC10-07: SCR-IMPORT-CONFIRM (Summary, confirm)
- ST-UC10-08: SCR-IMPORT-RESULT (Success/failure)
- ST-UC10-ERR: SCR-ERROR (validation/storage), SCR-OFFLINE (banner)

---

## Wireflow (Textual)
- Create Backup
  - SCR-SETTINGS-DATA → “Create Backup”
    - SCR-EXPORT-OPTIONS (Backup; choose encryption/settings) → “Next”
      - SCR-EXPORT-PREVIEW (manifest) → “Create Backup”
        - Progress → SCR-EXPORT-COMPLETE (file path; “Open File”)

- Restore on Fresh Install (Replace)
  - First-Run Card or SCR-SETTINGS-DATA → “Restore from Backup”
    - SCR-IMPORT-OPTIONS (pick .gpb.zip; enter passphrase if needed; mode=Replace)
      - SCR-IMPORT-VALIDATE (integrity, checksums, compatibility; migration plan)
        - SCR-IMPORT-CONFIRM (summary; warnings) → “Restore Now”
          - Progress (transactional apply) → SCR-IMPORT-RESULT → “Go to Dashboard”

- Export JSON Only
  - SCR-SETTINGS-DATA → “Export Data (JSON/CSV)”
    - SCR-EXPORT-OPTIONS (JSON only) → “Next”
      - SCR-EXPORT-PREVIEW → “Export” → SCR-EXPORT-COMPLETE