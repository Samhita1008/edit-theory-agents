# Restaurant Outreach AI System

Audits a restaurant's website, writes a pitch based on what's actually wrong with it, and tracks replies. A human approves every email before it goes out.

**Status:** Working · **Workflow:** not published (see [Access](#access))

## What it does

Three connected workflows on one n8n canvas:

1. **Lead drafting:** audits each lead's website and rates it Broken, Mediocre or Good. Good sites are skipped. For the rest, it writes a cold email around the specific problems it found. Restaurants with no website get a pitch written from scratch.
2. **Human review and send:** waits for a manual status change in Google Sheets before sending, so nothing goes out without my approval.
3. **Reply tracking:** watches Gmail for replies, classifies each one (Interested, Meeting Request, Not Interested) with Groq, and sends an instant Telegram alert for warm replies.

## Pipeline

```
Google Sheets (leads) → Website audit (Broken / Mediocre / Good) → Groq (pitch)
→ Human review (status change in Sheets) → Gmail (send)
→ Reply tracker (Groq classification) → Telegram alert on warm replies
```

## Stack

n8n · Groq (Llama 3.3 70B) · Google Sheets · Gmail · Telegram

## Engineering notes

- **AI Agent nodes drop every field except `output`.** Restaurant name, email and URL all vanished after the agent ran, which showed up as "Multiple matching items" errors in later Sheets nodes. Fix: a Merge node (combine by position) after every AI Agent node to reattach the earlier data.
- **`.item` breaks with multiple leads.** `$('Node').item.json[...]` worked for one lead, then threw "Can't determine which item to use" with several. Fix: use `$json['field']` and process items with `$input.all().map(...)`.
- **Silent data loss: 6 leads in, 1 out.** The HTML-stripping Code node used `$input.item`, so it processed only the first item and threw no error. Fix: rewrite it to loop over all items.
- **A Merge node pulled from the wrong input.** It paired 4 agent outputs with all 10 sheet rows by position, so a pitch landed on the wrong restaurant. Fix: feed both inputs from the same branch so the items line up.
- **Blank sheet rows triggered the workflow.** Formatted but empty rows sent ghost leads through. Fix: an IF node right after the trigger that checks the restaurant name isn't empty.
- **Every reply matched the same row.** My test leads all shared one email address. The matching logic was fine, and it worked once each test lead had a unique address.

## Why I built this

Generic cold emails to local businesses get ignored. I wanted every pitch based on something real, an audit of the restaurant's own site, so it reads like someone actually looked. I kept the human review step because outreach to local businesses is about relationships, and I don't want an automation sending things I haven't read.

## Access

The workflow export isn't published because it contains my prompts and audit logic. Email me if you'd like a walkthrough: samhitatavutu@gmail.com

---

Samhita - Edit Theory
