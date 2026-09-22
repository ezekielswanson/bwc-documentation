# Data and rules

How BWC turns many hapily Subscription rows into one Contact + portal view. This is not a hapily property catalog.

hapily object in this portal: custom object **`2-32975090`**. Current Subscription label: HubSpot association **`typeId` 92** (Contact → Subscription); reverse **93**.

Hapily vs BWC overlay: see [01-current-state.md](./01-current-state.md). This page is the winner algorithm.

## Winner model

```text
+--------------------------------------------------+
| All associated Hapily subscriptions for Contact |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
| Keep eligible rows only                         |
| - blank subscription_type                       |
| - subscription_status present                   |
| - subscription_id present                       |
| - billing_start_date present                    |
| - billing_end_date present                      |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
| Group by Stripe subscription_id                 |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
| Pick one representative per lifecycle           |
| 1. latest billing_start_date                    |
| 2. latest hs_createdate                         |
| 3. highest HubSpot object ID                    |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
| Pick the Contact-level standard winner          |
| using the same three-field order                |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
| Apply manual-review safety exception            |
+--------------------------------------------------+
                         |
             +-----------+-----------+
             |                       |
             v                       v
+---------------------------+   +---------------------------+
| Safe winner               |   | No safe winner            |
| Update Contact            |   | Keep Contact unchanged    |
| Move current label        |   | Keep label unchanged      |
| Clear anomaly             |   | Set anomaly for review    |
+---------------------------+   +---------------------------+
```

## Winner selection (`updateContactFields.js`)

1. Resolve the Contact associated with the triggering hapily Subscription.
2. Load all hapily Subscriptions associated with that Contact.
3. Exclude any record where:
   - `subscription_type` is populated
   - `subscription_status` is blank
   - `subscription_id` is blank
   - `billing_start_date` is missing
   - `billing_end_date` is missing
4. Group the remaining records by Stripe `subscription_id`.
5. Within each `subscription_id`, choose one representative by:
   - latest `billing_start_date`
   - then latest `hs_createdate`
   - then highest HubSpot object ID
6. Compare the representatives using the same order to select the standard Contact-level winner.
7. Apply the manual-review exception before writing.
8. For a safe winner, copy `status_of_membership`, `billing_start_date`, `billing_end_date`, `products`, `coupon`, and `discount` to the Contact; reconcile the **Current Subscription** label to that record; and clear prior anomaly fields.
9. If no eligible or safe winner exists, do not overwrite the Contact and do not auto-move the label. Set `subscription_sync_anomaly` and `subscription_sync_anomaly_reason`.
10. Keep `requested_cancellation` and `requested_cancellation_date` separate from live subscription state.

### Manual-review exception

Do not auto-select the standard winner when all of these conditions are true:

- the standard winner is `canceled`
- a competing eligible subscription is `active`
- the active subscription has `paid_through`
- the case appears merge-affected or tied to a deleted Stripe subscription lineage

