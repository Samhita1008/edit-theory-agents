# Restaurant Outreach AI System

Status: ✅ Live workflow

A single n8n canvas running 3 connected workflows for automated restaurant outreach:

1. **Lead draft generation**, audits each lead's website (Broken / Mediocre / Good) and skips sites that don't need outreach. Writes a personalized cold email based on the specific problems found. For restaurants with no website, generates a pitch from scratch.
2. **Human review + send**, waits for a manual status change in Google Sheets before sending, keeping a human in the loop before anything goes out.
3. **Reply tracking**, monitors Gmail for replies, classifies them (Interested / Meeting Request / Not Interested) via Groq, and fires an instant Telegram alert for warm replies.

## Pipeline
Google Sheets (leads) → Website Audit (Broken / Mediocre / Good) → Groq (personalized pitch)
→ Human Review (Sheets status change) → Gmail (send)
→ Reply Tracker (Groq classification: Interested / Meeting Request / Not Interested) → Telegram (instant alert on warm reply)

## Stack
n8n · Groq (Llama 3.3 70B) · Google Sheets · Gmail · Telegram

## Engineering Notes

Real problems hit while building this, not just the happy path:

- **AI Agent nodes only output one field.** Every AI Agent node in n8n returns just `output`, Restaurant Name, Email, Website URL, everything else gets dropped the moment data passes through. Didn't realize this until every downstream Google Sheets update node started throwing "Multiple matching items" errors. Fix: a Merge node after every AI Agent node (Input 1 = agent output, Input 2 = the data from before the agent ran), merged by position. Repeated 4 times across the 3 workflows.
- **`.item` breaks the moment multiple leads are in flight.** Expressions like `$('Google Sheets Trigger').item.json['Restaurant Name']` worked fine for one lead, then threw "Can't determine which item to use" as soon as multiple leads came through, `.item` is ambiguous with multiple items in the pipeline. Fix: replaced every `.item` reference with `$json['field_name']`, and rebuilt Code nodes around `$input.all()` with `.map()` to process every item explicitly.
- **Silent data loss, 6 leads in, 1 lead out.** The HTML-stripping Code node used `$input.item.json.data` (same `.item` problem as above), no error thrown, it just silently dropped 5 of 6 website leads. Fix: rewrote it as `const items = $input.all(); return items.map(item => {...})`, processing every item instead of just the first.
- **Merge node pulling from the wrong input.** The no-website path's Merge node had the raw Google Sheets Trigger (all 10 leads) as Input 2, merged by position against 4 agent outputs, item 1 from the agent ended up paired with item 1 from the full sheet, a completely different restaurant. Fix: pointed Input 2 to the FALSE output of the IF node instead, so both inputs carried the same 4 items in the same order.
- **Blank sheet rows triggering the workflow.** Formatted-but-empty rows at the bottom of the Google Sheet fired the trigger anyway, sending 4 ghost leads through the pipeline. Fix: an IF node right after the trigger checking `Restaurant Name` is not empty, with the FALSE path disconnected.
- **Reply tracking matching every reply to the same row.** All 10 test leads shared one email address for testing, so the Code node's `rows.find()` always matched the first row (row 2) regardless of who actually replied. Not a code bug, confirmed the matching logic was correct once tested with real, unique email addresses (a reply from a different lead matched its correct row every time).


## Why I built this
Generic cold emails to local businesses get ignored. Built this to ground every pitch in something real and specific, an actual audit of the restaurant's own website, so outreach reads like it was written by someone who looked, not a template. Kept a human-review step before sending, since local business outreach is relationship-sensitive in a way D2C isn't.

## Note
Full workflow file available on request. Contact samhitatavutu@gmail.com
