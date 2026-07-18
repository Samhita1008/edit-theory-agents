# Lead Discovery System

Status: ✅ Live workflow

Finds and qualifies Instagram leads by niche automatically — from hashtag generation to a scored, personalized outreach draft ready to send.

## What it does
- Trigger: webhook accepts a niche as input
- Groq generates 5 relevant Instagram hashtags for that niche
- Apify scrapes Instagram by hashtag to pull matching creator profiles
- Leads are deduplicated and filtered by follower count
- Groq scores each lead and detects their language
- Groq generates a personalized opener + cold email per qualified lead
- Every lead is appended to Google Sheets, ready for the Outreach System (Agent 2) to pick up

## Pipeline
Webhook (niche input) → Groq (hashtag generation) → Apify (Instagram scrape by hashtag)
→ Deduplicate + Filter (by followers) → Groq (score lead + detect language)
→ Groq (personalized opener + cold email) → Google Sheets (append lead)

## Stack
n8n · Groq (Llama 3.3 70B) · Apify (Instagram Hashtag Scraper) · Google Sheets

## Why I built this
Manually searching Instagram by hashtag and copy-pasting profiles into a sheet doesn't scale past a handful of leads. Built this to turn a single niche into a scored, qualified, ready-to-contact lead list automatically — feeding directly into Agent 2 so the whole pipeline from discovery to outreach runs without a manual handoff in between.

## Engineering Notes

Real problems hit while building this, not just the happy path:

- **Groq wrapping JSON in Markdown fences.** Both the hashtag generation and lead-scoring calls came back wrapped in ```json fences, breaking `JSON.parse()` on every run. Fix: strip the fences before parsing, applied consistently across every Groq response in this workflow.
- **Apify rejects `#` in hashtag input.** The Instagram Hashtag Scraper's API expects a bare hashtag string, not the leading `#`. Fix: sanitize every hashtag (strip the `#`) before it's sent to Apify.
- **HTTP Request nodes overwrote the original lead data — repeatedly.** Same root issue as Agent 4/5: n8n's HTTP Request node replaces `item.json` with the raw API response, dropping whatever came in. Here it hit twice — first Google Sheets ended up with only AI-generated fields because the original lead data (name, username, followers) had been silently overwritten by the Groq response before it reached the sheet. Fix: manually re-merge the original lead fields back in after every HTTP Request node, and make sure each Groq prompt carries the *complete* lead context itself (name, username, followers, topic) rather than relying on fields surviving the trip through multiple HTTP nodes.
- **Apify's first response is just run metadata, not the scraped leads.** Calling the scraper API returns a run object with a `defaultDatasetId` — the actual leads have to be fetched separately from that dataset. Missed this at first and tried to parse leads straight out of the run-trigger response.
- **A fixed 90-second Wait was unreliable.** Apify's scrape completion time varies run to run, so a hardcoded wait either fired too early (empty results) or wasted time waiting longer than necessary. Still a fixed delay rather than a true completion check — a known tradeoff, not fully solved.
- **Node reference errors from renaming nodes.** n8n expressions like `$('Extract Dataset ID')` require the exact, case-sensitive node name — renaming a node without updating every downstream reference broke the chain silently until run.
- **Duplicate leads across hashtags.** The same Instagram creator often showed up under multiple related hashtags. Fix: deduplicate by username before scoring, so the same lead doesn't get scored and drafted twice.
- **Follower filtering moved before AI enrichment.** Originally every scraped lead went through Groq scoring regardless of follower count. Moved the follower filter earlier in the pipeline so leads that don't qualify never reach Groq — cuts API usage and matters on Apify's free-tier credit limits, which required testing carefully and avoiding unnecessary reruns.
- **Webhook validation added up front.** A `Validate Input` code node checks required fields exist before any API call fires, so a malformed or incomplete trigger doesn't burn Apify/Groq calls for nothing.

## Note
Workflow file not published in this repo (IP protection — same pattern as Agents 2 and 4). Available on request. Contact samhitatavutu@gmail.com
