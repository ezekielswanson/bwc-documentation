# Task handoff: Better World Club system documentation

**Audience:** Collin / documentation reviewer  
**Assignee:** Cursor agent (fresh context)  
**Repository:** `bwc_full_repo_for_documentation`  
**HubSpot portal:** `44020082` (`bwc`)  
**Date prepared:** 2026-09-19

---

## Orientation

> This is a **business and architecture orientation** for reviewing the repository — not a claim that every implementation detail below is current. Trace live code, configuration, workflows, and environment variables. Record any differences you find.

---

## Original task notes

- Get docs created for **Collin**.
- Start with the **normal flow** and **highlight Niko's code**.
- Document **non-Stripe subscriptions** and how they are handled.
- Get all business logic once **Incorrect merges - 09/08/2026 - Post merge logic & New Billing End Date Logic Update** is complete.
- Document **unknowns**, including batch Stripe subscription imports for vendors.
- Document **in-progress work** and callouts:
  - Add-to-Apple-Wallet functionality for the member page
  - Print member cards
  - GreenRope API emails
- **Repository warning:** repo access/currentness is uncertain. Some code may have been updated locally and pushed directly into HubSpot Design Manager.
- Reference the **BWC Notion pages** and this repo in Cursor.

---

## Objective

Document the complete member lifecycle and the code/assets that support it:

**User sign-up → Stripe subscription purchase → HubSpot Contact and hapily Subscription synchronization → portal access → renewal/cancellation → ongoing status reconciliation**

The core business requirement: HubSpot must expose **one trustworthy, current membership state per person** even when Stripe, hapily, legacy imports, aliases, or multiple historical subscription records disagree.

---

## System roles

| System | Primary responsibility |
| --- | --- |
| Better World Club public site | Sends prospects to the Join flow |
| `join.betterworldclub.net` | HubSpot-hosted React Join form — plan selection, add-ons, discounts, member details, Stripe checkout initiation |
| Stripe | Payment, customer, Checkout Session, subscription, invoice, billing portal, renewal, payment status |
| hapily / Zaybra | Syncs Stripe billing records into HubSpot as hapily Subscription custom-object records |
| HubSpot Contact | Member identity, portal registration/access, member identifiers, contact info, denormalized current-membership fields |
| HubSpot hapily Subscription object | Stripe subscription lifecycle, billing windows, products, status, customer/subscription IDs, renewal data, payment metadata |
| `members.betterworldclub.net` | Private HubSpot membership portal — authoritative membership display and renewal/payment actions |
| HubSpot serverless / workflows / private apps | Custom orchestration where native HubSpot or hapily behavior is insufficient |

---

## Local repo mapping (starting inventory)

| Asset | Local path | Investigate first |
| --- | --- | --- |
| Join / quote form | `bwc-quote-form/` | `src/App.js`, `src/StateStore.js`, `src/components/forms/FormContainer.js`, `main.js` |
| Serverless backend | `cms-webpack-serverless-boilerplate/app.functions/` | `serverless.json`, `handleSubmit.js`, `handleRenewalSubmit.js`, `handleSuccessfulPayment.js`, mapping functions |
| Portal theme | `Spark-copy/` | `modules/custom_modules/Contact Information.module/`, `memebrship.functions/` |
| Sibling webhook repo | `../bwc_repo_webhook_final/` | Reconciliation docs, webhook handlers, `AGENTS.md` |

---

## Sign-up flow (orientation diagram)

```
┌──────────────────────────────┐
│ 1. User opens Join form     │
│ join.betterworldclub.net    │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 2. User enters information  │
│ • Contact details           │
│ • Membership plan          │
│ • Add-ons / associates     │
│ • Discounts / start date   │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 3. HubSpot serverless code  │
│ validates the submission    │
│ and builds Stripe checkout  │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 4. Stripe Checkout          │
│ • Customer pays            │
│ • Subscription is created  │
│ • Invoice/payment recorded │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 5. Stripe confirms payment  │
│ and sends webhook events    │
└──────────────┬───────────────┘
               │
        ┌──────┴─────────┐
        │                │
        ▼                ▼
┌────────────────┐  ┌────────────────────┐
│ Custom HubSpot │  │ hapily / Zaybra    │
│ automation     │  │ Stripe sync        │
│ • Join date    │  │ • Subscription CO  │
│ • Member card  │  │ • Billing/status   │
└───────┬────────┘  └─────────┬──────────┘
        │                     │
        └──────────┬──────────┘
                   │
                   ▼
┌──────────────────────────────┐
│ 6. HubSpot Contact is tied  │
│ to subscription data and    │
│ current membership state    │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 7. Member receives portal   │
│ registration/access email   │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 8. Member signs in at       │
│ members.betterworldclub.net │
│ and views their membership  │
└──────────────────────────────┘
```

