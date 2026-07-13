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

## Engineering Notes

A few real issues hit while building this, and how they were fixed:

**Transcript shape mismatch.** The transcript API doesn't return one clean string — it returns an array of small timestamped segments (`{text, offset, duration}`). Feeding that directly to the AI would have wasted tokens on noise and broken the prompt. Fixed with a Code node that joins segments into one continuous string and caps it at ~12,000 characters to stay within context limits.

**Notion auth kept failing silently.** The Notion node repeatedly threw "Authorization failed" even after triple-checking the API key was correct and valid (verified independently via a direct curl request). Root cause turned out to be a broken/stale credential object in n8n itself, not the key — recreating the credential from scratch (rather than editing the existing one) resolved it. Lesson: when a directly-verified-valid key still fails inside a tool, suspect the tool's credential cache before the key.

**Database connection permissions don't always propagate from parent pages.** A Notion integration connected at a workspace level didn't automatically grant write access to a specific database nested inside it. Had to explicitly add the integration's connection on the exact database page itself.

**Notion's "From list" database picker returned empty despite a valid, connected integration.** Switched to referencing the database directly by ID (via its URL) instead of relying on the dropdown search — more reliable in practice.

## Engineering Notes

Real problems hit while building this, not just the happy path:

- **Transcript shape mismatch.** The transcript API doesn't return one clean string — it returns an array of small timestamped segments (`{text, offset, duration}`). Early runs fed this straight to Groq and got garbage output. Fix: a dedicated trim/join step that concatenates `.text` from every segment into one continuous string, capped at ~12,000 characters to stay inside Groq's context window.
- **Notion auth: OAuth vs. API Key.** The Notion node defaulted to OAuth2, which needs a Client ID/Secret — the wrong credential type for a personal integration. Fix: switch the credential type to "Notion API" (Internal Integration Token) instead, then explicitly add the integration as a connection on the target database page — page-level sharing doesn't inherit from parent pages automatically.
- **Notion "From list" returning empty.** Even with correct auth, the Database dropdown sometimes returns no results due to how Notion's search API scopes visibility. Fix: switch to "By ID" and paste the database ID directly from the page URL — bypasses the list-search step entirely.

## Contact
- Email: samhitatavutu@gmail.com
- Topmate: https://topmate.io/samhita_tavutu

