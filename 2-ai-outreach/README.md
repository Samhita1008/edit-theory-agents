# AI Outreach System

Status: ✅ Live workflow

A single n8n canvas combining 3 connected workflows for automated cold outreach:

1. **AI Outreach Agent** — pulls leads from Google Sheets, generates a personalized opener and cold email via Groq, and sends through Gmail.
2. **Follow Up Sequence** — a scheduled 3-day / 7-day / 14-day follow-up cadence, checking reply status before each send and stopping the sequence once a lead responds.
3. **Reply Tracker** — monitors Gmail for replies and updates lead status in Google Sheets accordingly.

## Stack
n8n · Groq (Llama 3.3 70B) · Google Sheets · Gmail

## Engineering Notes

Real problems hit while building this, not just the happy path:

- **Debug code left in the production path.** After fixing an unrelated "empty item" issue on the IF node, the Match Replies Code node still had temporary debug code in it — returning a placeholder (`{ debug: 'check logs' }`) instead of the real matching logic. It ran with a clean "Success" status every time, silently doing nothing. Easy to miss because n8n doesn't distinguish "ran successfully" from "ran successfully and did the wrong thing." Fix: swapped the placeholder back for the actual matching logic, and now treat a suspiciously clean run as reason to check the node's code, not just its status.
- **`console.log` doesn't show up in n8n's Logs panel.** Debugging the empty-match issue by watching n8n's built-in execution log got nowhere — it only shows timing, not `console.log()` output. Had to open the browser's DevTools console (F12 → Console tab) to actually see what the Code node was logging.
- **"No matches" looked like a bug — it wasn't.** Burned a full debugging cycle checking the Gmail data, the sheet data, and the matching regex, all of which were correct. The actual issue: no real lead had replied yet, so there was nothing to match against. Lesson — confirm test data actually exists before assuming the matching logic is broken.

## Note
AI prompts and API credentials redacted from `workflow.json` for IP protection — same pattern as Agent 4. Full workflow file available on request. Contact samhitatavutu@gmail.com or book a call: https://topmate.io/samhita_tavutu