---

## Normal end-to-end flow (trace and confirm)

### 1. User starts a membership

1. User opens Join form on `join.betterworldclub.net`.
2. Frontend loads active Stripe products/prices; depends on stable Stripe `lookup_key` values.
3. User provides contact details, base plan, associates/add-ons, start date, discount info.
4. Form must distinguish **net-new signup** from **authenticated renewal**. Stale login/session state has previously routed new signups into renewal logic.
5. Form submits to HubSpot serverless endpoint — likely `handleSubmit` family. **Confirm:** component, payload, route, validation, error handling.

### 2. Stripe Checkout and subscription creation

1. Backend resolves products to Stripe Price IDs.
2. Creates or resolves Stripe Customer; creates Checkout Session/subscription.
3. Stripe is payment/invoice authority — document plan, amount, discount, add-on, collection method, trial/start timing, customer metadata from code.
4. Successful Checkout redirects to thank-you page.
5. **Stripe limitation:** one Checkout Session can use a predefined discount/coupon **or** allow a promotion code, not both. Confirm current handling.

### 3. Payment completion and HubSpot enrichment

1. Stripe emits post-checkout events. Custom path listens for `checkout.session.completed` to set Contact `join_date` from event timestamp.
2. `join_date` is **write-once** — existing values must not be overwritten.
3. Member-card automation generates `member_card_no` for eligible new members. Confirm workflow trigger and whether imported/non-Stripe members use a separate path.
4. hapily/Zaybra syncs Stripe Customer/subscription into HubSpot; associates hapily Subscription with Contact.
5. Timing gap may exist between Contact creation, payment, and hapily Subscription appearance. Earlier mapping code retried up to ~30 minutes — confirm what is still active.

### 4. Subscription data becomes HubSpot member state

HubSpot can hold **multiple** associated hapily Subscription records per Contact (renewals, imports, canceled lineages, duplicate sync, corrections). Do not assume the first association is current.

**Current reconciliation pattern:**

1. Resolve the Contact from the triggering subscription and load the full associated hapily Subscription set.
2. Exclude records with populated `subscription_type`, blank `subscription_status`, blank `subscription_id`, or missing `billing_start_date` / `billing_end_date`.
3. Group eligible records by Stripe `subscription_id`.
4. Pick one representative per group by latest `billing_start_date`, then latest `hs_createdate`, then highest HubSpot object ID.
5. Pick the Contact-level standard winner using the same order.
6. If the standard winner is canceled while a competing eligible active record has `paid_through` and the case appears merge-affected or tied to a deleted Stripe lineage, do not update the Contact or move the label. Set the anomaly fields for manual review.
7. Otherwise copy `status_of_membership`, `billing_start_date`, `billing_end_date`, `products`, `coupon`, and `discount`; reconcile the **Current Subscription** label; and clear prior anomaly fields.
8. If no safe winner exists, leave Contact subscription fields and the current label unchanged; set `subscription_sync_anomaly` and `subscription_sync_anomaly_reason`.

`billing_start_date` selects the winner. `hs_createdate` and object ID are tie-breakers. `billing_end_date` and status are required/copied as applicable but do not rank the winner.
> **Current Subscription** label reflects a code decision — it is not the selection algorithm itself.

### 5. Portal registration and access

1. Eligible Contacts added to HubSpot private-content access group/list.
2. HubSpot sends/resends private-content registration email; member creates credentials.
3. Member signs in at `members.betterworldclub.net`.
4. Portal reads labeled **Current Subscription**, not first generic association.
5. HubL `crm_associations()` must explicitly request every downstream property the module renders.
6. Portal displays member info, plan, coverage dates, status, payment info; links to member-card and account management.
7. Stripe `customer_id` from authoritative subscription creates Billing Portal session.

### 6. Renewal flow

