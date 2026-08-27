# Data Processing Log

## Where the data actually came from

The uploaded `Raw_Data` and `images` folders came through empty (0 bytes each) -
most likely a folder-upload issue, since individual files upload fine but folders
don't reliably. Rather than block on that, the four source CSVs
(`ad_events.csv`, `ads.csv`, `campaigns.csv`, `users.csv`) were **recovered
directly from the `.pbix`'s embedded data model** using
[`pbixray`](https://github.com/Hugoberry/pbixray), which reads Power BI's
internal VertiPaq store. This is the same data Power BI itself loaded - row
counts and every KPI below tie out exactly to the dashboard.

The Power Query source step inside the `.pbix` also revealed the original local
folder layout the file was built from:
```
Power BI Project Data\Meta - Ad Power BI Dashboard\Raw Data\ad_events.csv
Power BI Project Data\Meta - Ad Power BI Dashboard\Raw Data\ads.csv
Power BI Project Data\Meta - Ad Power BI Dashboard\Raw Data\campaigns.csv
Power BI Project Data\Meta - Ad Power BI Dashboard\Raw Data\users.csv
```
- confirming these four files are exactly what belongs in `data/`.

The five images in `images/` are original charts generated directly from this
recovered data (not screenshots) - see the note in `README.md`'s Dashboard section.

## Row counts (recovered data)

| Table | Rows | Columns |
|---|---|---|
| `ad_events` | 400,000 | 7 |
| `ads` | 200 | 7 |
| `campaigns` | 50 | 6 |
| `users` | 9,841 | 7 |

Two Power-BI-generated helper columns (`Event Date`, `Event Hour` - auto-derived
from `timestamp` by Power BI's date hierarchy, not present in the original source
per the Power Query step) were dropped from the exported CSVs to keep `data/`
matching the true original source files.

## Verification against the PDF's "Dashboard Insights"

Every KPI in the PDF's insights section was recomputed independently from the
recovered data and matches exactly - **once filtered to the Facebook page**. The
PDF's numbers (Impressions 216K, Clicks 25.4K, CTR 11.76%, etc.) are **Facebook-only**,
not a combined Facebook+Instagram total, though the original document never states
that. `README.md` reports both platforms separately to avoid the same ambiguity.

| Metric | PDF (unlabeled) | Recomputed, Facebook only | Recomputed, Instagram only |
|---|---|---|---|
| Impressions | 216K | 215,972 | 123,840 |
| Clicks | 25.4K | 25,389 | 14,690 |
| CTR | 11.76% | 11.76% | 11.86% |
| Engagement Rate | 13.56% | 13.56% | 13.60% |
| Conversion Rate | 5.21% | 5.21% | 4.82% |
| Purchase Rate | 0.61% | 0.61% | 0.57% |

## Findings that refine or correct the original PDF

**1. The "Gender" donut chart is targeting data, not actual-audience data - and the
two tell different stories.** The PDF's donut ("Female 43% / Male 22% /
Other-Not-Specified 35%") breaks Facebook engagement down by `ads.target_gender`
- i.e., who the *ad* was aimed at - not `users.user_gender`, who *actually*
engaged. Recomputing by actual user gender gives a close-to-opposite picture:
**Male 55% / Female 35% / Other 10%**. Ads are targeted at women most often, but
men generate the majority of engagement. This is worth flagging prominently -
see `README.md` Key Findings.

**2. `event_type` has a 6th value, `Like`, missing from the documentation and from
the `Engagements` measure.** 12,013 of 400,000 events (3.0%) are `Like`, split
~7,505 Facebook / 4,508 Instagram. Neither the domain knowledge document nor the
BRD mentions it, and it isn't summed into `Engagements` (`Clicks + Shares +
Comments` only). Whether that's intentional (Likes considered too passive to
count as "engagement") or an oversight isn't clear from the source files - worth
a product decision either way, not just a silent gap.

**3. Top-countries ranking doesn't match the PDF's list.** The PDF names "US,
India, Brazil, Germany, UK" as top contributors. Recomputed from raw event
volume, the actual order is **US, UK, Canada, India, Germany** (Brazil is 7th,
Australia - not mentioned in the PDF at all - is 6th). Full ranking in
`images/top_countries.png`.

**4. The "peaks in the evening" claim doesn't hold up.** The PDF states hourly
engagement "peaks around late afternoon & evening (~15-20 hours)" and is
"lowest early morning." Recomputed from the categorical `time_of_day` field,
Facebook engagement is **essentially flat** across Morning/Afternoon/Evening/Night
(7,275-7,382, a ~1.5% spread) - see `images/time_of_day.png`. This looks like
uniformly-distributed synthetic data rather than a real usage pattern, which
makes sense given the domain doc describes this as data "modelled after" Meta's
platform, not real platform data.

**5. Weekly consistency claim does hold up.** Facebook engagement by
`day_of_week` ranges 4,128-4,274 (a ~3.4% spread) - genuinely flat, matching the
PDF's "fairly consistent across weeks, no sharp drop" claim.

## Known limitation

`data/ad_events.csv` is ~28MB. That's within GitHub's normal file-size limits (the
hard cap is 100MB) but large enough that GitHub will show it as a diff-suppressed
binary-like file rather than a friendly inline preview - expected and fine for a
raw data file, just noted here so it's not mistaken for an error.
