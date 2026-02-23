# User Flow: Share Achievement & Social Features

Summary
-------
This user flow describes how a user captures and shares an achievement in GoalPost. It covers sharing to external social networks, internal feeds and accountability buddies, generating shareable media (cards/photos), privacy controls, post-creation interactions (comments, reactions), and analytics. It includes happy paths, alternative flows, error handling, data model notes, acceptance criteria, and implementation considerations.

Actors
------
- User: a signed-in individual who created or completed a goal/milestone.
- System: GoalPost backend, UI components, media generation service, notification system.
- Recipient: another user in GoalPost (accountability buddy) or external social platform (e.g., Twitter, Instagram).
- Third-party services: social sharing APIs, image hosting (if applicable).

Preconditions
-------------
- The user has at least one completed milestone or goal.
- The user is signed in.
- The user has granted necessary permissions for external sharing (when required).
- Media generation and social API services are available.

Entry Points
------------
- `Goal Overview` after marking a milestone/goal as complete: `Share Achievement` CTA.
- `Achievements` feed: overflow menu on a completed item -> `Share`.
- `Profile` achievements list -> share icons.
- Post-creation prompt: `Share your success` toast with quick actions.

Primary (Happy) Flow — Share to Internal Feed / Accountability Buddy
-------------------------------------------------------------------
1. Trigger
   - You tap `Share` on a completed milestone or goal (from `Goal Overview` or `Achievements` feed).

2. Share Sheet Opens
   - Screen: `Share Achievement` modal with tabs/options:
     - `Internal Feed` (GoalPost)
     - `Accountability Buddy`
     - `Export Image / Photo`
     - `External Networks` (if connected)
   - UI elements: title, auto-generated description, generated achievement card preview, privacy selector, message input, emoji/reaction picker, `Share` and `Cancel` buttons.

3. Choose Internal Feed
   - You select `Internal Feed`.
   - Optionally edit message, add tags, mention buddies (`@username` autocomplete), attach photo (from camera or gallery).
   - Choose visibility: `Public`, `Friends`, `Private`.
   - Tap `Share`.

4. System Publishes Post
   - System creates a `Post` linked to the achievement (post metadata includes `achievement_id`, `creator_id`, text, media).
   - Post appears in your feed and (based on visibility) in followers' feeds and relevant discovery pages.
   - Notifications:
     - If you mentioned someone, that user receives a mention notification.
     - Followers receive a feed update notification (respecting user notification preferences).
   - UX: Show confirmation `Shared to Feed` and quick actions (e.g., `View Post`, `Share Externally`).

Primary Flow — Share with Accountability Buddy (Direct)
-------------------------------------------------------
1. Choose `Accountability Buddy` option.
2. UI shows list of buddies with search and recent contacts; supports selecting multiple recipients.
3. Add a personalized message and optional private photo.
4. Tap `Send`.
5. System sends an in-app message with the achievement card and optionally triggers push notifications to recipients.
6. Recipients can react/acknowledge (like, cheer) and reply in-thread.

Primary Flow — Export Image & Share Externally
----------------------------------------------
1. Choose `Export Image / Photo`.
   - System generates an achievement card image (templates for `Photo Card`, `Story`, `Banner`).
   - UI shows generated preview with edit controls: background, quote, stats, photo overlay, watermark toggle.
2. Edit & Export
   - You edit text/visuals and tap `Export`.
   - System renders high-resolution image (client-side or server-side).
3. Share Sheet (OS-level)
   - OS share sheet opens with the exported image and prefilled message; you select an external app (Instagram, X, WhatsApp).
   - For platforms supporting direct API post (optional), app can attempt to post directly if OAuth is configured.
4. On Success
   - System logs the external share event and optionally prompts to `Link this post` (user provides URL) to attach to the achievement in your profile.

Alternative Flows
-----------------
A1 — Quick Share (One-tap)
  - After completing a milestone, the system shows a one-tap `Share` button with recommended message; tapping it shares to most recently used destination or internal feed using default visibility.

A2 — Anonymous / Private Share
  - For `Private` visibility, the post is stored but not broadcast; you can share a link to selected recipients only.

A3 — Batch Share
  - You can select multiple achievements and share them as a single `Progress Update` post (aggregate card with summary stats).

A4 — Re-share / Amplify
  - For older achievements, you can `Reshare` which republishes the post with updated timestamp and optional commentary.

