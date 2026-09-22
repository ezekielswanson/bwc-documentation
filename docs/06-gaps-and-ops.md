# Gaps, operations, and ownership

What this snapshot does **not** fully explain, plus how to operate and where source lives.

## Non-Stripe / bulk-paid members

Project notes cite Equipment Controls, TriMeter, Pointz, League of American Bicyclists, New Wheel. **No first-class non-Stripe path** exists in these three Design Manager folders.

| Question | Finding |
| --- | --- |
| Dedicated serverless route for externally billed members | **Unknown** / not present |
| Coverage dates without Stripe | Would have to be written onto hapily Subscription or Contact **manually or via import scripts** (sibling `bwc_repo_webhook_final`, `bwc-production-migration`)
Treat vendor **batch Stripe subscription imports** are pending method of how to easily import stripe subscription in mass

## In-progress (handoff callouts)

| Item | In this snapshot |
| --- | --- |
| Add Member Cards to Apple Wallet | |
| Print member cards | 
| GreenRope API emails - Complete the green rope -> hubspot connection to send welcome pack emails to new member sign ups in hubspot|

## Known edge cases (from code + handoff)

| Case | Evidence |



## Operations (when something is wrong)

1. **Portal shows canceled/old plan.** Check association label 92 and `subscription_sync_anomaly_reason`. Review the [Contacts With Conflicting Data](https://app.hubspot.com/contacts/44020082/objectLists/1116/filters) segment before backfill. Do not force a winner when the canceled-vs-active manual-review exception applies.
2. **No member card.** Confirm hapily Subscription exists, then `member_card_no` empty, then webhook/secret `generateMemberCardNo`.
3. **Join date missing.** Confirm `checkout.session.completed` reached `join-date-mapping`; Contact email must match Stripe customer email; field is write-once.
4. **Checkout products empty.** `get-stripe-product-data` + primary_product metadata + lookup keys.
5. **Billing Portal fails.** Portal uses `customer_id` from labeled sub; missing ID alerts in UI.
6. **Renewal.** Live path is `handle-renewal-submit`, not `handle-renewal-purchase`.

Do not use member PII in docs or logs. Winner-selection logs identifiers only.

## Stripe webhook operations

The live destination list, event subscriptions, snapshot payload types, and API versions were verified in Stripe on 2026-09-22. The code-level controls below still require verification in the active HubSpot bundles.

| Control | Current evidence | Required operating rule |
| --- | --- | --- |
| Signature verification | **Unknown** from Stripe configuration | Verify `Stripe-Signature` with the signing secret assigned to that exact endpoint. Store only the secret name in docs; never the value. |
| Idempotency | **Unknown** | Store or otherwise deduplicate Stripe event IDs before making repeatable HubSpot or Climate Clean writes. |
| Event ordering | Stripe does not guarantee business events arrive in application order | Resolve current Stripe/HubSpot state rather than assuming delivery order. |
| Acknowledgment | **Unknown** | Return a successful `2xx` promptly; perform long-running work safely after acknowledgment when the platform permits. |
| Retries and replay | Stripe retries failed deliveries; BWC replay procedure is **Unknown** | Reprocess only after confirming idempotency and the current downstream state. |
| Payload schema | **Confirmed:** snapshot events; Zaybra `2020-08-27`, BWC `2024-04-10`, Gift Up `2022-11-15` | Parse against the destination-specific version. Test version changes before production. |
| Test-mode parity | Stripe test mode exists but was not included in the reviewed connection | Confirm equivalent test destinations, event lists, secrets, and synthetic end-to-end tests. |
| Vendor ownership | Zaybra and Gift Up endpoints are external | Escalate vendor delivery/mapping failures to the vendor; do not send vendor events to a BWC endpoint as a substitute. |

### Webhook incident flow

1. Identify the Stripe event ID, type, destination, livemode value, and delivery status in Workbench.
2. Confirm whether the destination is Zaybra, BWC custom, or Gift Up before taking action.
3. Check the destination-specific API version and inspect the matching handler/vendor logs without copying PII into the ticket.
4. Confirm whether the downstream write already succeeded. A Stripe delivery failure and a HubSpot business-rule failure are different incidents.
5. Replay only when the handler is idempotent and the desired HubSpot/Climate Clean outcome is known.
6. Recheck the Contact, hapily Subscription, association label 92, and anomaly fields after recovery.

Stripe references: [webhook guidance](https://docs.stripe.com/webhooks), [process undelivered events](https://docs.stripe.com/webhooks/process-undelivered-events), [testing webhooks](https://docs.stripe.com/automated-testing/webhooks), [subscription webhooks](https://docs.stripe.com/billing/subscriptions/webhooks), and [webhook versioning](https://docs.stripe.com/webhooks/versioning).

## Environment and ownership

| Layer | Authority |
| --- | --- |
| HubSpot environment | Production portal `44020082`; **no HubSpot sandbox** |
| HubSpot source files | Collin should access `bwc-quote-form`, `cms-webpack-serverless-boilerplate`, and `Spark copy` in HubSpot Design Manager |
| Stripe live account | Better World Holdings (`acct_1PIy6rKNMTeBGk8y`) |
| Stripe live vs test | Separate Stripe live and test modes; serverless secret `sandboxStripe` name is suspicious — **verify** |
| Stripe event destinations | Stripe Workbench; confirmed live inventory in [03-serverless.md](./03-serverless.md) |
| hapily sync | hapily app in portal `44020082` plus the vendor-owned Zaybra Stripe destination |
| BWC custom Stripe webhooks | HubSpot serverless routes plus Stripe Workbench configuration |
| This documentation repo | Snapshot only; may drift |
| Sibling scripts | `../bwc_repo_webhook_final/` (backfills, runbooks), migration folders |

CLI: always `--account 44020082`. Machine default may be a different portal.

Re-fetch:

```bash
cd bwc_full_repo_for_documentation
hs cms fetch "bwc-quote-form" "./bwc-quote-form" --account 44020082
hs cms fetch "cms-webpack-serverless-boilerplate" "./cms-webpack-serverless-boilerplate" --account 44020082
hs cms fetch "Spark copy" "./Spark-copy" --account 44020082
```

External configuration not stored in this repo: HubSpot workflows, private apps, access groups, hapily installation settings, Stripe event-destination configuration, signing-secret values, and Stripe test-mode parity.

## Remaining questions

- Is Stripe Tax enabled for BWC recurring payments, and are the required registrations active? Do not infer tax collection from Checkout alone. See [Stripe Tax for subscriptions](https://docs.stripe.com/tax/subscriptions).
- Are BWC’s three custom webhook endpoints signature-verified and idempotent in the active HubSpot bundles?
- Is there a documented replay owner and recovery threshold for Zaybra, BWC custom, and Gift Up deliveries?

## Notion titles to reconcile (not copied here)

Better World Club; BWC Library of Scripts / Sub Import; Better World Domains; sign-up form errors; Imports; Alias Mapping v2; Current work stream; Current State Items; Incorrect merges - 09/08/2026.

When Notion and this snapshot disagree, cite both and mark **Unknown** until code or live portal wins.
