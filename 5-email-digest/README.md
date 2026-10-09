# Email Digest Agent

Sorts your inbox by urgency every morning and sends one short summary to Telegram.

**Status:** Working · **Workflow:** [workflow.json](./workflow.json)

## What it does

- Pulls every email from the last 24 hours, read or unread
- Filters out newsletters and promo mail before the AI sees it, which saves tokens
- Groq rates each email: 🔴 needs a reply today, 🟡 FYI, 🟢 can wait
- Writes a one-line summary per email instead of reproducing the thread
- Sends one sorted digest to Telegram at 7am

## Pipeline

```
Gmail (last 24h) → Noise filter → Groq (priority + summary) → Telegram digest
```

## Stack

n8n · Groq (Llama 3.3 70B) · Gmail · Telegram

## Setup

1. Import `workflow.json` into n8n.
2. Connect your Gmail (OAuth) and Telegram bot credentials.
3. Create a Header Auth credential for Groq (`Authorization: Bearer YOUR_GROQ_API_KEY`) and select it on the HTTP Request node.
4. Replace `YOUR_TELEGRAM_CHAT_ID` in the Telegram node with your own chat ID.
5. Adjust the schedule if you don't want 7am (the cron expression is `0 7 * * *`).
6. Run it once manually and check the output before activating.

## Engineering notes

- **Sender and subject got lost after the HTTP Request node.** n8n replaces the item with the raw API response, dropping the original Gmail fields. My first fix matched items back by array position, which broke when the AI responses came back in a different order, and senders ended up paired with the wrong summaries. Real fix: one Code node with a `for` loop that calls the API inside the same iteration as the original email, so the pairing can't get lost between nodes.
- **Gmail returns `From` and `Subject` capitalized.** My lowercase `from` filter matched nothing and let every email through, with no error. I only caught it by inspecting one raw output item instead of trusting that the filter "ran".
- **The JSON body field rejected a valid expression.** The HTTP Request node needs "Specify Body" set to "Using JSON" before an expression-mode `JSON.stringify(...)` body evaluates. Otherwise n8n validates the raw text and calls it invalid JSON.

## Why I built this

Triaging my inbox by hand every morning is dead time. I built this to do the sorting before I open Gmail, so opening Telegram tells me what actually needs attention today.