Leave the Contact subscription fields and **Current Subscription** label unchanged, then set the anomaly fields for review. Use the [Contacts With Conflicting Data](https://app.hubspot.com/contacts/44020082/objectLists/1116/filters) segment to resolve these cases. The anomaly fields are cleared when a safe winner is found later.

### Rules that must not drift

- `billing_start_date` is the primary winner field.
- `hs_createdate` is only a tie-breaker.
- Highest HubSpot object ID is the final deterministic tie-breaker.
- `billing_end_date` is required for eligibility and copied to the Contact, but it does not select the winner.
- `subscription_status` is copied from the winner, but it does not select the winner.
- The function must evaluate the complete Contact-level subscription set, not only the triggering record.
- The **Current Subscription** label must follow the same safe winner.
## Source-of-truth matrix

| Field / event | Authority | HubSpot landing |
| --- | --- | --- |
| Payment, plan, coupon, collection method | Stripe | hapily Subscription (hapily sync) |
| Stripe Customer / Subscription IDs | Stripe | hapily `customer_id`, `subscription_id`; Contact `field_id` (Join HubL) |
| Contact match | hapily (email) | Contact |
| Upgrade/downgrade history | hapily delta row (`subscription_type`) | Excluded from winner |
| Current membership display | BWC winner algorithm | Contact denormalized fields + label 92 |
| `join_date` | Stripe `checkout.session.completed` → `event.created` | Contact, write-once |
| `member_card_no` | BWC generator after hapily sub exists | Contact, skip if set |
| `member_id` | Legacy / Contact id fallback | Contact; card uses last 5 digits |
| Portal login | HubSpot private content | Access group (**Unknown** name) |
| Cancellation **intent** | HubSpot workflows | Contact `requested_cancellation` / `requested_cancellation_date`; never used to rank the live subscription |

## Stripe event inputs and downstream ownership

**Confirmed from live Stripe destination configuration on 2026-09-22.** See [03-serverless.md](./03-serverless.md) for the complete inventory and API versions.

| Stripe event | Consumer | Confirmed responsibility or boundary |
| --- | --- | --- |
| `checkout.session.completed` | BWC `join-date-mapping` | Sets Contact `join_date` from the event timestamp; write-once behavior is confirmed in code. |
| `customer.subscription.created` | Zaybra + BWC `handle-successful-payment` | Zaybra consumes the subscription for its sync; BWC performs post-payment work described in Stripe as Climate Clean/date correction. Exact bundled field writes require code verification. |
| `customer.subscription.updated` | Zaybra + BWC `handle-subscription-lifecycle` | Zaybra consumes lifecycle updates; BWC description names `paid_through` synchronization. Exact bundled field writes require code verification. |
| `customer.subscription.deleted` | Zaybra | Vendor sync input. No BWC custom destination is subscribed directly. |
| `invoice.paid` | Zaybra | Vendor sync input. |
| `invoice.payment_succeeded` | BWC `handle-successful-payment` | BWC post-payment input; Zaybra is not subscribed to this event. |
| `invoice.payment_failed` | BWC `handle-subscription-lifecycle` | BWC failed-payment input; Zaybra is not subscribed to this event. |
| `invoice.finalized` | Zaybra + Gift Up | Two external consumers with separate sync responsibilities. |

Receiving an event confirms an integration input. It does **not** prove that the consumer calls the equivalent Stripe API, nor does it prove which object fields it writes into HubSpot. Exact Zaybra API calls and mappings remain **Unknown** until verified with Stripe Workbench request logs or hapily documentation.

## Contact fields written by winner selection

| Contact field | Source |
| --- | --- |
| `billing_start_date` | hapily `billing_start_date` |
| `billing_end_date` | hapily `billing_end_date` |
| `status_of_membership` | hapily `subscription_status` (`cancelled` → `canceled`) |
| `products` | hapily `products` |
| `coupon` | hapily `coupon` |
| `discount` | hapily `discount` |
| `subscription_sync_anomaly` | sync function |
| `subscription_sync_anomaly_reason` | sync function |

## Other identifiers (do not conflate)

| ID | Meaning |
| --- | --- |
| Contact email | Default identity; alias emails `club+<member_id>@betterworldclub.com` are **Assumed** from project notes, not coded in this snapshot |
| `field_id` | Stripe Customer ID on Contact (Join module) |
| `subscription_id` | Stripe Subscription |
| `member_id` | Membership identity |
| `member_card_no` | Printed/displayed card number (`{2-digit}00000{last-5-of-member_id}`) |
| hapily object id | HubSpot record id; not Stripe |

## Dates

| Date | Meaning in this snapshot |
| --- | --- |
| `join_date` | First successful Checkout (`event.created`), ISO `YYYY-MM-DD`, write-once |
| `billing_start_date` | **Primary winner field** |
| `hs_createdate` | First tie-breaker only |
| HubSpot object ID | Final deterministic tie-breaker |
| `billing_end_date` | Required for eligibility and copied to Contact; does not rank the winner |
| `paid_through` | Used only by the manual-review exception as active-payment evidence |
| Renewal form min start | First associated sub `billing_end_date` + 1 day (`main.js`) — **not** necessarily the labeled Current Subscription |

## Status mapping

Contact `status_of_membership` is copied from the safe winner’s hapily `subscription_status`, with spelling normalized. Status does not rank the winner. Portal past-due color is a display rule on the copied status.

## Open questions

- Alias-email mapping implementation — not in these three folders.
- Exact Zaybra API requests and field mappings beyond the confirmed subscribed-event families.
