# Method

Six checks. Each produces a list of findings, each finding carries the spend
it touches, and the report ranks everything by that spend.

Thresholds come from the Structure block in account.md. Defaults are in
brackets.

## 1. Crowded ad groups

Distinct ASINs per **live** SP ad group, from `get_product_ads`.

- Flag ad groups advertising more than **[5]** distinct ASINs.
- Spend touched: the ad group's spend.
- Why it matters: every keyword in the group bids the same amount for every
  product in it. A $12 accessory and a $160 hero share one bid.
- Fix: split by price band or product line so bids can differ.

## 2. The same product bid twice on one search

Live SP product ads (pull 4) joined to live keywords (pull 5) on
`ad_group_id`.

- Flag a keyword — same normalised text, same match type — that is bid in
  **two or more live ad groups advertising the same ASIN**. Those copies
  enter the same auction for the same shopper and bid against each other,
  and the report cannot say which one earned the sale.
- **An ASIN in many ad groups is not a finding on its own.** Accounts split
  by intent — a campaign per match type, keyword theme, auto targeting
  group, brand or generic — put a hero product in twenty or more ad groups
  on purpose. Show the count for context; flag only the overlap.
- Spend touched: the spend of the overlapping keyword copies, each keyword
  counted once even when its ad group holds several flagged ASINs.
- Report one row per keyword and match type, listing its ad groups and
  products.
- Fix: keep the copy in the ad group built for that intent — a
  single-keyword or exact campaign over a multi-keyword catch-all — or, if
  none is, the one with the highest ROAS on at least **[10]** orders. Negate
  or pause the others. Negatives are not visible to this skill; say so.

## 3. Match-type stacking

Keywords from `get_targets` (`record_type='KEYWORD'`), live only. Group by
`ad_group_id` and normalised keyword text.

- Flag any keyword present in two or more match types **inside one ad group**.
- Spend touched: the combined spend of the stacked copies.
- Why it matters: exact and broad in one ad group share a budget and cannot be
  bid separately by intent.
- Fix: move the exact copy to its own exact ad group and negate it in the
  broad one. Say that negatives are not visible to this skill.

The same keyword in *different* ad groups for the same product is check 2.
The same keyword across unrelated products is a wasted-spend question — leave
it to `trackiq-amazon-wasted-spend`.

## 4. Spend outside any portfolio

Every `get_campaigns` row carries `portfolio_id`; it is null when the
campaign sits outside any portfolio.

```
outside = spend of enabled campaigns whose portfolio_id is null
share   = outside / spend of all enabled campaigns
```

Use `get_portfolios` for portfolio names only.

- Flag when `share` exceeds **[10%]**.
- Spend touched: `outside`.
- Why it matters: portfolios are the only budget grouping Amazon enforces.
  Spend outside one cannot be capped or paced as a group.

## 5. Selling products with no ad

ASINs with ordered revenue in `get_product_performance` and no **live**
SP or SD product ad in `get_product_ads`. An ASIN whose only enabled ads sit
in paused campaigns is unadvertised. Join on uppercase ASIN.

- Flag when the ASIN's revenue in the window exceeds **[$500]**.
- Spend touched: none — rank these by revenue instead, in their own table.
- Check stock before recommending an ad; a product with nothing to sell
  should stay dark.

## 6. Spend concentration

From `get_campaigns`, enabled only.

- Share of spend in the top 10 campaigns, and campaigns with spend but zero
  orders in the window.
- Informational. It frames the account; it does not produce a fix row.

## Ranking and the fix order

One combined table, sorted by spend touched, descending. Unadvertised
products sit in their own table sorted by revenue.

The fix order is the top five rows, rewritten as instructions a team can
start today, each with the control it buys: "split Patio Lighting —
Broad into two ad groups so the $38 and $164 products stop sharing a bid".

## Headline

Total spend touched by at least one finding, counted **once per campaign**
— a campaign can carry several findings, and adding them double-counts.
