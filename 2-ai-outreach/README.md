# AI Outreach System

Status: ✅ Live workflow

A single n8n canvas combining 3 connected workflows for automated cold outreach:

1. **AI Outreach Agent** — pulls leads from Google Sheets, generates a personalized opener and cold email via Groq, and sends through Gmail.
2. **Follow Up Sequence** — a scheduled 3-day / 7-day / 14-day follow-up cadence, checking reply status before each send and stopping the sequence once a lead responds.
3. **Reply Tracker** — monitors Gmail for replies and updates lead status in Google Sheets accordingly.

## Stack
n8n · Groq (Llama 3.3 70B) · Google Sheets · Gmail

## Engineering Notes

*Being written up — real bugs from combining the 3 sub-workflows onto one canvas, not the happy path. Coming soon.*

## Note
AI prompts and API credentials redacted from `workflow.json` for IP protection — same pattern as Agent 4. Full workflow file available on request. Contact samhitatavutu@gmail.com or book a call: https://topmate.io/samhita_tavutu
