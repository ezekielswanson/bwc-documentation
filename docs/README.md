# Better World Club — solution architect pack

Current-state documentation for HubSpot portal **44020082** (`bwc`). Snapshot fetched from Design Manager on **2026-09-19**.

This pack explains the **BWC overlay**: Join form, serverless, portal, and how custom code picks one current membership on top of hapily. It is not a hapily tutorial.

## Snapshot warning

Some production code may have been edited locally and uploaded **directly to Design Manager**. This GitHub repository contains documentation only. **Collin should inspect the actual live files in HubSpot Design Manager for portal `44020082` before changing production.** BWC has no HubSpot sandbox; Stripe test mode is separate.

## 15-minute reading path

1. [01-current-state.md](./01-current-state.md) — system picture and hapily vs BWC boundary
2. [02-join-form.md](./02-join-form.md) — `join.betterworldclub.net`
3. [03-serverless.md](./03-serverless.md) — Checkout, webhooks, reconciliation endpoints
4. [04-member-portal.md](./04-member-portal.md) — `members.betterworldclub.net`
5. [05-data-and-rules.md](./05-data-and-rules.md) — fields, source of truth, winner selection
6. [06-gaps-and-ops.md](./06-gaps-and-ops.md) — non-Stripe, in-progress, unknowns

**Downloadable PDF:** [BWC Contact Subscription Status and Winner Logic](./assets/bwc-contact-subscription-status.pdf?raw=1)

Status tags used throughout:

| Tag | Meaning |
| --- | --- |
| **Confirmed** | Verified in this snapshot, with a file citation |
| **Assumed** | Inferred; not proven in this snapshot |
| **Unknown** | Not found here; needs portal, Notion, or sibling-repo inspection |

## Local folders

| Folder | Role |
| --- | --- |
| `../bwc-quote-form/` | Join / renewal React module |
| `../cms-webpack-serverless-boilerplate/` | HubSpot serverless (`/_hcms/api/...`) |
| `../Spark-copy/` | Membership portal theme |

