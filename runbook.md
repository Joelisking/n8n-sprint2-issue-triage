# Runbook: GitHub Issue Triage workflow

## Purpose

Keep one ledger row per GitHub issue and one internal route-log row per new issue, and make any route-log failure visible. Use this when the workflow is running on my machine (n8n at http://localhost:5678, GitHub CLI forwarding events to it).

## Normal operation

1. Confirm n8n is up (http://localhost:5678 loads) and the workflow `Sprint 2 - GitHub Issue Triage` shows Published.
2. In a separate terminal, run `gh webhook forward --repo=Joelisking/n8n-sprint2-issue-triage --events=issues --url=http://localhost:5678/webhook/sprint2-issue-triage` and leave it open. A waiting terminal with no new prompt is normal.
3. Open an issue in the repository.
4. Expected result: one new execution in the workflow's Executions tab, one new `github_issue_ledger` row with `route_log_status = recorded`, and one new `github_issue_route_log` row with the required route name.

Evidence: `test_evidence.md`, tests 1 to 4.

## Duplicate delivery

1. A second delivery of an already known issue reaches the webhook (same `issue_key`).
2. Expected result: the path **If ledger row exists** runs, only `last_seen_at` changes on the existing ledger row, and no new ledger row or route-log row appears. The path **If ledger row does not exist** does not run.
3. To test it, send the same issue fields again to http://localhost:5678/webhook/sprint2-issue-triage. GitHub's own Redeliver button returns 401 through the forwarder, so it cannot be used here.

Evidence: `test_evidence.md`, test 5 (execution 24).

## Route-log failure

1. The ledger row is inserted as `pending` first. If **Record internal route** fails, its error output goes to **Update route-log failed**.
2. Expected result: the ledger row stays and shows `route_log_status = failed` with the safe `error_note` "Route-log write failed; review the route-log node configuration and execution details." No route-log row exists for that `issue_key`.
3. Find failures by filtering the ledger on `route_log_status = failed`, then open the matching execution to read the node error.
4. The failure path can be rehearsed on demand by opening an issue whose title contains `intentional route-log failure test`.

Evidence: `test_evidence.md`, test 7 (execution 26).

## Recovery action

1. Open the failed execution and read the error on **Record internal route**. Fix the cause (for example the data table selection or a column name).
2. A replay of the delivery will not repair the record, because the ledger row already exists and the replay only updates `last_seen_at`. Repair it by hand instead.
3. In `github_issue_route_log`, add a row with the same `issue_key`, `category`, `route_name` and the current time in `recorded_at`.
4. In `github_issue_ledger`, set `route_log_status` to `recorded` and clear `error_note`.
5. Expected result: one ledger row and one route-log row for the issue, with the ledger showing `recorded`.

## Safe shutdown

1. Press Ctrl+C in the terminal running `gh webhook forward`. GitHub removes its temporary CLI webhook when the client disconnects.
2. In n8n, open the workflow and unpublish it so its production webhook no longer accepts requests.
3. In GitHub, open the repository's Settings > Webhooks and confirm no CLI webhook remains.
4. Stop n8n with `docker compose stop n8n` if it is no longer needed. Do not run `docker compose down -v`, because that deletes the Data Tables.
5. Expected result: new issues no longer create executions or rows. Existing table data is kept.

## What this prototype does not test

- Two deliveries of the same new issue arriving at the same moment. Both could pass **If ledger row does not exist** before either insert finishes.
- A real public webhook URL, uptime, or GitHub's own redelivery (the forwarder returned 401 for redeliveries).
- Non-text issue content, very long titles, issues in other repositories, or high volume.
- Backup and restore of the Data Tables.
- Authentication on the webhook endpoint: the production path accepts any POST, so it must stay on localhost.
