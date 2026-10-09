# Lead Discovery System

Finds Instagram leads for a niche, scores them, and drafts a personalized cold email for each one.

**Status:** Working · **Workflow:** [workflow.json](./workflow.json)

## What it does

You send a niche and a follower range to a webhook. The system:

1. Generates 5 relevant Instagram hashtags (Groq)
2. Scrapes creators posting under those hashtags (Apify)
3. Removes duplicates and filters by follower count
4. Scores each lead and detects their language (Groq)
5. Writes a personalized opener and cold email per lead (Groq)
6. Appends every lead to Google Sheets, ready for the [Outreach System](../2-ai-outreach) to pick up

## Pipeline

```
Webhook (niche) → Groq (hashtags) → Apify (Instagram scrape) → Dedupe + follower filter
→ Groq (score + language) → Groq (opener + email) → Google Sheets
```

## Stack

n8n · Groq (Llama 3.3 70B) · Apify (Instagram Hashtag Scraper) · Google Sheets

## Setup

1. Import `workflow.json` into n8n.
2. Replace the placeholders:
   - `YOUR_GROQ_API_KEY` in each Groq HTTP Request node
   - `YOUR_APIFY_TOKEN` in the Apify HTTP Request node URL
   - `YOUR_SHEET_ID` in the Google Sheets node, and connect your Google credential
3. Make a sheet with these columns: `Name`, `Platform`, `Niche`, `Profile URL`, `Recent post topic`, `Generated Opener`, `Full Cold Email`, `Status`, `Lead Score`.
4. Send a test request: `POST /webhook/scrape-leads` with body `{"niche": "fitness", "min_followers": 1000, "max_followers": 50000}`.

Apify's free tier has limited credits, so test with small follower ranges and avoid unnecessary reruns.

## Engineering notes

- **Groq wrapped its JSON in Markdown fences.** This broke `JSON.parse()` on every run. Fix: strip the fences before parsing, on every Groq response.
- **Apify rejects `#` in hashtags.** The scraper wants a bare string. Fix: strip the `#` before sending.
- **Apify's first response is only run metadata.** The leads live in a separate dataset, fetched using the `defaultDatasetId` from that first response. I initially tried to parse leads out of the trigger response.
- **HTTP Request nodes overwrote my lead data.** n8n replaces the item with the raw API response, so the sheet ended up with only AI-generated fields. Fix: re-merge the original lead fields after each HTTP node, and give every Groq prompt the full lead context.
- **Waiting for Apify is a fixed delay, not a real completion check.** Scrape time varies, so a fixed wait can fire too early or wait too long. A known tradeoff that I haven't fully solved.
- **Renaming a node broke references silently.** Expressions like `$('Extract Dataset ID')` need the exact node name. Fix: rename carefully and recheck downstream expressions.
- **The same creator showed up under several hashtags.** Fix: deduplicate by username before scoring.
- **Filtering moved before the AI step.** Leads outside the follower range used to reach Groq anyway. Filtering first cuts API usage and protects the free-tier credits.
- **Input validation up front.** A `Validate Input` node checks the required fields before any API call, so a bad request doesn't burn Apify or Groq calls.

## Why I built this

Searching Instagram by hashtag and pasting profiles into a sheet doesn't scale past a handful of leads. I built this to turn one niche into a scored, ready-to-contact lead list, feeding straight into the outreach system with no manual handoff.

---

Samhita - Edit Theory
