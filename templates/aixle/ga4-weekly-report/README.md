# GA4 Weekly Report

A weekly Google Analytics 4 report for any website. It compares the last full week with the week before and lists where the site can convert better and where spend can be cut. You get a short summary for stakeholders and an HTML presentation. Posting to Slack is optional.

## What you get

- `ga_weekly_summary.md`: a one-page executive summary with the headline KPIs and their week-over-week changes, the top wins and concerns, and the top 3–5 recommended actions.
- `ga_weekly_report.html`: a self-contained presentation that opens offline. It has KPI cards, bar charts comparing the two weeks, and sections for acquisition, content, audience, findings, actions and data quality.
- `ga_weekly_report.md` and `ga_raw_data.json`: the full analysis and the raw GA4 responses behind it.
- Optional: a Slack message with the summary and the HTML file attached.

## How it works

```
Collect GA4 data and analyze  →  Prepare report  →  Post to Slack (optional)
  last full week vs previous      summary + HTML      skipped when no channel
  data quality first              presentation        set; never fails the run
```

- **Data quality comes first.** The report counts only traffic on your site's host. Localhost, preview and staging domains are listed as excluded, and suspected bot traffic is shown separately.
- **Every finding cites a number.** Small week-to-week swings and tracking problems are reported as noise, not as reasons to change the site.
- **Cost is never estimated.** Google Ads cost appears only when Ads is linked to the property and GA4 actually reports it. GA4 has no cost data for other ad platforms.
- **Runs unattended.** A weekly schedule (Monday 09:00 New York time) is installed turned off. You turn it on after a manual test run.

## What it cannot see

- Ad spend outside Google Ads, unless you import cost data into GA4.
- Conversions that are not set up as key events in GA4.
- Anything outside the two weeks it compares. The template does not track long-term trends.

See [SETUP.md](SETUP.md) to connect it.
