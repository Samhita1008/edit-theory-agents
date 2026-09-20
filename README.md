# Edit Theory, Automation Agents

n8n workflow files for the AI automation agents built as part of **Edit Theory**, an AI automation practice for D2C brands and local businesses, by [Samhita Tavutu](mailto:samhitatavutu@gmail.com).

**[Portfolio](https://edittheory-portfolio.vercel.app/)** · **[Concierge — Live Demo](https://edit-theory-concierge-s6ra.vercel.app)** · **[LinkedIn](https://linkedin.com/in/samhita-tavutu-b17b2a37b/)**

---

## Agents

| # | Agent | Status | Workflow | Stack |
|---|---|---|---|---|
| 6 | [Edit Theory Concierge](./6-edit-theory-concierge) ⭐ | 🟡 Demo mode | Redacted | React, Tailwind, n8n, Groq, OpenStreetMap → Google Places (swap-ready), WhatsApp Business (swap-ready), Google Sheets, Gmail |
| 1 | [Content Repurposing System](./1-content-repurposing) | ✅ Live workflow | [Published](./1-content-repurposing/workflow.json) | n8n, Groq, Supadata, Notion |
| 2 | [AI Outreach System](./2-ai-outreach) | ✅ Live workflow | Redacted | n8n, Groq, Gmail, Google Sheets |
| 3 | [Lead Discovery System](./3-lead-scraper) | ✅ Live workflow | [Published](./3-lead-scraper/workflow.json) | n8n, Groq, Apify, Google Sheets |
| 4 | [Restaurant Outreach AI System](./4-restaurant-outreach) | ✅ Live workflow | Redacted | n8n, Groq, Gmail, Google Sheets, Telegram |
| 5 | [Email Digest Agent](./5-email-digest) | ✅ Live workflow | [Published](./5-email-digest/workflow.json) | n8n, Groq, Gmail, Telegram |

**Concierge** is the flagship build here, a full-stack (frontend + backend) customer-facing booking product, not an internal automation. See its README for the architecture and what's live vs. demo-mode.

Each folder contains a README describing what the agent does and its stack. Where the "Workflow" column shows Redacted, the implementation (prompts and code) is kept private to protect proprietary logic, available on request for serious inquiries.

## Stack overview

- **Automation:** n8n
- **AI:** Groq (Llama 3.3 70B)
- **Integrations:** Gmail API, Google Sheets, Telegram Bot API, Apify, Supadata

## Access

Published workflows (Agents 1, 3, 5) have credentials and API keys redacted, replace placeholder values with your own before importing. Agents 2, 4, and Concierge keep their full implementation private, available on request for serious inquiries, see contact below.

## Contact

- Email: samhitatavutu@gmail.com
- LinkedIn: https://linkedin.com/in/samhita-tavutu-b17b2a37b/

---

<div align="center">
Samhita — Edit Theory
</div>
