# Setup

## 1. Give a Google service account read access to GA4

1. In Google Cloud, choose or create a project, then enable the **Google Analytics Data API** and the **Google Analytics Admin API**.
2. Create a service account and download a JSON key for it.
3. In GA4, open **Admin → Property access management** and add the service account's email as **Viewer**.

## 2. Add the key to the project

1. Upload the JSON key as a project asset named exactly `google-credentials.json`, **in the root folder**, not in a subfolder.
2. Add this asset to the workflow's base assets.

The `google-analytics` MCP server reads the key from `/workspace/assets/google-credentials.json`, the value of the `GOOGLE_APPLICATION_CREDENTIALS` config item. If you store the key elsewhere, change that value.

> **Note:** every workflow in the project can read a project asset. Keep the key only in a project whose members may read your analytics, and never share the asset publicly.

## 3. Fill in the settings

The install form asks for the property id, the site URL and the Slack channel, and asks for `GOOGLE_PROJECT_ID` as a secret. You can change them later under **Secrets & Variables**.

| Config item | What to enter |
|---|---|
| `GA4_PROPERTY_ID` | The numeric property id from **Admin → Property details**, for example `123456789`. Not the `G-…` measurement id. |
| `SITE_URL` | Your site, for example `https://www.example.com`. KPIs count traffic on this host only. |
| `GOOGLE_PROJECT_ID` | The id of the Google Cloud project that owns the service account. |
| `SLACK_CHANNEL_ID` | Optional. A Slack channel id or name. Leave it as `none` to skip Slack. |

## 4. Slack (optional)

The checklist lists the chat integration as required because the Slack step uses it. The workflow runs without it: the Slack step is then skipped and the report stays in the run's outputs.

To post to Slack:

1. Connect the Slack integration.
2. Connect the channel in Aixle (**Integrations → Slack → Connect a channel**). Inviting the bot to the channel is not enough.
3. Put the channel id or name in `SLACK_CHANNEL_ID`.

## 5. Check the runtime

- The steps do not pin a runtime: they run on the agent you have connected (Claude Code, Codex, Kiro and others). They were tested on Claude Code.
- Agent containers need `pipx` and outbound HTTPS to PyPI (`pypi.org`, `files.pythonhosted.org`) and `*.googleapis.com`.

## 6. Run it once, then turn on the schedule

1. Start the workflow by hand from the Workflows page. One run takes about 10–15 minutes.
2. Check `ga_weekly_summary.md` and `ga_weekly_report.html` in the run outputs.
3. Turn on the **Weekly GA4 report (Mon 9:00 New York)** trigger. It runs every Monday at 09:00 `America/New_York`. Change the time zone or time first if you need to.

Scheduled runs belong to the person who installed the template: they use that person's agent credential and are billed to them. Turning the trigger on does not change this. To run as another project member or a team, change **Runs as** in the trigger's settings.
