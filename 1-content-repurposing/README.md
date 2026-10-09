# Content Repurposing System

Turns one YouTube video into 10 ready-to-post content assets and saves them to Notion.

**Status:** Working · **Workflow:** [workflow.json](./workflow.json)

## What it does

You send a YouTube URL to a webhook. The system:

1. Fetches the video transcript (Supadata)
2. Sends it to Groq, which generates all 10 assets in one pass
3. Saves everything to a Notion database, one row per video

The 10 assets:

- LinkedIn carousel outline (5 slides)
- 3 short-form reel scripts, each with a different angle
- LinkedIn post
- Email newsletter version
- Standalone quote post
- Instagram caption
- X (Twitter) thread
- Ad concept (hook, angle, CTA)

## Pipeline

```
YouTube URL → Webhook → Supadata (transcript) → Groq (10 assets) → Notion
```

## Stack

n8n · Groq (Llama 3.3 70B) · Supadata · Notion

## Setup

1. Import `workflow.json` into n8n.
2. Replace the placeholders:
   - `YOUR_SUPADATA_API_KEY` in the transcript HTTP Request node (`x-api-key` header)
   - `YOUR_GROQ_API_KEY` in the Groq HTTP Request node (`Authorization: Bearer ...`)
   - `YOUR_NOTION_DATABASE_ID` in the Notion node
3. Create a Notion database with these properties, spelled exactly like this: `Title` (title), and text properties `Carousals`, `Reelscript 1`, `Reelscript 2`, `Reelscript 3`, `LinkedIn Post`, `Newsletter`, `Quote Post`, `Instagram Caption`, `X Thread`, `Ad Concept`.
4. Connect your Notion integration to that database page.
5. Send a test request: `POST /webhook/content-repurpose` with body `{"youtube_url": "https://..."}`.

## Engineering notes

- **The transcript isn't a string.** The API returns an array of small timestamped segments. Feeding that straight to Groq gave garbage. Fix: a Code node joins the `text` from every segment and caps the result at about 12,000 characters to fit Groq's context window.
- **Notion defaulted to the wrong credential type.** It offered OAuth2, which needs a Client ID and Secret. A personal integration needs the "Notion API" (internal token) credential, and the integration also has to be added as a connection on the database page. Access doesn't inherit from parent pages.
- **The Notion database dropdown came back empty.** Even with correct auth, search sometimes returns nothing. Fix: switch the field to "By ID" and paste the database ID from the page URL.

## Why I built this

Repurposing one video into 10 platform-specific posts by hand is repetitive work that automation should own. I wanted to prove the whole chain, transcript, generation, storage, could run with no manual step beyond sharing a link.
