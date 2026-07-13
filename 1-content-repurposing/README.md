# Content Repurposing System

Status: ✅ Live workflow

Turns one long-form video/podcast into 10 platform-ready content assets, automatically.

## What it does
- Trigger: paste a YouTube URL to a webhook to kick off the pipeline
- Fetches the full transcript automatically (Supadata)
- Groq reads the transcript and generates 10 content assets in one pass:
  - LinkedIn carousel outline (5 slides)
  - 3 short-form reel scripts (different angles)
  - Full LinkedIn post
  - Email newsletter version
  - Standalone quote post
  - Instagram caption
  - X (Twitter) thread
  - Ad concept (hook + angle + CTA)
- Every asset is saved directly to a Notion database, one row per video, titled by the source YouTube link
- Runs independently of any local machine once deployed — no manual step beyond sending the link

## Stack
n8n · Groq (Llama 3.3 70B) · Supadata (transcript extraction) · Notion · Railway (deployment)

## Access
Full n8n workflow export is not published in this repo to protect the underlying prompt engineering and implementation logic. Available on request for serious inquiries — see contact below.

## Contact
- Email: samhitatavutu@gmail.com
- Topmate: https://topmate.io/samhita_tavutu