1. Logged-in member starts renewal from portal.
2. Frontend prefills Contact/subscription context; routes to renewal handler, not net-new handler.
3. Renewal code creates Stripe session or updates subscription — determine whether each branch updates existing subscription, creates new lineage, or uses Billing Portal.
4. After payment/lifecycle change, Stripe and hapily update HubSpot subscription data.
5. Reconciliation reruns; **Current Subscription** label moves if needed; Contact fields update from new winning billing window.
6. Portal renders refreshed authoritative record.

**Cancellation intent** is tracked separately (`requested_cancellation`, `requested_cancellation_date`). Member may remain active through paid-through date after requesting cancellation — intent must not immediately replace live Stripe lifecycle status.

---

## Non-Stripe and bulk-paid members

Not every covered person is individually billed in Stripe. Corporate, fleet, distributor, partner, complimentary, or manually invoiced members may bypass Stripe while still needing HubSpot records, portal access, member cards, and active coverage.

Examples from project notes: Equipment Controls, TriMeter, Pointz, League of American Bicyclists, New Wheel — exact treatment differs.

**Document whether the repo has a first-class non-Stripe path** or these members are handled via imports/scripts/manual ops:

- Coverage start/end dates and status without Stripe
- Member cards and portal access creation
- Renewal initiation
- Stripe invoice/renewal email suppression
- Participation in Current Subscription algorithm
- Authoritative source for externally paid coverage

---

## Important data-model rules

| Topic | Rule |
| --- | --- |
| Contact identity | Generally email-based; legacy alias emails (`club+<member_id>@betterworldclub.com`) represent real members |
| Stripe identity | Preserve `customer_id`, `subscription_id`, invoice IDs; distinguish live vs test |
| Membership identity | `member_id`, `member_card_no`, legacy contract IDs are separate — do not conflate |
| Subscription truth | Billing windows and selected Stripe lineage beat arbitrary record creation order |
| Many records, one view | Contact may have several historical subscriptions; portal needs one authoritative current record |
| Dates | Document join, requested/effective start, billing start/end, renewal, paid-through, trial end, legacy contract dates |
| Status | Map Stripe statuses (`active`, `trialing`, `past_due`, `unpaid`, `canceled`, etc.) to HubSpot membership status and portal display |

---

## Known edge cases to trace

- New signup treated as renewal due to authenticated Contact state
- Missing/inactive Stripe `lookup_key` → undefined product/add-on data
- Multiple subscriptions per Contact; duplicate lineages; multiple Contacts on one subscription
- Generic association order returns canceled record before active record
- Labeled association correct but returns only IDs (properties not explicitly requested)
- hapily sync delay; webhook retry/idempotency
- Legacy trial/imported billing dates → false past-due invoices
- Test Stripe Customer IDs on production HubSpot records
- Alias-email collisions, duplicate contacts, merged contacts
- Portal caching/registration; password-reset before registration
- Cross-subdomain calls `join.*` ↔ `members.*`; prefer relative `/_hcms/api/...`
- Zero-dollar/complimentary memberships triggering paid-member automations
- Stripe invoice-email suppression for externally billed groups
- Contact info changes propagating to associated subscription records
- Renewal creating new subscription record vs updating expected lineage

---

## Repository review checklist

Inventory and document:

1. React/UI entry points for Join and Renewal
2. State store; how `isRenewal` / logged-in Contact context is determined
3. Product-fetch logic; every Stripe `lookup_key` dependency
4. Checkout payload: discounts, promotion codes, add-ons, associates, VIN/green-vehicle, carbon-offset
5. All HubSpot serverless routes in `serverless.json` — method, file, secrets, caller, response
6. Stripe webhook endpoints — event types, signature verification, retries, idempotency
7. hapily-managed vs custom code — clearly separate vendor sync from repo logic
8. Contact ↔ hapily Subscription property mappings (both directions)
9. Workflows/custom-coded actions for member IDs/cards, lists/access
10. Current Subscription winner-selection and association-label updates
11. Anomaly fields, review lists, logging, recovery/backfill tools
12. Private-content registration, login, redirects, access-group enrollment
13. Stripe Billing Portal creation and return URLs
14. Renewal, cancellation, failed-payment, past-due, invoice-payment branches
15. Import/reconciliation scripts for legacy, alias, non-Stripe, externally billed members
16. Environment boundaries: local snapshot, HubSpot production Design Manager (no HubSpot sandbox), and Stripe live/test modes
17. Deployment commands; where source is authoritative vs Design Manager–only edits

