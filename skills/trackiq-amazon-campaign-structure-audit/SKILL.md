---
name: trackiq-amazon-campaign-structure-audit
description: Audits how an Amazon Sponsored Ads account is built rather than how it performed last week — ad groups crowded with unrelated products, the same product advertised in a dozen ad groups competing with itself, keywords stacked in every match type inside one ad group, spend sitting outside any portfolio, and selling products with no ad at all — ranked by the spend each problem touches, with a fix order the team can work through. Use when the user asks for an account structure audit, campaign structure review, PPC account audit, restructure plan, account cleanup, portfolio setup, ad group hygiene, cannibalization between ad groups, or which products are not advertised.
---

# Sponsored Ads Structure Audit

Checks the shape of a Sponsored Ads account — how campaigns, ad groups,
keywords and products are arranged — and ranks every structural problem by
the spend it touches. Performance reports say what happened; this one says
why the account is hard to steer.

Run it when taking over an account, before a restructure, and once a quarter.

## Requires

- The TrackIQ MCP, for `list_marketplaces`, `get_campaigns`, `get_ad_groups`,
  `get_targets`, `get_product_ads`, `get_portfolios` and
  `get_product_performance`.
- Nothing else. No filesystem or internet needed.
- **Without the MCP:** works from a Sponsored Products bulk file exported
  from Campaign Manager plus a business report by ASIN.

## First run

Fill in a copy of `assets/account.example.md` saved as account.md beside the
skill. Every TrackIQ skill reads the same file, so an account already set up
for another TrackIQ report needs nothing added here. The audit thresholds
below live in its Structure block; the defaults are sensible for most brands.

If the runtime has no filesystem, print the same block and ask the user to
paste it into their project instructions once.

## Read first

- `assets/pulls.md` — the seven pulls, state handling and pagination
- `assets/method.md` — the six checks, their thresholds, and the ranking
- `assets/checks.md` — what to verify before anything is sent
- `assets/report-template.html` — the report. Replace every `{{TOKEN}}`.

## Non-negotiables

1. **Rank findings by the spend they touch, never by count.** Forty crowded
   ad groups spending $300 between them matter less than one spending
   $9,000. A finding with no spend behind it is listed, not prioritised.
2. **Judge structure on enabled entities only.** Pull `state='all'` so the
   totals reconcile, then filter to `enabled` before flagging anything.
   Archived clutter is not a structural problem; recommending changes to
   paused ad groups is noise.
3. **Never recommend restructuring something that is working just to make
   it tidy.** A crowded ad group at 6x ROAS gets a note, not a rebuild.
   Every recommendation names the control the client gains by making it.
4. **Sponsored Brands has no product ads.** `get_product_ads` covers SP and
   SD only. Product-coverage checks say so rather than implying SB was
   checked.
5. **Paginate until a pull returns fewer rows than the limit.** Targets and
   product ads cap out on any real account. If you stop early, call every
   count a floor.
6. **Portfolio coverage is arithmetic, not a join.** Spend outside any
   portfolio = total campaign spend minus the sum of portfolio spend, over
   the same window and states. State the window.
7. **An unadvertised product needs sales to matter.** Only flag ASINs with
   ordered revenue in the window and no enabled product ad — and check
   stock before recommending ads on one.
8. **Never mix ID namespaces.** Only Sponsored Ads IDs belong in this report.
   AMC `campaign_id`s are a different namespace and never join to these.
9. **Never print `account_id`.**

## What the account will argue with

Some sprawl is deliberate: a catch-all auto campaign, a brand-defence ad group
holding the whole catalogue, a launch campaign that duplicates the hero on
purpose. Read the top findings before sending and mark the ones that look
intentional rather than calling them errors.

## What it pairs with

`trackiq-amazon-wasted-spend` finds the keywords to cut; this finds why they
were hard to see. `trackiq-amazon-budget-pacing` needs portfolios to pace
against — the portfolio-coverage finding here is what makes it usable.

## Delivery

The output is produced in the chat first. Delivery is the last step and the
method comes from the Delivery block in account.md — never ask per run.

| Method | What to do | Needs |
|---|---|---|
| `in-chat` | Return the report. The default, and the fallback for every other method. | nothing |
| `file` | Write it beside the skill, dated. | a filesystem |
| `slack` | Post the headline findings as text, then upload the file. | a connected Slack tool |
| `n8n` | POST it to the configured webhook. | network access |
| `email` | Hand it to the connected mail tool. | a connected mail tool |

Confirm before the first outward send of a session, fall back to in-chat
loudly when a method is unavailable, and never substitute a different
outward channel.

## Version

`trackiq-amazon-campaign-structure-audit` v1.0.0 (2026-09-21).

If the user asks whether this skill is current, fetch
`https://trackiq.com/skills/registry.json`, compare the `version` field for
`trackiq-amazon-campaign-structure-audit`, and if it is newer, give them the download link and
the one-line changelog. Do not fetch at any other time.
