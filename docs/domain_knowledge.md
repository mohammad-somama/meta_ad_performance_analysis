# Domain Knowledge - Meta_Ad-Performance_Dataset

## About the Data

This dataset represents Meta Ads Performance data, covering campaigns, ads, user
demographics, and ad interaction events. It's modelled after how Facebook/Instagram
(Meta) ad platforms capture data.

The purpose is to analyze advertising performance, optimize targeting, and measure
ROI through KPIs such as:

- Impressions (how often ads are seen)
- Clicks (engagement with ads)
- Purchases (conversions)
- CPM, CPC, CTR, and ROAS (efficiency metrics)
- Audience insights (demographics, location, interests)

## Star Schema

```
                ┌────────────┐
                │   users    │  (dimension)
                └─────┬──────┘
                      │ user_id
                      │
┌────────────┐   ┌────┴───────┐   ┌────────────┐
│    ads     ├───┤ ad_events  │   │ campaigns  │
│ (dimension)│ad_id│  (fact)  │   │(dimension) │
└──────┬─────┘   └────────────┘   └─────┬──────┘
       │           campaign_id           │
       └────────────────────────────────-┘
```

- **Fact table**: `ad_events` - every KPI in this project is derived from this table.
- **Dimension tables**: `ads`, `campaigns`, `users`.

| Relationship | Cardinality | Purpose |
|---|---|---|
| `ad_events[ad_id]` -> `ads[ad_id]` | Many:1 | Get platform, creative type, targeting per event |
| `ads[campaign_id]` -> `campaigns[campaign_id]` | Many:1 | Get budget, timeframe per ad |
| `ad_events[user_id]` -> `users[user_id]` | Many:1 | Get demographics per event |

## Table 1: `ad_events` (fact table)

**Purpose:** captures every interaction between a user and an ad.

| Field | Description | Example | Use in Analysis |
|---|---|---|---|
| `event_id` | Unique identifier for each event | `100234` | Primary key |
| `ad_id` | Links to `ads` table | `501` | Join -> get `ad_platform`, `ad_type` |
| `user_id` | Links to `users` table | `U_1204` | Join -> get demographics |
| `timestamp` | Exact time of event | `2025-03-12 14:30:00` | Date hierarchy (Day, Week, Month) |
| `day_of_week` | Derived: day of the week | `Tuesday` | Weekday vs. weekend comparison |
| `time_of_day` | Derived: segment of day | `Afternoon` | See when users engage most |
| `event_type` | Type of event -> see note below | `Click` | Funnel analysis (Impression -> Click -> Purchase) |

> **Note -> this list is incomplete in the original document.** The source
> `Terminologies`-style doc lists `event_type` as one of `Impression, Click, Share,
> Comment, Purchase`, but the actual data also contains a sixth value, **`Like`**
> (12,013 of 400,000 rows / 3.0%) that isn't mentioned anywhere in the original
> documentation, and isn't included in the `Engagements` measure either (see
> `logs/data_processing_log.md`).

**Usage:** foundation for Impressions, Clicks, CTR, Conversion Rate, and ROAS-style KPIs.

## Table 2: `ads`

**Purpose:** defines ad-level metadata.

| Field | Description | Example | Use in Analysis |
|---|---|---|---|
| `ad_id` | Unique ad identifier | `501` | Primary key; joins to `ad_events` |
| `campaign_id` | Campaign association | `C_101` | Joins to `campaigns` |
| `ad_platform` | Facebook, Instagram, Messenger, Audience Network | `Instagram` | Compare platform performance |
| `ad_type` | Image, Video, Carousel, Story | `Video` | Performance by creative type |
| `target_gender` | Gender targeted | `Female` | Targeting efficiency |
| `target_age_group` | Age group targeted | `25-34` | Target vs. actual engagement |
| `target_interests` | Topics/interests targeted | `Travel, Fashion` | Match vs. actual user interests |

**Usage:** identifies which platform + ad-type combination works best, and whether
targeting matches actual engagement - **this dataset's actual `ad_platform` values
are only `Facebook` and `Instagram`** (127 and 73 ads respectively); Messenger and
Audience Network don't appear in the data despite being mentioned as possible values.

## Table 3: `campaigns`

**Purpose:** campaign-level budget, duration, and strategy.

| Field | Description | Example | Use in Analysis |
|---|---|---|---|
| `campaign_id` | Unique campaign ID | `C_101` | Primary key; joins to `ads` |
| `name` | Campaign name | `"Spring Promo 2025"` | Reporting, filtering |
| `start_date` | Campaign launch date | `2025-03-01` | Track active campaigns |
| `end_date` | Campaign end date | `2025-03-31` | Duration analysis |
| `duration_days` | Derived: campaign length | `30` | Compare pacing & performance |
| `total_budget` | Budget allocated | `$50,000` | Basis for CPM, CPC, ROAS |

**Usage:** enables budget tracking, pacing, and ROI analysis.

## Table 4: `users`

**Purpose:** demographic and interest details of users engaging with ads.

| Field | Description | Example | Use in Analysis |
|---|---|---|---|
| `user_id` | Unique user identifier | `U_1204` | Primary key; joins to `ad_events` |
| `user_gender` | Gender of user | `Male` | Gender-based performance |
| `user_age` | Age of user | `27` | Custom segmentation |
| `age_group` | Grouped age bucket | `25-34` | Compare engagement by age |
| `country` | User's country | `India` | Country-level reach analysis |
| `location` | City/state | `Bangalore` | Geo-targeting |
| `interests` | User's interests | `Tech, Travel` | Match vs. targeting interests |

**Usage:** measures audience targeting accuracy - e.g. comparing who an ad was
*targeted* at (`ads.target_gender`) against who *actually* engaged
(`users.user_gender`). See `logs/data_processing_log.md` for what this project
found when that comparison was actually run.