---

## Expected documentation output

Create repo-grounded markdown under `docs/`:

| # | Deliverable | Target file |
| --- | --- | --- |
| 1 | Architecture summary + system-context diagram | `01-architecture.md` |
| 2 | Sequence diagrams: signup, purchase, renewal, cancellation, failed payment, non-Stripe creation | `02-signup-flow.md`, `03-renewal-cancellation.md`, `04-non-stripe-members.md` |
| 3 | Component inventory: UI, serverless, webhooks, workflows, private apps, integrations | `01-architecture.md` (section) |
| 4 | Data dictionary: critical Contact + hapily Subscription fields | `05-data-dictionary.md` |
| 5 | Source-of-truth matrix by field and lifecycle event | `05-data-dictionary.md` (section) |
| 6 | Endpoint reference with request/response examples and failure behavior | `06-endpoints.md` |
| 7 | Business-rule catalog: products, discounts, dates, statuses, winner selection, aliases, exclusions | `07-business-rules.md` |
| 8 | Operations runbook: support cases, reconciliation, imports, replay/backfill, anomalies | `08-operations-runbook.md` |
| 9 | Known gaps and open questions — confirmed vs assumed | `09-known-gaps.md` |
| 10 | Deployment and ownership notes; HubSpot-only assets | `10-deployment-notes.md` |

Each document must tag findings as:

- **Confirmed** — verified in code/config with file citation
- **Assumed** — inferred but not yet verified
- **Unknown** — not found; needs client input or live HubSpot inspection

---

## Execution order (recommended)

### Phase 1 — Normal flow (priority)

1. Trace Join form → `handleSubmit` → Stripe Checkout → webhooks → hapily sync → Contact enrichment.
2. Trace portal module → Current Subscription label → displayed fields → Billing Portal.
3. Highlight **Niko's code** paths in each step.

### Phase 2 — Reconciliation (after merge logic confirmed)

1. Read Notion: *Incorrect merges - 09/08/2026 - Post merge logic & New Billing End Date Logic Update*.
2. Verify the billing-start-first winner logic and canceled-vs-active manual-review exception against the live Design Manager code.
3. Update the business-rule catalog and data dictionary when implementation changes.

### Phase 3 — Non-Stripe and edge cases

1. Document non-Stripe / bulk-paid member handling.
2. Document vendor batch Stripe import unknowns.
3. Capture in-progress items: Apple Wallet, print cards, GreenRope emails.

### Phase 4 — Handoff polish

1. Fill known gaps / open questions.
2. Add deployment notes and Design Manager drift warnings.
3. Optional: publish summary to BWC Notion via `knowledge-capture`.

---

## In-progress work callouts (document status only)

| Item | Document as |
| --- | --- |
| Add-to-Apple-Wallet (member page) | In progress — note intended behavior and current code presence/absence |
| Print member cards | In progress |
| GreenRope API emails | In progress |

---

## Sources to cross-reference

- Better World Club (Notion)
- BWC Library of Scripts Read Me / Sub Import
- Better World Domains
- Better world - sign up form error and data stuff
- Better World Club Imports
- BWC - Alias Mapping v2
- BWC Current work stream
- BWC - Current State Items
- Incorrect merges - 09/08/2026 - Post merge logic & New Billing End Date Logic Update
- `../bwc_repo_webhook_final/docs/` (architecture, runbooks, current workstream)
- This repo's fetched CMS folders

---

## Agent kickoff prompt (copy into fresh Cursor chat)

```
You are documenting the Better World Club member lifecycle for client handoff.

Workspace: /Users/zeke/Desktop/projects/client_projects/betterworld
Primary repo: bwc_full_repo_for_documentation
Read first: bwc_full_repo_for_documentation/docs/TASK_HANDOFF.md
HubSpot portal: 44020082 (always use --account 44020082)

Deliver markdown docs under bwc_full_repo_for_documentation/docs/.
Trace live code; distinguish confirmed vs assumed vs unknown.
Start with the normal signup → Stripe → HubSpot → portal flow and highlight Niko's code.
Do not modify CMS source files unless asked.
```
