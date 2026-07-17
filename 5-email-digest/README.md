# Email Digest Agent

Status: ✅ Live workflow

Triages your inbox every morning before you open Gmail — categorizes each email by urgency and delivers a single clean summary to Telegram.

## What it does
- Pulls every email from the last 24 hours (not just unread)
- Filters out newsletter/promo noise before it hits the AI, to save tokens and keep signal clean
- Groq categorizes each email: 🔴 needs reply today, 🟡 FYI, 🟢 can wait
- Writes a one-line summary per email, not the full thread
- Sends one sorted, formatted digest to Telegram every morning

## Pipeline
Gmail (last 24h) → Noise Filter (strip newsletters/promo) → Groq (categorize + one-line summary) → Telegram (digest)

## Stack
n8n · Groq (Llama 3.3 70B) · Gmail · Telegram

## File
`workflow.json` — full n8n export, import directly into your own n8n instance to inspect or run.

## Engineering Notes

**Sender/subject data silently dropped after the HTTP Request node.** n8n's HTTP Request node replaces `item.json` with the raw API response, discarding the original Gmail fields (`From`, `Subject`) that came in. First fix attempt (indexing back into the original Gmail items by array position `[i]`) broke once the AI responses came back in a different order than the input — sender names ended up paired with the wrong email. Real fix: rebuilt the AI call as a single Code node with a `for` loop, calling the API directly via `this.helpers.httpRequest()` inside the same iteration as the original item — guarantees correct pairing since there's no separate node boundary for order to get lost across.

**Gmail field capitalization.** The Gmail node returns `From` and `Subject` capitalized — a lowercase `from` filter silently matched nothing and let all emails through unfiltered, with no error thrown. Caught by inspecting one raw output item directly rather than trusting the filter "ran successfully."

**JSON body field rejecting a valid expression.** The HTTP Request node's body needs "Specify Body" explicitly set to "Using JSON" (not "Using Fields Below") before an expression-mode `JSON.stringify(...)` body will actually evaluate — otherwise n8n validates the literal unrendered text and rejects it as invalid JSON.


## Why I built this
Opening Gmail and triaging everything manually every morning is dead time. Built this to do the sorting before I even open the inbox — filter the noise, categorize by urgency, summarize in one line — so opening Telegram tells me what actually needs attention today.

## Contact
- Email: samhitatavutu@gmail.com
- Topmate: https://topmate.io/samhita_tavutu
