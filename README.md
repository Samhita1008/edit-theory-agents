# Edit Theory Agents

n8n automation agents I built for Edit Theory, my AI automation practice for D2C brands and local businesses. I'm Samhita, a CS student, and this repo documents what I built, how it works, and what broke along the way.

[Portfolio](https://edittheory-portfolio.vercel.app/) · [Concierge demo](https://edit-theory-concierge-s6ra.vercel.app) · [LinkedIn](https://linkedin.com/in/samhita-tavutu-b17b2a37b/)

## Agents

| # | Agent | What it does | Workflow file |
|---|---|---|---|
| 1 | [Content Repurposing](./1-content-repurposing) | Turns one YouTube video into 10 content assets and saves them to Notion | [Published](./1-content-repurposing/workflow.json) |
| 2 | [AI Outreach](./2-ai-outreach) | Sends personalized cold emails, follows up on a schedule, tracks replies | Not published |
| 3 | [Lead Discovery](./3-lead-scraper) | Finds and scores Instagram leads for a niche and drafts outreach for each | [Published](./3-lead-scraper/workflow.json) |
| 4 | [Restaurant Outreach](./4-restaurant-outreach) | Audits restaurant websites and writes pitches based on what's wrong | Not published |
| 5 | [Email Digest](./5-email-digest) | Sorts your inbox by urgency and sends a morning summary to Telegram | [Published](./5-email-digest/workflow.json) |
| 6 | [Concierge](./6-edit-theory-concierge) | Booking assistant for local businesses (backend docs; frontend is a separate repo) | Not published |

Concierge is the biggest build here: a customer-facing product with a frontend, not just an internal automation. Parts of it run in demo mode, and its README says exactly which parts.

## Repo layout

```
edit-theory-agents/
├── 1-content-repurposing/    README + workflow.json
├── 2-ai-outreach/            README
├── 3-lead-scraper/           README + workflow.json
├── 4-restaurant-outreach/    README
├── 5-email-digest/           README + workflow.json
└── 6-edit-theory-concierge/  README
```

Every README covers what the agent does, how the pipeline runs, the stack, and the engineering notes: real bugs I hit and how I fixed them.

## Using a published workflow

Agents 1, 3 and 5 include a `workflow.json`. All keys and personal IDs are replaced with placeholders.

1. In n8n, create a new workflow, open the menu, and choose **Import from file**.
2. Search the workflow for `YOUR_` and replace each placeholder with your own value (listed in each agent's README).
3. Re-select the credentials on any Gmail, Google Sheets, Notion or Telegram nodes.
4. Run it once manually and check each node's output before activating it.

Never commit real API keys to a repo. If you export your own workflow, search the JSON for keys and IDs first.

## Not published

Agents 2, 4 and Concierge contain prompts and logic I'm keeping private for now. Their READMEs still cover the design and the problems I solved. If you want a walkthrough, email me.

## Stack

n8n · Groq (Llama 3.3 70B) · Gmail · Google Sheets · Telegram · Notion · Apify · Supadata

## Contact

Email: samhitatavutu@gmail.com
