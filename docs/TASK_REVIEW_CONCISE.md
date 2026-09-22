# Task: tighten the SA pack (only where needed)

**Audience:** Collin / solution architect pack  
**Assignee:** fresh Cursor chat  
**Do not:** rewrite the pack, merge files, or cut Confirmed behavior.

The six client pages are already KISS-shaped. This pass removes **duplication only**. Keep unique facts, diagrams that show different things, and status tags.

## Kickoff prompt (paste into a new chat)

```
Tighten bwc_full_repo_for_documentation/docs/ for a solution architect.
Read docs/TASK_REVIEW_CONCISE.md and apply only the listed cuts.
Do not merge the six client pages. Do not invent new content.
Keep Confirmed / Assumed / Unknown. Keep unique diagrams and file citations.
```

## Keep as-is

| File | Why |
| --- | --- |
| `02-join-form.md` | Already short. Live vs archive + `isRenewal` matter. Drop only the Status table (duplicates bullets). |
| `03-serverless.md` | Endpoint catalog is the value. Do not collapse routes. |
| `04-member-portal.md` | Sequence + HubL details are unique. Move the in-progress table, don’t delete the facts. |
| `05-data-and-rules.md` | Winner algorithm is the core page. Keep numbered steps, SoT matrix, identifiers. |
| Snapshot warning on `docs/README.md` | One place is enough (see cuts). |

## Cuts (needed)

### 1. Hapily overlay — keep once (`01`), point from `05`

`01` already has the boundary table + delta mermaid. `05` “Why custom code exists” repeats that.

- **Keep** in `01`: owner table, one-paragraph hapily delta, overlay mermaid, hapily links.
- **In `05`:** delete the “Why custom code exists” bullets and hapily links. One line: “Hapily vs BWC overlay: see 01. This page is the winner algorithm.”
- **Keep** in `05`: winner model (different from `01` overlay), numbered rules, manual-review exception, SoT, and field maps.

### 2. Two mermaids on `01` — keep both, but don’t retell

System-context mermaid + overlay mermaid show different things. Keep both.

- Delete “The solution architect already knows hapily…” (index already says this).
- Fold “Niko’s code” into the component inventory notes. Drop the extra Niko list (same files).

### 3. `02` Status table

Claims are already in Live vs archive + How it works. Delete the Status table. Keep Open questions; drop the preview bullet that is Confirmed (move into How it works if missing, else delete).

### 4. In-progress items — keep once (`06`)

Apple Wallet / print cards / GreenRope appear in `04` and `06`.

- **Keep** the table in `06`.
- **In `04`:** delete “Not in this snapshot”. One line: “In-progress (Wallet, print, GreenRope): see 06.”

### 5. Edge cases — keep once (`06`)

Logged-in=renewal, lookup_key, coupon vs promo, `items[0]` fallback, 30-minute poll, dead `handle-renewal-purchase` are already documented on 02–04.

- **In `06`:** keep the edge-case table as an ops index (useful), but shrink evidence cells to “see 02 / 03 / 04” instead of restating.
- Do not delete the ops runbook numbered list.

### 6. Secrets list on `03`

Listing every secret name is noise for an SA. Keep: “Secrets are named in `serverless.json`. Verify whether `sandboxStripe` is actually the live key.” Drop the comma-separated dump.

### 7. Mapping helpers on `03`

Unrouted `mapAllContacts`, `mapMemberId`, etc. can be one line: “Additional `map*` files on disk are not in `serverless.json`.” Keep routed mapping endpoints.

### 8. Duplicate open questions

| Drop from | Keep on |
| --- | --- |
| `01` workflow-trigger question | `03` |
| `01` Design Manager drift | `docs/README.md` snapshot warning |
| `04` fallback `items[0]` still hit? | already a Confirmed risk in How it works; drop from Open questions |
| `05` non-Stripe card generation | `06` |

### 9. Root `README.md` vs `docs/README.md`

Root README already points at `docs/README.md`. Do not duplicate the 15-minute path on the root file. Leave portal ID + folder table on the root; leave reading path only in `docs/README.md`.

## Do not cut

- Winner-selection steps and canceled-vs-active manual-review exception (`05`)
- Discount vs `allow_promotion_codes` (`03`)
- `handle-renewal-purchase` early `return;` (`03`)
- Portal property list on `crm_associations` (`04`)
- Ops numbered list (`06`)
- Non-Stripe finding: no first-class path in this snapshot (`06`)

## Done when

- No hapily overlay explained twice
- No in-progress / edge-case restatement except `06` as index
- Six client files still exist
- Word count down only in the sections above; no new pages
