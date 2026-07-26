# DesignCraft Bot — Bot specification

**Archetype:** custom

**Voice:** professional and creative — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that transforms free-form fashion and logo requests into finished visuals and branded PDF packages. Users submit briefs, receive preview images, and download high-res assets, editable source files, and a branded PDF with mockups and specs. The bot supports lightweight revisions and tracks job status until completion.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual consumers
- small business owners

## Success criteria

- User receives preview images within 5 minutes of submitting a request
- User can download final assets and PDF package within 24 hours
- Admin receives notifications for new jobs and failures

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and welcome message
- **/help** (command, actor: user, command: /help) — Show help and usage instructions
- **/status** (command, actor: user, command: /status) — Check the status of a design job
- **Accept** (button, actor: user, callback: accept:job) — Confirm acceptance of preview designs
  - inputs: job_id
  - outputs: final assets, PDF package
- **Revise** (button, actor: user, callback: revise:job) — Request revisions to preview designs
  - inputs: job_id, revision_notes
  - outputs: revised preview images
- **Download** (button, actor: user, callback: download:job) — Download final package of assets
  - inputs: job_id
  - outputs: download links

## Flows

### Design Request Flow
_Trigger:_ user message

1. User sends free-form request
2. Bot acknowledges and confirms with summary
3. Bot returns 1-3 preview images
4. User selects Accept or Revise
5. If accepted, deliver final package
6. If revised, show updated previews

_Data touched:_ Request, Design job

### Status Check Flow
_Trigger:_ /status

1. User requests job status
2. Bot displays current status and progress

_Data touched:_ Job status

### Admin Notification Flow
_Trigger:_ job status change

1. New job created
2. Admin receives notification
3. Job failure detected
4. Admin receives failure alert

_Data touched:_ Job status

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Request** _(retention: persistent)_ — User-submitted free-form brief with optional deadline
  - fields: user_id, text, deadline, timestamp
- **Design job** _(retention: persistent)_ — Generated logo and/or fashion visuals derived from Request
  - fields: job_id, request_id, status, preview_images, final_assets, revision_count
- **Assets** _(retention: persistent)_ — Generated deliverables including preview images, high-res images, editable source files, and PDF package
  - fields: asset_id, job_id, file_type, download_url, timestamp
- **Job status** _(retention: persistent)_ — Current processing state of a design job
  - fields: job_id, status, timestamp, admin_notes

## Integrations

- **Telegram** (required) — Bot API messaging
- **Admin Report Channel** (required) — Send job alerts and failures to owner/admin
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Admin report channel for job alerts
- Configure retention period for assets (default 90 days)
- Enable/disable paid upgrades later

## Notifications

- New job created
- Job status update
- Job failure alert
- Asset download available

## Permissions & privacy

- User data is stored for 90 days by default
- Only the requesting user can access their assets
- Admin has read-only access to job metadata for reporting

## Edge cases

- User submits incomplete or ambiguous brief
- Design generation fails due to technical issues
- User requests multiple revisions beyond allowed scope
- Asset download links expire or become invalid

## Required tests

- End-to-end test of design request to final delivery
- Test revision flow with updated previews
- Verify admin notifications for new jobs and failures

## Assumptions

- Users will provide clear enough briefs for design generation
- Design generation system will be reliable enough for production use
- Storage costs will be manageable with 90-day retention
