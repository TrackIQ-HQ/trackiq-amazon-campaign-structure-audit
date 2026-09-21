# Method

Six checks. Each produces a list of findings, each finding carries the spend
it touches, and the report ranks everything by that spend.

Thresholds come from the Structure block in account.md. Defaults are in
brackets.

## 1. Crowded ad groups

Distinct ASINs per **enabled** SP ad group, from `get_product_ads`.

- Flag ad groups advertising more than **[5]** distinct ASINs.
- Spend touched: the ad group's spend.
- Why it matters: every keyword in the group bids the same amount for every
  product in it. A $12 accessory and a $160 hero share one bid.
- Fix: split by price band or product line so bids can differ.

## 2. The same product everywhere

Enabled SP ad groups per ASIN, from `get_product_ads`.

- Flag ASINs advertised in more than **[6]** enabled ad groups.
- Spend touched: the sum of that ASIN's product-ad spend.
- Why it matters: the ad groups bid against one another for the same
  shopper, and the report cannot say which one earned the sale.
- Fix: keep one ad group per intent (brand, generic, competitor, auto) and
  pause the rest. Name the one to keep — the highest ROAS with at least
  **[10]** orders.

## 3. Match-type stacking

Keywords from `get_targets` (`record_type='KEYWORD'`), enabled only. Group by
`ad_group_id` and normalised keyword text.

- Flag any keyword present in two or more match types **inside one ad group**.
- Spend touched: the combined spend of the stacked copies.
- Why it matters: exact and broad in one ad group share a budget and cannot be
  bid separately by intent.
- Fix: move the exact copy to its own exact ad group and negate it in the
  broad one. Say that negatives are not visible to this skill.

The same keyword in *different* ad groups is a wasted-spend finding, not a
structure one — leave it to `trackiq-amazon-wasted-spend`.

## 4. Spend outside any portfolio

```
outside = total campaign spend (pull 1, enabled) - sum of portfolio spend (pull 2, enabled)
share   = outside / total campaign spend
```

- Flag when `share` exceeds **[10%]**.
- Spend touched: `outside`.
- Why it matters: portfolios are the only budget grouping Amazon enforces.
  Spend outside one cannot be capped or paced as a group.

## 5. Selling products with no ad

ASINs with ordered revenue in `get_product_performance` and no **enabled**
SP or SD product ad in `get_product_ads`. Join on uppercase ASIN.

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
