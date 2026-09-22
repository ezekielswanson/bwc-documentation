# Gaps, operations, and ownership

What this snapshot does **not** fully explain, plus how to operate and where source lives.

## Non-Stripe / bulk-paid members

Project notes cite Equipment Controls, TriMeter, Pointz, League of American Bicyclists, New Wheel. **No first-class non-Stripe path** exists in these three Design Manager folders.

| Question | Finding |
| --- | --- |
| Dedicated serverless route for externally billed members | **Unknown** / not present |
| Coverage dates without Stripe | Would have to be written onto hapily Subscription or Contact **manually or via import scripts** (sibling `bwc_repo_webhook_final`, `bwc-production-migration`) |
| Member cards | `generate-member-card-no` waits for hapily object `2-32975090`. If no hapily row, card generation never completes. |
| Portal access | HubSpot private-content lists — **Unknown** configuration |
| Invoice email suppression | **Unknown** in this snapshot (`stripe_invoice_email.module` exists in Spark as a module shell only) |
| Winner algorithm | Requires `subscription_id`, status, billing dates, blank `subscription_type`. Purely manual HubSpot records may be excluded or anomalous. |

Treat vendor **batch Stripe subscription imports** as **Unknown**. Confirm in Notion *Better World Club Imports* / *BWC Library of Scripts*.

## In-progress (handoff callouts)

| Item | In this snapshot |
| --- | --- |
| Add to Apple Wallet | **Not found** |
| Print member cards | **Not found** (portal shows `member_card_no`; `emailCards` only sets `send_membership_cards_via_email`) |
| GreenRope API emails | **Not found**. Sibling `bwc_repo_webhook_final/docs/integrations/greenrope.md` marks GreenRope as planned/test-stage. |

## Known edge cases (from code + handoff)

| Case | Evidence |
| --- | --- |
| Logged-in new signup treated as renewal | see 02 |
| Missing Stripe `lookup_key` | see 02 |
| Checkout coupon vs promo code | see 03 |
| Generic association returns wrong sub | see 04 |
| hapily sync delay | see 03 |
| `handle-renewal-purchase` dead | see 03 |
| Test Stripe IDs on production Contacts | **Unknown** here; operational risk |
| Alias / merge collisions | **Unknown** here; see Notion *Incorrect merges - 09/08/2026* |
| Cross-subdomain API | Join uses relative `/_hcms/api` — good if Join and serverless share a HubSpot domain |

## Operations (when something is wrong)

1. **Portal shows canceled/old plan.** Check association label 92 on the Contact. Re-run `updateContactFields` / backfill if hapily rows look complete. If no winner, read `subscription_sync_anomaly_reason`.
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
| Test-mode parity | Not available in the reviewed Stripe connection | Confirm equivalent test destinations, event lists, secrets, and synthetic end-to-end tests. |
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
| Stripe live account | Better World Holdings (`acct_1PIy6rKNMTeBGk8y`) |
| Stripe live vs test | Stripe dashboard; serverless secret `sandboxStripe` name is suspicious — **verify** |
| Stripe event destinations | Stripe Workbench; confirmed live inventory in [03-serverless.md](./03-serverless.md) |
| hapily sync | hapily app in portal `44020082` plus the vendor-owned Zaybra Stripe destination |
| BWC custom Stripe webhooks | HubSpot serverless routes plus Stripe Workbench configuration |
| Design Manager folders | `bwc-quote-form`, `cms-webpack-serverless-boilerplate`, `Spark copy` |
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
