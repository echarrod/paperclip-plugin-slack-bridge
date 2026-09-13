# Changelog

All notable changes to this project are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning: [SemVer](https://semver.org/).

## [Unreleased]

### Fixed
- The human-loop poller now derives approval candidacy from `GET /api/companies/{id}/approvals?status=pending` in addition to the issue-attention scan, so a pending approval is no longer invisible to the once-a-minute backstop when its linked issue is `done`/`cancelled`, when a higher-priority attention branch has already claimed the issue's single attention slot, or when a second approval shares an issue with one already surfaced by that slot. `slack_human_loop_poll_completed` now also reports `pending_approvals_seen` and `pending_approvals_recovered`, so a healthy idle bridge (`dispatched: 0`) can be told apart from one where pending approvals are silently going uncovered. (#24)
- `approval.decided` events are now normalized instead of being dropped, so approvals resolved in Paperclip (rather than from a Slack button) reach Slack at all. The event handler was registered in the worker but `normalizeEvent` had no case for it, so every decision fell through to `default: return null`.
- `approval.decided` events are now enriched from the approval record before rendering (the same path `approval.created` already used, renamed `approvalCreatedEventFromApi` -> `approvalEventFromApi`), so the resolved card shows the real decision status rather than depending on what the raw event payload carries. (The decision note stays out of the card itself, matching the compact resolved receipt introduced in 0.1.1.)
- A decided approval now edits the original approval card in place (`chat.update`) instead of posting a second message, using a new per-approval message ref in plugin state. Approvals whose card predates that state, or whose edit Slack rejects, fall back to posting a new message.
- Approval cards always post to the approvals channel (or the default channel) instead of replying in the linked issue's Slack thread. On a long-lived issue every approval landed as a reply under the first card ever posted for it, invisible in the channel's history. Approval cards also no longer become an issue's thread, which pulled that issue's later notifications - run failures included - into the approvals channel. Other notification kinds still thread under the issue.

## [0.1.1] - 2026-07-12

### Changed
- Human-in-the-loop confirmation Slack cards now use a two-stage send-back flow when a decline reason is required, preserving custom action labels and showing the notes input only after the send-back action is chosen.
- Resolved approval and confirmation Slack cards now collapse to compact receipt-style messages with a Paperclip link instead of repeating full decision details in Slack.

### Fixed
- Checkbox confirmations that require decline reasons now support the same staged Slack decline flow as regular confirmations.
- Slack interactive button payloads are bounded to stay within Slack value-size limits.
- Test fixtures no longer use real-looking Slack token shapes.

## [0.1.0] - 2026-07-01

Initial public release.

### Added
- Paperclip event notifications rendered as deterministic Slack Block Kit cards
  (`approval.created`, `approval.decided`, agent run failures/finishes, issue updates —
  each individually toggleable in plugin config).
- Human-in-the-loop approval flow: approve/reject Paperclip approvals directly from Slack.
- Human-input-needed polling: agents waiting on a human surface as Slack cards.
- `/paperclip` (and `/clip`) slash commands: status, companies, issues, issue detail, issue creation (`issues.create`), and agent wakeup (`issues.wakeup`).
- Slack Socket Mode ingress only — no public URL, webhooks, or signing secret required.
- Slack app manifests (`slack-app-manifest.socket-mode.{json,yaml}`) for one-click app creation.
