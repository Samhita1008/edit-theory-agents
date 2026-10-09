# AI Outreach System

Sends personalized cold emails, follows up on a schedule, and tracks replies, all from one n8n canvas.

**Status:** Working · **Workflow:** not published (see [Access](#access))

## What it does

Three connected workflows:

1. **Outreach agent:** reads leads from Google Sheets, writes a personalized opener and cold email with Groq, and sends it through Gmail.
2. **Follow-up sequence:** sends follow-ups at 3, 7 and 14 days. It checks reply status before each send and stops once a lead responds.
3. **Reply tracker:** watches Gmail for replies and updates each lead's status in Google Sheets.

## Pipeline

```
Google Sheets (leads) → Groq (opener + email) → Gmail (send)
        → Follow-up sequence (3d / 7d / 14d, stops on reply)
        → Reply tracker (Gmail → match to lead → update Sheets status)
```

## Stack

n8n · Groq (Llama 3.3 70B) · Google Sheets · Gmail

## Engineering notes

- **Debug code left in the production path.** After fixing an unrelated issue on an IF node, the reply-matching Code node still had a placeholder (`{ debug: 'check logs' }`) instead of the real logic. It ran with a green "Success" every time and did nothing. n8n can't tell "ran" from "ran correctly", so now a suspiciously clean run makes me check the node's code, not just its status.
- **`console.log` doesn't appear in n8n's Logs panel.** I wasted time watching the wrong place. The output shows up in the browser DevTools console (F12), not in n8n's execution log.
- **"No matches" wasn't a bug.** I spent a full debugging cycle checking the Gmail data, the sheet and the regex, and all of them were fine. No lead had actually replied yet, so there was nothing to match. Lesson: confirm the test data exists before debugging the logic.

## Why I built this

Cold outreach dies without follow-up, and tracking replies across dozens of leads by hand doesn't scale. I built this to run the whole loop: a personal first message, a follow-up cadence that respects replies, and automatic tracking, without checking a spreadsheet every day.

## Access

The workflow export isn't published because it contains my prompts and matching logic. Email me if you'd like a walkthrough: samhitatavutu@gmail.com
