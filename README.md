# Better World Club — Documentation Repository

Client handoff package for documenting the BWC member lifecycle, HubSpot/Stripe/hapily integration, and related CMS assets.

## Repository path

```
/Users/zeke/Desktop/projects/client_projects/betterworld/bwc_full_repo_for_documentation
```

## HubSpot portal

| Field | Value |
| --- | --- |
| Portal name | `bwc` |
| Portal ID | `44020082` |
| Sandbox | `bwc_sandbox` (`51439185`) |

All HubSpot CLI and API work for this documentation effort must target portal **44020082**, not the machine default account.

```bash
hs accounts use 44020082   # or: --account 44020082 on individual commands
```

## What is in this folder

This repo is a **snapshot** fetched from HubSpot Design Manager on 2026-09-19:

| Local folder | Design Manager source | Role |
| --- | --- | --- |
| `bwc-quote-form/` | `bwc-quote-form` | Join / quote React form (`join.betterworldclub.net`) |
| `cms-webpack-serverless-boilerplate/` | `cms-webpack-serverless-boilerplate` | Serverless functions (checkout, webhooks, mapping, renewal) |
| `Spark-copy/` | `Spark copy` | Membership portal theme and modules (`members.betterworldclub.net`) |

## Start here

Solution architects: start at **[docs/README.md](./docs/README.md)**.

Internal assignment notes: [docs/TASK_HANDOFF.md](./docs/TASK_HANDOFF.md), [docs/TOOLS_AND_SETUP.md](./docs/TOOLS_AND_SETUP.md).

## Related workspace folders

Additional logic, scripts, runbooks, and historical context may exist outside this folder:

| Path | Notes |
| --- | --- |
| `../bwc_repo_webhook_final/` | Webhooks, membership reconciliation docs, `AGENTS.md` |
| `../bwc_repo_membership portal/` | Earlier membership portal work |
| `../bwc_login_page/` | Login page modules |
| `../bwc-production-migration/` | Migration and import scripts |

## Client documentation pack

```
docs/
├── README.md                 ← start here
├── 01-current-state.md
├── 02-join-form.md
├── 03-serverless.md
├── 04-member-portal.md
├── 05-data-and-rules.md
├── 06-gaps-and-ops.md
├── TASK_HANDOFF.md           ← internal assignment
└── TOOLS_AND_SETUP.md        ← internal tooling
```
