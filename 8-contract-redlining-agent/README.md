# Contract Redlining and Legal Risk Review Agent

An n8n workflow that reviews a contract clause by clause against a company playbook, scores the risk, and sends it to counsel for approval. Nothing goes to the counterparty.

**Status:** Built and validated in n8n, not yet run end to end on a real contract · **Workflow:** [workflow.json](./workflow.json)

## What it does

1. **Intake.** A webhook takes a contract (PDF upload or pasted text) plus client ID, counterparty, contract type, deal value, governing law and which side we are on. Bad requests get a 400 before any AI call is made.
2. **Prepare.** The text is extracted, hashed with SHA-256 for an audit trail, and split into clauses. The playbook is loaded from a Google Sheet.
3. **Review.** An AI agent reads each clause against the playbook and returns a risk level, whether it breaches a red line, the deviation, and suggested redline language with a fallback.
4. **Score and route.** Risk is scored 0 to 100 and the contract is routed:
   - **Escalate** to general counsel: any critical finding, any red-line breach, any unparsed AI output, or a high-risk finding on a deal above the value threshold.
   - **Counsel review**: any high finding, three or more medium findings, or a missing required clause.
   - **Fast track**: everything else. Counsel still gets an FYI email.
5. **Brief.** A second agent writes an executive summary: headline, top risks, negotiation strategy, recommended action.
6. **Approval.** Counsel gets an HTML redline report with Approve and Reject links. The workflow waits up to 48 hours.
7. **Outcome.** The decision and the fee are logged. The outcome-based fee is only booked after counsel approves.

## Pipeline

```
Webhook -> Config -> Validate -> (PDF extract) -> Normalize -> Hash -> Load Playbook -> Split Clauses
   -> Review Clause (AI) -> Merge -> Parse -> Score and Route -> Executive Summary (AI) -> Merge -> Build Report
   -> Respond to caller
   -> Log Review
   -> Route Switch -> Escalate / Counsel Review -> Email -> Wait for decision -> Record Outcome -> Log Outcome
                   -> Fast Track -> FYI Email ------------------------------^
```

## Stack

n8n · Anthropic (Claude Sonnet) · Google Sheets · Gmail

## Setup

1. Import `workflow.json` into n8n.
2. Create a Google Sheet with three tabs:
   - `Playbook`: Row ID, Clause, Standard Position, Fallback, Red Line, Required (Yes or No)
   - `Review Log`: Row ID, review_id, received_at, client_id, counterparty, contract_type, deal_value, route, risk_score, critical, high, medium, low, red_lines, contract_sha256, status
   - `Outcomes`: Row ID, review_id, client_id, counterparty, route, decision, critical_caught, high_caught, billable_amount, currency, decided_at
3. In the `Config` node, replace `REPLACE_WITH_GOOGLE_SHEET_ID`, set `counsel_email` and `escalation_email`, and set the pricing values (`fee_per_critical_catch`, `fee_per_high_catch`, `base_fee`, `currency`).
4. Re-select the credentials on the Anthropic, Google Sheets and Gmail nodes. The file ships with `REPLACE_ME` as the credential ID.
5. Open `Log Review` and `Log Outcome` and click Refresh Columns to confirm the column mapping.
6. Test with a dummy contract and your own email first. The Gmail nodes send real mail, and the approval links only work while the workflow is active.

Request shape:

```
POST /webhook/contract-review
multipart: contract (file)   or   JSON: contract_text
fields: client_id, counterparty, contract_type, deal_value, governing_law, our_role
```

## Design choices

- **Human in the loop.** The AI drafts and scores. A lawyer approves. No redline leaves the building automatically.
- **Fail toward a human.** If the AI returns something that cannot be parsed, or a risk level outside the allowed four, that clause is marked high and the contract escalates. An empty playbook or unreadable contract text stops the run with an error instead of passing.
- **Contract text is treated as data.** The review prompt tells the model to ignore any instruction written inside a clause.
- **Portable per client.** Sheet ID, emails, thresholds and pricing live in one Config node. A new client means changing that node and the playbook sheet, not the workflow logic.
- **Pay on outcome.** The fee is computed per critical and high catch, but only recorded as billable after counsel clicks Approve. A timed-out review bills nothing.

## Engineering notes

- **The Crypto node changed between versions.** Version 2 only does encrypt and decrypt. The SHA-256 hash action exists only in version 1, so the workflow pins version 1.
- **Pushing a workflow through the API dropped every credential.** The nodes arrived with none attached, and the response listed no auto-assigned credentials. I only caught it by reading the stored workflow back instead of trusting the success message.
- **Pre-filled `=ROW()` formulas break appends.** A Row ID formula filled down 1000 rows makes Google Sheets treat the table as ending at row 1000, so n8n appends below it. Only the Playbook tab, which is read and never appended to, has the formula pre-filled.
- **AI Agent nodes drop the incoming fields.** A Merge node (Combine by Position) after each agent reattaches the clause and contract data. Both merge inputs come from the same upstream node so positions line up.
- **The approval link comes from `$execution.resumeUrl`.** The email node runs before the Wait node, and the URL it embeds resumes that same execution. A resume with no decision is treated as timed out, not approved.

## Known limitations

- Not yet run on a real contract. Clause splitting uses numbered headings and "Article" or "Section" labels, so unusual layouts will group badly.
- Scanned PDFs have no text layer and fail with an OCR message. OCR is not built in.
- Clauses are reviewed independently. A risk that only shows up across two clauses can be missed.
- Contracts longer than 80 clauses are truncated, and the report says so.
- The starter playbook rows are samples. Real use needs the client's own positions reviewed by their counsel.
- This supports a lawyer. It is not legal advice.

## Why I built this

High-stakes document review is a place where reasoning quality and approval gates matter more than speed, and where a client can reasonably pay per outcome rather than per hour. I wanted a build that shows the same approval-gated pattern as my other agents, applied to contracts instead of local-business outreach.

---

Samhita - Edit Theory