Error Flows
-----------
E1 — Media Generation Failure
  - If image/card generation fails:
    - Show `Unable to generate media right now` with `Retry` and `Share without image` options.
    - If server-side, fall back to a lightweight client-side template.

E2 — Social API Failure / Permission Denied
  - If external API fails or lacks permissions:
    - Show clear error: e.g., `Could not post to Instagram. Reauthorize app or use OS share sheet.`
    - Offer `Open Settings` / `Re-authorize` action.

E3 — Rate Limits / Quota Exceeded
  - If sharing endpoint is rate-limited:
    - Show `Sharing temporarily unavailable. Try again later.` queue the share attempt if feasible.

E4 — Privacy Conflict
  - If a user attempts to share a post that includes another user's private content:
    - Block publish and show `Cannot share: contains private content` with guidance.

E5 — Offline / Network Unavailable
  - Queue the share action locally and present `Pending Share` state; auto-retry when online. Allow manual `Retry` and `Cancel`.

UI Wireframes / Screen Map (high-level)
---------------------------------------
- `Goal Overview`
  - CTA: `Share Achievement` -> `Share Achievement` modal
- `Share Achievement Modal`
  - Header: Achievement title | `Preview` thumbnail
  - Tabs: `Internal Feed` | `Buddies` | `Image` | `External`
  - Controls: Message input, tags, visibility, attach photo, template selector
  - Actions: `Share` | `Cancel`
- `Export Image / Template Editor`
  - Canvas preview, template toolbar, export button
- `Post View`
  - Achievement card, reactions, comments, share actions

Data Model Notes
----------------
- Achievement
  - id, user_id, goal_id, milestone_id (nullable), title, description, completed_at, stats
- Post
  - id, creator_id, achievement_id (nullable), text, media[], visibility, tags[], created_at
- MediaAsset
  - id, post_id, type (image|video), url, metadata (template_id, dimensions)
- ShareRecord
  - id, user_id, post_id (nullable), external_service, external_post_url, status, created_at
- BuddyMessage
  - id, sender_id, recipient_ids[], achievement_id, text, read_status
- Notification
  - id, user_id, type, payload, read_at
- Audit & Analytics
  - share_events: user_id, achievement_id, destination, success/failure, timestamp

Acceptance Criteria
-------------------
- You can share an achievement to the GoalPost internal feed with editable message and visibility controls.
- You can share directly to one or more accountability buddies with an in-app message and notification.
- You can generate a shareable image/card, edit it, and export it for external sharing.
- Privacy settings are respected; private achievements are not publicly visible.
- Sharing actions emit analytics events and handle offline/queued states.
- Mentions and tags notify recipients appropriately and support autocomplete.
- Errors during share present clear, actionable UI and fallback options.

Edge Cases & Considerations
---------------------------
- Duplicate content: detect and warn if the same achievement has been shared multiple times recently to avoid spam.
- Accessibility: generated images must include alt text and a textual fallback for screen readers when posted internally.
- Internationalization: template text and date/time formatting must honor locale settings.
- GDPR / Data retention: respect user requests to remove shared posts and associated media; provide deletion endpoints for external links where possible.
- Abuse & moderation: implement reporting, takedown, and content moderation workflows for public posts.
- Preview caching: when linking to external posts, cache metadata but revalidate on-demand to avoid stale links.
- Watermark / Branding: provide an option to toggle GoalPost branding on exported images.
- Templates A/B testing: allow experimenting with card templates and measure sharing conversion.

Implementation Notes
--------------------
- Generate cards using a templating system that can run client-side (Canvas/WebGL) for quick edits and server-side for high-resolution exports.
- Treat `Share` as an idempotent operation: use client-generated `client_request_id` to deduplicate retries.
- Use a background job queue for external API posting to avoid blocking client and to handle retries gracefully.
- Store `ShareRecord` for auditability and for linking external posts back to achievements.
- Implement granular permission checks: who can see shared content based on `visibility` and relationships.
- Track analytics keys: `achievement_shared`, `achievement_shared_destination`, `achievement_image_exported`, `achievement_mentioned_user`.
- Provide webhooks or integrations for enterprise customers to mirror achievements into Slack, Teams, or other collaboration tools.

Related Design Files
--------------------
- `docs/design/01-draft/overview.md` — project goals and sharing motivations.
- `docs/design/02-userflow/create-goal.md` — for linking achievements to goals and templates.
- `docs/design/02-userflow/track-progress.md` — for triggers when achievements complete.

Revision History
----------------
- 2026-02-12 — Initial draft of share achievement and social features.
