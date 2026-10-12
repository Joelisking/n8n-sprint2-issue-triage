# Test evidence

Environment: n8n 2.36.9 at http://localhost:5678, workflow `Sprint 2 - GitHub Issue Triage` (published), GitHub CLI `gh webhook forward` running as account Joelisking. All times are 10/11/2026, US Eastern (UTC-04:00). Issue keys are shown as `#N` for `Joelisking/n8n-sprint2-issue-triage#N`.

Evidence is given as execution IDs from the workflow's Executions tab and as the table contents read from n8n Data Tables after each test.

## Test script results

| Test | What I did | Issue | n8n execution | Result |
| --- | --- | --- | --- | --- |
| Setup check | Started the forward; GitHub sent a `ping` event | none | 19 (15:56:20) | Stopped at **Opened action?**; no rows written |
| 1. Bug | Created issue `Bug: blank screen after logout` | #1 | 20 (16:02:52) | Pass: category bug, route bug-triage |
| 2. Feature | Created issue `Feature request: export task list to CSV` | #2 | 21 (16:03:24) | Pass: category feature, route feature-requests |
| 3. Help | Created issue `Help: change the public webhook address` | #3 | 22 (16:03:38) | Pass: category help, route support |
| 4. Other | Created issue `Documentation: clarify local setup` | #4 | 23 (16:03:51) | Pass: category other, route general-triage |
| 5. Redelivery of test 1 | See the note below | #1 | 24 (16:14:13) | Pass: one ledger row, `last_seen_at` updated, no second route-log row |
| 6. Opened-action gate | Appended the course sentence to the body of issue #4 | #4 | 25 (16:15:19) | Pass: event `issues.edited` stopped at the gate; no new or changed rows |
| 7. Route-log failure | Created issue `Documentation: intentional route-log failure test` | #5 | 26 (16:15:40) | Pass: ledger row `failed` with safe `error_note`; no route-log row |

## Ledger after tests 1 to 4 (`github_issue_ledger`)

| issue_key | title | category | route_name | route_log_status | last_seen_at | error_note |
| --- | --- | --- | --- | --- | --- | --- |
| #1 | Bug: blank screen after logout | bug | bug-triage | recorded | 16:02:52.850 | (blank) |
| #2 | Feature request: export task list to CSV | feature | feature-requests | recorded | 16:03:24.607 | (blank) |
| #3 | Help: change the public webhook address | help | support | recorded | 16:03:38.889 | (blank) |
| #4 | Documentation: clarify local setup | other | general-triage | recorded | 16:03:51.393 | (blank) |

## Route log after tests 1 to 4 (`github_issue_route_log`)

| issue_key | category | route_name | recorded_at |
| --- | --- | --- | --- |
| #1 | bug | bug-triage | 16:02:52.850 |
| #2 | feature | feature-requests | 16:03:24.607 |
| #3 | help | support | 16:03:38.889 |
| #4 | other | general-triage | 16:03:51.393 |

Four ledger rows and four route-log rows, one per issue, each with the required route name.

## Test 5: redelivery (duplicate delivery of issue #1)

What happened with GitHub's button: in Settings > Webhooks > Recent deliveries I opened delivery `bd9640ea-c5ae-11f1-8441-6b265c121f76` (issue #1, `issues.opened`) and clicked Redeliver. GitHub listed the redelivery at 16:12:58 with a red warning. Its response was **401**, the request URL was `https://webhook-forwarder.github.com/hook`, and n8n logged no execution (the latest execution stayed at 23). I did not find the cause. Other events (the edit and the failure issue) still reached n8n, so the workflow and the forward were working.

Stand-in used: from the n8n browser tab I sent a POST to `http://localhost:5678/webhook/sprint2-issue-triage` with the same delivery ID header and the fields the workflow reads (`action`, `repository.full_name`, `issue.number`, `issue.html_url`, `issue.title`, `issue.user.login`, `issue.created_at`) taken from the original delivery. The response was `200 {"message":"Workflow was started"}` and execution 24 ran.

Result:

| Check | Before (16:12) | After (16:14) |
| --- | --- | --- |
| Ledger rows for `#1` | 1 | 1 |
| `last_seen_at` of `#1` | 16:02:52.850 | 16:14:13.904 |
| Route-log rows for `#1` (bug-triage) | 1 | 1 |
| Total route-log rows | 4 | 4 |

The original `bug-triage` route-log row (recorded 16:02:52.850) was unchanged and no second one appeared.

## Test 6: opened-action gate

At 16:15 I edited the body of issue #4 and saved. GitHub delivered `issues.edited` (delivery `7a698e10-c5b0-11f1-97b9-7979e31ed55e`) and n8n ran execution 25, which ended at **Opened action?**. Afterwards the route log still had 4 rows and `#4` kept `last_seen_at` 16:03:51.393 in the ledger.

## Test 7: intentional route-log failure (issue #5)

Issue #5 `Documentation: intentional route-log failure test` produced execution 26. The title matched **Intentional failure test?**, **Simulate route-log failure** threw its error, and the error output went to **Update route-log failed**.

| issue_key | category | route_name | route_log_status | error_note |
| --- | --- | --- | --- | --- |
| #5 | other | general-triage | failed | Route-log write failed; review the route-log node configuration and execution details. |

`github_issue_route_log` still had 4 rows and none for `#5`. The ledger now has 5 rows. The simulation nodes were left in place.

## Not shown

Screenshots are in the `evidence/` folder and linked from `README.md`. The execution IDs above can be opened in the local n8n instance, and the `issues` deliveries are listed under Settings > Webhooks while the forward is running.
