# n8n Sprint 2: GitHub Issue Triage

Repository: https://github.com/Joelisking/n8n-sprint2-issue-triage (public)

Author: Joel Adu-Kwarteng, CS 59300 Low-Code No-Code Rapid Prototyping, Fall 2026

## Purpose

A self-hosted, event-driven n8n workflow that receives newly opened GitHub issues from this repository, classifies each one from its title with fixed rules, saves a durable ledger row, writes one category-specific route-log entry, and handles a duplicate delivery without creating a second record or a second route-log entry. A failed route-log write leaves a visible `failed` ledger row instead of disappearing.

## Services

| Service | Role | Where it runs |
| --- | --- | --- |
| GitHub (this public repo) | Emits `issues` webhook events | github.com |
| GitHub CLI `gh webhook forward` | Receives the events on GitHub's side and forwards them to localhost while the command runs | My computer |
| n8n (self-hosted, Docker) | Runs the workflow | http://localhost:5678 |
| n8n Data Tables | `github_issue_ledger` and `github_issue_route_log` | Inside my n8n instance |

No GitHub credential, outside notification service, tunnel or public domain is used. The workflow file is `sprint2_github_issue_triage.json`.

## Node sequence

1. **GitHub issue webhook** (Webhook, POST, path `sprint2-issue-triage`)
2. **Opened action?** (IF, `body.action` equals `opened`; only the true output is connected, so edits, closes and the webhook ping stop here)
3. **Normalize issue** (Edit Fields)
4. **Classify title** (Switch with four outputs)
5. **Set bug route**, **Set feature route**, **Set help route**, **Set other route** (Edit Fields, fixed `category` and `route_name`)
6. Each route node feeds two parallel checks:
   - **If ledger row exists** then **Update existing last seen** (updates only `last_seen_at`; the path ends here)
   - **If ledger row does not exist** then **Insert pending ledger row** then **Intentional failure test?**
7. **Intentional failure test?** (IF, title contains `intentional route-log failure test`)
   - false: **Record internal route** (inserts the route-log row, On Error: continue using error output) then **Update route-log recorded**
   - true: **Simulate route-log failure** (Code node that throws, On Error: continue using error output)
   - Both error outputs go to **Update route-log failed**

## Field mapping

| Normalized field | Source in the webhook body |
| --- | --- |
| `repository_full_name` | `body.repository.full_name` |
| `issue_number` | `body.issue.number` |
| `issue_key` | `repository.full_name + "#" + issue.number` |
| `issue_url` | `body.issue.html_url` |
| `title` | `body.issue.title` |
| `title_lower` | `body.issue.title.toLowerCase()` |
| `reporter` | `body.issue.user.login` |
| `opened_at` | `body.issue.created_at` |
| `last_seen_at` | `$now.toISO()` at processing time |

`github_issue_ledger` columns: `issue_key`, `issue_url`, `title`, `category`, `reporter`, `opened_at`, `last_seen_at`, `route_log_status` (pending, recorded or failed), `route_name`, `error_note`.
`github_issue_route_log` columns: `issue_key`, `category`, `route_name`, `recorded_at`.

## Classification rules

The rules run in order against the lowercased title. There is no LLM involved.

| Order | Regex (case folded) | Category | Route name |
| --- | --- | --- | --- |
| 1 | `\b(bug\|error)\b` | bug | bug-triage |
| 2 | `\b(feature\|request)\b` | feature | feature-requests |
| 3 | `\b(help\|question)\b` | help | support |
| 4 | fallback | other | general-triage |

## Setup notes

1. Start n8n with Docker (`docker compose up -d n8n` in the course stack folder) and open http://localhost:5678.
2. Create the two Data Tables with the columns above, then build and publish the workflow.
3. Log in to GitHub CLI as the repository owner (`gh auth switch --user Joelisking` if several accounts are logged in) and install the extension once: `gh extension install cli/gh-webhook`.
4. Start forwarding and leave the window open:
   `gh webhook forward --repo=Joelisking/n8n-sprint2-issue-triage --events=issues --url=http://localhost:5678/webhook/sprint2-issue-triage`
5. Create issues in this repository. Stop with Ctrl+C when finished (see `runbook.md`).

Known limit found during testing: GitHub's **Redeliver** button returns 401 when the target is the `gh webhook forward` endpoint, so the duplicate-delivery test was done by replaying the same payload fields to the local webhook. Details are in `test_evidence.md`.

## Evidence and other files

- `test_evidence.md`: the five issues, the duplicate delivery, the opened-only gate and the failure test, with execution IDs and table contents
- `runbook.md`: how to operate, recover and shut down the workflow
- `own_vs_rent.md`: comparison with Zapier
- `journals/week-06.md`: Week 6 build journal with the extensibility observation
- `sprint2_github_issue_triage.json`: submission-safe export of the workflow
