# Setup Guide

End-to-end setup, from zero to a working research agent.

## Prerequisites

- An n8n account (Cloud or self-hosted)
- A Google account (for the Gemini API key)
- A Tavily account
- A Notion account with permission to create databases and integrations

All three (Google, Tavily, Notion) have free tiers that are sufficient for personal use.

## 1. Get the API keys

### Google Gemini

1. Go to [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
2. Click **Create API key**
3. Save the key — it starts with `AIza...`

Free tier limits as of writing: 1500 requests/day for Gemini 2.5 Flash. Generous for personal use.

### Tavily

1. Go to [tavily.com](https://tavily.com)
2. Sign up (Google login works)
3. Dashboard → **API Keys** → copy your key — starts with `tvly-...`

Free tier: 1000 searches/month.

### Notion

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. **New integration** → give it a name like "n8n research agent" → submit
3. Copy the **Internal Integration Token**

You'll also need to share the specific database with this integration once you've created it — see `notion-schema.md`.

## 2. Create the Notion database

Follow [`notion-schema.md`](notion-schema.md) for the exact column structure and how to connect the integration.

## 3. Import the workflow into n8n

1. Open n8n → **Workflows** → **+** (or **…** menu) → **Import from File**
2. Select `workflow/research-agent.json` from this repo
3. The workflow appears on the canvas with credential placeholders (red icons)

## 4. Configure credentials in n8n

For each red credential icon:

### Google Gemini Chat Model

- Click the credential dropdown → **Create New**
- Paste your Gemini API key
- Save

In the model settings, set **Model** to `models/gemini-2.5-flash`. Avoid preview models like `gemini-3.1-pro-preview` — they often fail silently on free API keys.

### Tavily Search

- Click the credential dropdown → **Create New**
- Paste your Tavily API key
- Save

In the tool's settings, leave **Query** as the `$fromAI()` expression — the AI fills this at runtime.

### Notion

- Click the credential dropdown → **Create New**
- Paste your Notion Internal Integration Token
- Save

In the Notion tool node:
- **Resource**: `Database Page`
- **Operation**: `Create`
- **Database**: pick your database from the list (after the integration is connected to it)
- Map each property using the `$fromAI()` expressions from `notion-schema.md`

## 5. Customize the agent persona

Open the **AI Agent** node, find the **System Message** field. Paste a customized version of [`SOUL-template.md`](SOUL-template.md).

The template uses bracketed placeholders like `[AGENT_NAME]` and `[YOUR_ORG]` — replace these with your context.

## 6. Test

In n8n, open the workflow's chat panel (button on the canvas) and try:

```
Research the top 3 complaints customers have about [a topic you care about]. Save the finding to Notion.
```

Expected behavior:
1. The agent calls Tavily to search the web
2. The agent calls Notion to create a new database row
3. The agent replies with a summary, including source URLs

Then go to your Notion database — you should see a new row with the topic, TLDR, findings, sources, and an auto-filled `Created` timestamp.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Atlas responds but doesn't save to Notion | AI didn't decide to call the tool | Add explicit instruction in your prompt: "Save the finding to Notion." |
| Notion writes literal `$fromAI(...)` strings | Expression mode isn't active in the property field | Click the `fx` icon on the left of the field |
| Notion returns `Invalid parameter key` | Column name has forbidden character (`;` `:` `?` etc.) | Rename the column in Notion |
| Tavily returns nothing | Free tier limit hit or invalid key | Check Tavily dashboard for usage |
| Memory doesn't remember prior turns | Session ID isn't being passed | Verify Simple Memory's Session ID is set to "Take from previous node automatically" |
| Workflow doesn't fire at all | Chat Trigger not connected | Make sure the trigger node is wired into the AI Agent |

## Cost expectations

With normal personal use (10-30 research queries per day):

- Google Gemini: $0 (well under 1500 req/day limit)
- Tavily: $0 (well under 1000 searches/month)
- Notion: $0 (no API quota issues at this volume)
- n8n: $0 on Cloud free tier

Heavy use (hundreds of queries daily, or multiple agents): you'll outgrow free tiers and need to budget ~$20-50/month for the same stack on paid tiers.
