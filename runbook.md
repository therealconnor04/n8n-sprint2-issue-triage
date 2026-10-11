# Sprint 2 Runbook

## Purpose

This runbook describes how to operate, verify, recover, and safely shut down the Sprint 2 GitHub Issue Triage workflow.

The workflow receives GitHub issue events through GitHub CLI webhook forwarding, accepts only newly opened issues, classifies the issue title, stores issue information in a persistent ledger, and writes a category-specific internal route-log entry.

The supported categories and routes are:

- `bug` → `bug-triage`
- `feature` → `feature-requests`
- `help` → `support`
- `other` → `general-triage`

Evidence of workflow behavior is available in `test_evidence.md` and the `evidence/` folder.

---

## Normal operation

1. Start the local n8n environment and open:

   `http://localhost:5678`

2. Open the workflow:

   `Sprint 2 - GitHub Issue Triage`

3. Confirm that the workflow is published and active.

4. Open PowerShell and start GitHub CLI webhook forwarding:

   `gh webhook forward --repo=therealconnor04/n8n-sprint2-issue-triage --events=issues --url=http://localhost:5678/webhook/sprint2-issue-triage`

5. Leave the PowerShell window running while testing GitHub issues.

6. Create a new issue in the public GitHub repository.

7. Check the n8n Executions page and confirm that the issue follows the expected classification route.

8. Verify that the issue appears in `github_issue_ledger`.

9. Verify that a matching route entry appears in `github_issue_route_log`.

### Expected result

A newly opened issue should create one ledger row and one matching route-log row with `route_log_status = recorded`.

Evidence:

- `test_evidence.md`
- `evidence/sprint2screenshot1_bugroute.png`
- `evidence/sprint2screenshot2_featureroute.png`
- `evidence/sprint2screenshot3_helproute.png`
- `evidence/sprint2screenshot4_otherroute.png`
- `evidence/sprint2screenshot5_ledgertable1.png`
- `evidence/sprint2screenshot6_ledgertable2.png`
- `evidence/sprint2screenshot7_routelogtable.png`

---

## Duplicate delivery

The workflow uses `issue_key` to determine whether an issue has already been processed.

If the same issue is received again:

1. The workflow checks `github_issue_ledger` for the existing `issue_key`.
2. If the row already exists, the workflow follows the existing-row path.
3. The `last_seen_at` field is updated.
4. No second ledger row is created.
5. No second route-log row is created.

### Expected result

The duplicate event should follow:

`Set route → If ledger row exists → Update existing last seen`

The workflow should not continue through `Insert pending ledger row` or `Record internal route`.

During testing, GitHub's built-in webhook Redeliver action returned `401 Unauthorized` before the event reached n8n. To verify the workflow's duplicate-handling logic, the same issue payload was replayed directly to the production n8n webhook. The duplicate path worked correctly.

Evidence:

- `evidence/sprint2screenshot9_duplicatetest1.png`
- `evidence/sprint2screenshot10_duplicatetest2.png`
- `evidence/sprint2screenshot11_duplicatetest3.png`
- `evidence/sprint2screenshot12_githubredeliverfail1.png`
- `evidence/sprint2screenshot13_githubredeliverfail2.png`

---

## Route-log failure

The workflow contains an intentional failure test for verifying failure handling.

An issue with the title:

`Documentation: intentional route-log failure test`

is routed through the failure simulation path.

The workflow:

1. Inserts the issue into `github_issue_ledger`.
2. Detects the intentional failure-test phrase.
3. Runs `Simulate route-log failure`.
4. Sends the error output to `Update route-log failed`.
5. Updates the ledger row to `route_log_status = failed`.
6. Stores a readable message in `error_note`.
7. Does not create a route-log row for the failed write.

### Expected result

The ledger row remains available for investigation and shows:

- `route_log_status = failed`
- `route_name = general-triage`
- a readable `error_note`

No new row should appear in `github_issue_route_log` for the failed issue.

Evidence:

- `evidence/sprint2screenshot14_intentionalfailuretest1.png`
- `evidence/sprint2screenshot15_intentionalfailuretest2.png`
- `evidence/sprint2screenshot16_intentionalfailuretest3.png`

---

## Recovery action

If a normal route-log write fails:

1. Open the failed execution in n8n.
2. Identify the node that produced the error.
3. Review the matching ledger row and its `error_note`.
4. Correct the node configuration or data problem.
5. Confirm that the workflow is published again after any changes.
6. Retest with a harmless course issue.
7. Verify that the ledger and route-log tables contain the expected values.

If GitHub events stop reaching n8n:

1. Confirm that n8n is running at `http://localhost:5678`.
2. Confirm that the workflow is published.
3. Check that GitHub CLI forwarding is still running.
4. Restart the forwarding command if necessary:

   `gh webhook forward --repo=therealconnor04/n8n-sprint2-issue-triage --events=issues --url=http://localhost:5678/webhook/sprint2-issue-triage`

5. Create a new harmless test issue and confirm that a new n8n execution appears.

If a duplicate delivery is suspected:

1. Check the `issue_key` in `github_issue_ledger`.
2. Confirm that only one ledger row exists for that key.
3. Check `last_seen_at` to determine whether the duplicate was processed.
4. Confirm that no second route-log row was added for the same issue.

---

## Safe shutdown

After all required evidence has been captured:

1. Export the final n8n workflow.
2. Save the private original export outside the public repository.
3. Create a submission-safe copy named `sprint2_github_issue_triage.json`.
4. Redact any credential, token, password, webhook secret, or other sensitive value before putting the JSON file in the public repository.
5. Confirm that the required documentation and evidence files have been saved.
6. Deactivate the GitHub issue workflow in n8n.
7. In GitHub, remove or disable the repository webhook used for the Sprint 2 test.
8. Stop GitHub CLI forwarding by pressing `Ctrl+C` in the PowerShell or Terminal window.
9. Confirm that the forwarding process has stopped and no new GitHub issue events are being sent to the local n8n webhook.
10. Record the shutdown action in this runbook or in the final project notes.

This shutdown prevents the temporary course workflow from continuing to process events after testing is complete.

---

## What this prototype does not test

This prototype does not test:

- production-scale issue traffic
- external notification services
- real client or production repository data
- long-term uptime or high availability
- advanced authentication or secret rotation
- automatic recovery from every possible failure
- human approval or escalation workflows
- large-scale performance or load testing
- production monitoring or alerting
- multi-user access control

The prototype is limited to a public course repository, local n8n execution, deterministic title classification, persistent n8n Data Tables, duplicate protection, and controlled failure handling.

The workflow is intended only as a course prototype and not as a production deployment.
