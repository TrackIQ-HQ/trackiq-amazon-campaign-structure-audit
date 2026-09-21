# Before you send it

## 1. Do the totals reconcile?

- Account spend from `get_campaigns` (all states) matches the console total
  for the window, within rounding.
- The headline "spend touched" counts each campaign once. Check it is
  smaller than the sum of the finding rows whenever a campaign appears twice.
- The report states which states were counted and the window.

## 2. Is every finding real?

- **No paused or archived entity appears in a finding.** Spot-check three.
- Every crowded ad group and every multi-ad-group ASIN names its ad groups.
- Every unadvertised product has revenue in the window and has been checked
  against stock.
- The match-type stacking section says negatives are not visible.

## 3. Did the pulls finish?

- `get_product_ads` and both `get_targets` pulls were paged until a call
  returned fewer rows than the limit. If not, every count is called a floor.
- No AMC `campaign_id` appears anywhere in the report.

## 4. Would the client recognise it?

Read the top ten findings as the account manager would. Brand-defence
catch-alls, a launch campaign duplicating the hero, an auto campaign
holding everything — mark them as likely deliberate rather than errors.

## 5. Render check

```js
({ overflows: document.documentElement.scrollWidth > window.innerWidth,
   tables: document.querySelectorAll('table').length,
   rows: [...document.querySelectorAll('table')].map(t => t.querySelectorAll('tbody tr').length),
   tokens: (document.body.innerText.match(/\{\{[A-Z_]+\}\}/g) || []).length })
```

`overflows` false, `tokens` 0, `rows` matching what you computed.
