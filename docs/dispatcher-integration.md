# Dispatcher ↔ Tasker integration contract (development only)

## Current state
Tasker stores tasks in browser-local storage and attachments in IndexedDB. It has no verified cloud identity or API. The existing browser data must remain untouched until backup and migration tests pass. This branch does not alter production Tasker.

## Required storage upgrade
- Authenticate users; authorize access by LeoNet organization and project.
- Move task/project state into a shared database with per-project membership rules.
- Move binary attachments to object storage; keep metadata, hash, owner and retention class in database.
- Preserve localStorage `leonet-tasker-v1` as a migration source only, not a live synchronization source.
- Provide a verified export, import dry-run, rollback, and idempotent migration path before cutover.

## Versioned API
All operations must authenticate callers; never expose API keys in the GitHub Pages client.
- POST /api/v1/tasks/upsert: {externalId, projectId, title, description, assignee, status, revision, idempotencyKey}
- GET /api/v1/tasks/changes?cursor=...: {changes:[{externalId, status, revision, updatedAt}], nextCursor}
- POST /api/v1/events: signed status updates delivered to Dispatcher; retries allowed, deduplicate eventId.
- Statuses: queued, in_progress, blocked, completed.
- Conflicts: reject stale revisions and return latest record. Never silently overwrite a human edit.
- On error: durable retry queue with last error and next attempt; no false "synced" status.

## Dispatcher mapping
Dispatcher work_assignments.id is externalId. tasker_external_id stores the remote Tasker ID. sync_status distinguishes not_connected, pending, synced, failed. workflow_events records every transition.

## Security and release
No customer data, real tokens, or personal information in test fixtures. No public webhook until authentication and authorization are implemented. Do not deploy to GitHub Pages or replace user data without explicit review.
