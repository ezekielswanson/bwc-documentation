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

## Environment and ownership

| Layer | Authority |
| --- | --- |
| Stripe live vs test | Stripe dashboard; serverless secret `sandboxStripe` name is suspicious — **verify** |
| hapily sync | hapily app in portal `44020082` |
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

HubSpot-only (not in repo): workflows, private apps, access groups, hapily install, Stripe webhook endpoints.

## Notion titles to reconcile (not copied here)

Better World Club; BWC Library of Scripts / Sub Import; Better World Domains; sign-up form errors; Imports; Alias Mapping v2; Current work stream; Current State Items; Incorrect merges - 09/08/2026.

When Notion and this snapshot disagree, cite both and mark **Unknown** until code or live portal wins.
