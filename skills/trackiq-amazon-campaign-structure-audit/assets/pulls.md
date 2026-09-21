# The pull sequence

## 0. Account

`list_marketplaces` first. Never print `account_id`. If more than one
TrackIQ MCP is connected, call it on each and match on `name` before
pulling anything.

Window: a trailing 30 days. Structure is slow-moving, but every finding is
weighted by spend, and 30 days is the widest window the daily tools keep.

## 1. The seven pulls

| # | Call | Arguments | Gives you |
|---|---|---|---|
| 1 | `get_campaigns` | `ad_type='all'`, `state='all'` | every campaign, its channel, state, spend, sales |
| 2 | `get_portfolios` | `state='all'` | spend and sales per portfolio |
| 3 | `get_ad_groups` | `ad_type='all'`, `state='all'` | `ad_group_id` → `campaign_id`, state, spend |
| 4 | `get_product_ads` | `ad_type='all'`, `state='all'` | one row per product ad: ASIN, SKU, ad group, state, spend |
| 5 | `get_targets` | `record_type='KEYWORD'`, `ad_type='all'`, `state='all'` | keyword text, match type, ad group, spend |
| 6 | `get_targets` | `record_type='TARGET'`, `ad_type='all'`, `state='all'` | product, category and audience targets |
| 7 | `get_product_performance` | same window | ordered revenue and units by ASIN |

## 2. State is the first trap

Every Sponsored Ads tool **defaults to `state='enabled'`**. Pull `'all'` so
the account totals reconcile to the console, then filter to `enabled`
before any check runs. Say in the report which states were counted.

`'all'` on `ad_type` blends attribution windows (SP 7 days, SB and SD 14).
That is fine for weighting findings by spend — spend has no window — but do
not quote a blended ROAS as if it were one channel's.

## 3. Paginate — the limit is binding

`get_product_ads` and `get_targets` return exactly `limit` rows on any real
account. Page with `offset` until a call returns fewer rows than the limit.
If you stop early, say so and call every count a floor.

## 4. What the data will not tell you

- **Sponsored Brands has no product ads.** Product coverage covers SP and SD.
- **No negative keywords.** The MCP does not expose negatives, so match-type
  stacking cannot see whether the broad copy already negates the exact
  term. Say so on that finding.
- **No budgets or budget caps.** Capping-out is a pacing question; send the
  user to `trackiq-amazon-budget-pacing` for it.
- **No campaign-to-portfolio mapping on every row.** Portfolio coverage is
  computed from totals — see `method.md`.

## 5. The ID rule

`get_campaigns`, `get_ad_groups`, `get_targets` and `get_product_ads` share
the Amazon ID namespace. The AMC tools use a different `campaign_id` —
small integers — that never joins to these.
