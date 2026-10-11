# Own vs. Rent Comparison

## Assumptions

For this comparison, I assume a small product team processes about **250 newly opened GitHub issues per month**. I also assume that route-log failures are uncommon, occurring only a few times per month, and that the workflow may need small rule or routing changes several times during a semester or project cycle.

The self-hosted option is the n8n workflow built for this sprint. The rented option is Zapier using a GitHub issue-created trigger followed by Formatter by Zapier to convert the issue title to lowercase.

## Zapier Comparison Exercise Notes

The required Zapier comparison was completed with two conceptual steps:

1. GitHub **New Issue** trigger
2. **Formatter by Zapier** text step that converts the issue title to lowercase

Observed test values:

- **Original issue title:** `Documentation: intentional route-log failure test 2`
- **Lowercased Formatter output:** `documentation: intentional route-log failure test 2`
- **Setup friction or permissions observed:** Zapier required connecting the GitHub account, selecting the repository, testing the GitHub trigger, and mapping the issue title into the Formatter step.
- **Free-tier limit or prompt encountered:** No free-tier limitation or payment prompt prevented completion of the required two-step test.

## Comparison Table

| Dimension | Self-hosted n8n (Own) | Zapier (Rent) | Evidence or assumption |
| --- | --- | --- | --- |
| Initial setup time | Higher because the local stack, n8n workflow, Data Tables, GitHub CLI forwarding, and routing logic had to be configured manually. | Lower for the small comparison because the GitHub trigger and Formatter step were configured through a hosted interface. | Based on the Sprint 2 build experience and the completed Zapier test. |
| Public webhook and credential setup | Required GitHub CLI forwarding from GitHub to the local n8n webhook. | GitHub connection was handled through Zapier's hosted integration and account authorization. | Observed directly during setup. |
| Ability to add branching | Strong control through IF, Switch, Set, Code, and Data Table nodes. | Branching can be added through more Zapier steps, but the required comparison only used two steps. | n8n evidence comes from the completed workflow; Zapier scope was intentionally limited. |
| Duplicate prevention and ledger | Custom `issue_key`, ledger lookup, `last_seen_at`, and route-log logic provide explicit idempotency. | The two-step Zapier comparison did not include a persistent duplicate ledger. | Based on the implemented n8n design and the limited Zapier exercise. |
| Failure visibility and recovery | Execution history, failed ledger status, `error_note`, and a controlled failure path make failures visible and recoverable. | Zapier provides hosted task history, but the class comparison did not implement the same custom ledger-based recovery design. | Based on the tested n8n failure path and the limited Zapier test. |
| Monthly volume assumption | 250 newly opened issues per month. | 250 newly opened issues per month. | Shared assumption for comparison. |
| Direct platform cost | No additional n8n subscription was required for this local course setup, but the user is responsible for the computer, storage, and operating time. | Direct cost depends on the Zapier plan and task limits. No exact paid-plan price is assumed here. | Avoids unsupported pricing claims; no payment information was entered. |
| Maintenance, security, and uptime responsibility | Mostly the user's responsibility because n8n is self-hosted locally. | More infrastructure responsibility is handled by the SaaS provider. | General ownership difference between self-hosting and hosted SaaS. |
| Data location and access considerations | Workflow data and Data Tables remain in the locally controlled n8n environment. | Workflow data is processed through an external hosted service connected to GitHub. | Based on the architecture of the two approaches. |

## Analysis

The self-hosted n8n approach required more setup work, but it also gave me much more control over the workflow. I had to configure the local n8n environment, GitHub CLI forwarding, classification rules, Data Tables, duplicate handling, and a controlled route-log failure path. That increased the initial setup time, but it also made the design very transparent. I could see exactly where the data was stored, how each category was routed, and how duplicate events were prevented from creating duplicate records.

Zapier represents the rent side of the comparison because the automation platform is hosted and much of the infrastructure is managed for the user. In the required comparison, I connected GitHub, selected the repository, tested the New Issue trigger, and then mapped the issue title into Formatter by Zapier. The Formatter successfully changed `Documentation: intentional route-log failure test 2` to `documentation: intentional route-log failure test 2`. This was quicker to configure than the full n8n workflow and required less local setup.

The biggest difference is control versus convenience. The n8n version includes persistent ledger storage, duplicate prevention with `issue_key`, category-specific routing, a route-log table, and explicit failure handling with `route_log_status` and `error_note`. The Zapier test was simpler, but the two-step exercise did not reproduce those deeper workflow controls.

For a small product team handling about 250 new issues per month, I think the better choice depends on the team's priorities. Zapier is attractive if the team wants quick setup and less infrastructure responsibility. Self-hosted n8n is stronger if the team needs more control over routing, data storage, duplicate handling, and recovery logic.

For this specific prototype, I would choose **self-hosted n8n**. The workflow already demonstrates custom classification, persistent Data Tables, idempotency, and failure handling. The tradeoff is that the team must accept more responsibility for maintenance, security, and uptime.
