# Sprint 2 — GitHub Issue Triage

Public repository:  
https://github.com/therealconnor04/n8n-sprint2-issue-triage

## Purpose

This project is a local n8n workflow that receives GitHub issue events, accepts only newly opened issues, classifies each issue by its title, prevents duplicate processing, stores issue data in a persistent ledger, and records an internal route-log entry.

The workflow uses deterministic title rules rather than an LLM or external AI service.

## Services and Tools Used

- GitHub
- GitHub Issues
- GitHub CLI
- n8n self-hosted locally
- n8n Webhook node
- n8n Data Tables
- PowerShell
- Zapier for the Own vs. Rent comparison

n8n runs locally at:

`http://localhost:5678`

## Workflow Sequence

The workflow is named:

`Sprint 2 - GitHub Issue Triage`

Main node sequence:

1. `GitHub issue webhook`
2. `Opened action?`
3. `Normalize issue`
4. `Classify title`
5. Route-setting node:
   - `Set bug route`
   - `Set feature route`
   - `Set help route`
   - `Set other route`
6. Duplicate checks:
   - `If ledger row exists`
   - `If ledger row does not exist`
7. Existing issue path:
   - `Update existing last seen`
8. New issue path:
   - `Insert pending ledger row`
9. `Intentional failure test?`
10. Normal path:
    - `Record internal route`
    - `Update route-log recorded`
11. Failure path:
    - `Simulate route-log failure`
    - `Update route-log failed`

## Field Mapping

The `Normalize issue` node maps the GitHub webhook payload into the fields used by the rest of the workflow.

| Field | Expression |
| --- | --- |
| `repository_full_name` | `{{ $json.body.repository.full_name }}` |
| `issue_number` | `{{ $json.body.issue.number }}` |
| `issue_key` | `{{ $json.body.repository.full_name + "#" + $json.body.issue.number }}` |
| `issue_url` | `{{ $json.body.issue.html_url }}` |
| `title` | `{{ $json.body.issue.title }}` |
| `title_lower` | `{{ $json.body.issue.title.toLowerCase() }}` |
| `reporter` | `{{ $json.body.issue.user.login }}` |
| `opened_at` | `{{ $json.body.issue.created_at }}` |
| `last_seen_at` | `{{ $now.toISO() }}` |

The `issue_key` is used as the stable duplicate-prevention key.

## Classification Rules

The `Classify title` Switch node evaluates the lowercase issue title in this order:

| Order | Match | Category | Route |
| --- | --- | --- | --- |
| 1 | `\\b(bug|error)\\b` | `bug` | `bug-triage` |
| 2 | `\\b(feature|request)\\b` | `feature` | `feature-requests` |
| 3 | `\\b(help|question)\\b` | `help` | `support` |
| 4 | Fallback | `other` | `general-triage` |

Each issue receives only one category and one route.

## Data Tables

The workflow uses two persistent n8n Data Tables.

### `github_issue_ledger`

Stores the main issue record and duplicate/failure state.

Fields include:

- `issue_key`
- `issue_url`
- `title`
- `category`
- `reporter`
- `opened_at`
- `last_seen_at`
- `route_log_status`
- `route_name`
- `error_note`

### `github_issue_route_log`

Stores successful routing results.

Fields include:

- `issue_key`
- `category`
- `route_name`
- `recorded_at`

## Setup Notes

Start n8n locally and open:

`http://localhost:5678`

Then start GitHub CLI forwarding in PowerShell:

`gh webhook forward --repo=therealconnor04/n8n-sprint2-issue-triage --events=issues --url=http://localhost:5678/webhook/sprint2-issue-triage`

The workflow must be published/active while testing.

Only GitHub issue events with:

`body.action = opened`

continue into the triage workflow.

## Evidence and Documentation

- [`test_evidence.md`](test_evidence.md) — test results and screenshots
- [`runbook.md`](runbook.md) — normal operation, duplicate handling, recovery, failure handling, and shutdown
- [`own_vs_rent.md`](own_vs_rent.md) — n8n vs. Zapier comparison
- [`journals/week-06.md`](journals/week-06.md) — Week 6 Build Journal
- [`sprint2_github_issue_triage.json`](sprint2_github_issue_triage.json) — submission-safe workflow export
- [`evidence/`](evidence/) — screenshots used for verification

## Security Note

This repository is public. The workflow JSON committed here is a submission-safe copy and should not contain credentials, tokens, passwords, webhook secrets, or other sensitive values.

The original working n8n export should remain private.
