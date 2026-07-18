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
- Runs independently of any local machine once deployed, no manual step beyond sending the link

## Pipeline
YouTube URL → Webhook Trigger → Supadata (transcript fetch) → Groq (10-asset generation) → Notion (structured storage)

## Stack
n8n · Groq (Llama 3.3 70B) · Supadata (transcript extraction) · Notion · Railway (deployment)

## Access
Full n8n workflow export is not published in this repo to protect the underlying prompt engineering and implementation logic. Available on request for serious inquiries, see contact below.

## Engineering Notes

Real problems hit while building this, not just the happy path:

- **Transcript shape mismatch.** The transcript API doesn't return one clean string, it returns an array of small timestamped segments (`{text, offset, duration}`). Early runs fed this straight to Groq and got garbage output. Fix: a dedicated trim/join step that concatenates `.text` from every segment into one continuous string, capped at ~12,000 characters to stay inside Groq's context window.
- **Notion auth: OAuth vs. API Key.** The Notion node defaulted to OAuth2, which needs a Client ID/Secret, the wrong credential type for a personal integration. Fix: switch the credential type to "Notion API" (Internal Integration Token) instead, then explicitly add the integration as a connection on the target database page, page-level sharing doesn't inherit from parent pages automatically.
- **Notion "From list" returning empty.** Even with correct auth, the Database dropdown sometimes returns no results due to how Notion's search API scopes visibility. Fix: switch to "By ID" and paste the database ID directly from the page URL, bypasses the list-search step entirely.


## Why I built this
Repurposing one video into 10 platform-specific assets by hand is the exact kind of repetitive, high-volume task automation should own. Built this to prove an end-to-end pipeline, transcript extraction, multi-format generation in one AI pass, structured storage, could run with zero manual steps beyond sharing a link.

## Contact
- Email: samhitatavutu@gmail.com

