# Business Requirements Document (BRD) - Meta_Ad_Performance_Analysis

## Business Objective

The business needs a performance tracking report for advertising campaigns running
on Facebook and Instagram. The report gives visibility into campaign reach,
engagement, conversions, and budget utilization, enabling the marketing team to:

- Identify the most effective platform (Facebook vs. Instagram)
- Track campaign ROI and optimize budget allocation
- Understand audience engagement patterns

## Scope

**In scope:** campaigns running on Facebook and Instagram only.

**Out of scope:** other platforms (Messenger, Audience Network); organic engagement
(only paid ads are included).

## KPIs & Definitions

| KPI | Definition | Formula | Example Use |
|---|---|---|---|
| Impressions | Number of times ads were displayed | `COUNT(event_type = Impression)` | Measure reach |
| Clicks | Number of times users clicked ads | `COUNT(event_type = Click)` | Measure engagement intent |
| Shares | Number of times ads were shared | `COUNT(event_type = Share)` | Viral engagement |
| Comments | Number of user comments on ads | `COUNT(event_type = Comment)` | User sentiment & feedback |
| Purchases | Number of purchases made after seeing ads | `COUNT(event_type = Purchase)` | Conversions |
| Engagements | Total interactions | `Clicks + Shares + Comments` | Engagement volume |
| CTR (Click Through Rate) | % of impressions that resulted in clicks | `(Clicks / Impressions) * 100` | Ad effectiveness |
| Engagement Rate | % of impressions that resulted in engagements | `(Engagements / Impressions) * 100` | Overall ad appeal |
| Conversion Rate | % of clicks that resulted in purchases | `(Purchases / Clicks) * 100` | Funnel efficiency |
| Purchase Rate | % of impressions that resulted in purchases | `(Purchases / Impressions) * 100` | Conversion from reach |
| Total Budget | Total spend allocated to campaigns | `SUM(campaigns.total_budget)` | Cost analysis |
| Avg. Budget per Campaign | Average budget allocation per campaign | `Total Budget / Campaign Count` | Budget distribution |

> The exact DAX implementing every one of these (extracted directly from the
> `.pbix`) is in [`scripts/dax_measures.dax`](../scripts/dax_measures.dax) -
> note that `Engagements` deliberately excludes the `Like` event type .

## Chart Requirements

| # | Visual | Chart Type | Notes |
|---|---|---|---|
| 1 | Target Gender | Donut chart | Metric switches dynamically via parameter, shows which gender segment contributes most to the selected metric |
| 2 | Target Age Group | Bar chart | One bar per age group; highlights the most responsive age group |
| 3 | Country | Map | Bubble size/color = selected metric; geographic view of reach and engagement |
| 4 | Calendar Month | Calendar heat map | Monthly view from `ad_events[timestamp]`; darker = higher activity; detects seasonal trends |
| 5 | Weekly Trend | Stacked column, by ad type | X = week number, stacks = `ad_type`, Y = selected metric |
| 6 | Hourly Trend | Area chart | X = hour of day (0-23), Y = selected metric; user activity patterns throughout the day |
| 7 | Ad Type | Matrix | Rows = ad types, columns = platform; side-by-side format comparison |

The dynamic metric switcher (chart 1-3 and 5-6) is implemented as a disconnected
"Select Dynamic Measure" table with `Impressions / Engagements / Clicks / Shares /
Comments / Purchases` as options - see `scripts/dax_measures.dax` for the exact
DAX pattern used (`SELECTEDVALUE` + `NAMEOF`).

## As actually built

The live report has **3 pages** rather than a single combined view: a **Facebook**
page and an **Instagram** page (each carrying the full KPI/chart set above,
platform-filtered), plus a **Calendar Tool Tip** page that appears on hover. This
matches the BRD's "identify the most effective platform" objective by giving each
platform its own dedicated, directly-comparable page instead of a single page with
a platform slicer.
