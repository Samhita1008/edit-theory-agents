# RWA Complaint and Vendor Coordination Agent

An n8n system that runs the complaint-to-resolution loop for an Indian apartment society (RWA) over Telegram, with the secretary approving everything that goes outside the building.

## Status

Built and tested end to end on test data (test residents, test vendors, my own Telegram account). It has not been piloted with a real society yet. WhatsApp is not connected: vendor messages go out through a pre-filled WhatsApp link the secretary taps. This is a working build, not a production deployment.

## What it does

Four workflows on one n8n canvas, built and tested one at a time.

1. **Complaint Intake.** A resident messages the bot ("water leak in B-402"). Groq classifies category (plumbing, electrical, security, common-area, other) and urgency (low, medium, high). The ticket is logged to Google Sheets with a ticket ID, flat number, status and timestamp. The resident gets the ticket ID back.
2. **Vendor Dispatch.** A new ticket triggers a lookup in the society's vendor list for that category. Groq drafts a short dispatch message. The secretary gets it on Telegram with Approve and Disapprove buttons. On approval she gets a pre-filled WhatsApp link to send to the vendor, and the ticket becomes `dispatched`. If no vendor exists for the category, she is told to handle it manually.
3. **Status Tracking.** The secretary sends `/resolve <ticket_id>`. The sheet updates with `resolved` and a timestamp, and the resident is notified automatically. Only the secretary's chat can resolve tickets.
4. **Notice Broadcast.** The secretary sends `/notice <text>`. Groq formats it without adding facts, she approves a preview, and it is sent to every resident on file. Residents reply `/ack`, and each acknowledgement is logged to a sheet so she doesn't scroll a group chat to see who has seen it.

## Pipeline

```
Telegram Trigger -> Config -> Is Resolve? -> Parse -> Find Ticket -> Mark Resolved -> Notify Resident
                                  |
                                  v
                             Is Notice? -> Groq Format -> Approve -> Log -> Send to all residents
                                  |
                                  v
                               Is Ack? -> Log Ack
                                  |
                                  v
        Normalize Input -> Lookup Resident -> Groq Classify -> Parse -> Create Ticket -> Sheets -> Confirm

Sheets Trigger (new ticket) -> Get Vendor -> Groq Draft -> Secretary Approval -> WhatsApp link + status update
```

## Stack

n8n (self-hosted, local Docker with an ngrok tunnel), Groq (`llama-3.3-70b-versatile`), Google Sheets, Telegram Bot API.

## Design choices

- **Human in the loop.** Nothing reaches a vendor or the residents without the secretary's approval. The AI drafts, a person decides.
- **Portable per society.** Sheet IDs, society name and categories live in one Config node. A new society means changing that node and the sheets, not editing workflow logic.
- **No invented content.** The notice formatter is told to keep every fact exactly as given and add nothing.

## Engineering notes

- **One Telegram trigger per bot.** A bot has one webhook, so `/resolve`, `/notice`, `/ack` and normal complaints all enter through the same trigger and are split with IF nodes. A second trigger would conflict.
- **Fixed vs Expression mode.** A value typed with a leading `=` in a Fixed field is stored as literal text. My Normalize Input fields held a literal `=` and the resident lookup silently matched nothing. In Expression mode, never type the `=` yourself.
- **After a Sheets node, `$json` is the sheet row.** The original message is gone. Downstream nodes reference earlier nodes by name, for example `$('Create Ticket').first().json.ticket_id`.
- **Always Output Data on lookups.** A Get Row(s) that finds nothing stops the workflow. Turning this on lets the "not found" and "no vendor" branches run.
- **Strip Groq's markdown fences before `JSON.parse()`.** The classifier sometimes wraps JSON in code fences despite being told not to.
- **Test and production webhooks conflict for Telegram.** n8n can't listen for a test event while the workflow is active. Deactivate to test.
- **Google Sheets "By List" needs the Drive API.** The Sheets API alone leaves the dropdown disabled and the node returns "resource not found".
- **Telegram can't message people first.** Residents must press Start on the bot before they can receive broadcasts.

## Known limitations

- Vendor messages need a manual tap on a WhatsApp link until the WhatsApp Cloud API is connected.
- Residents are added to the `Residents` sheet by hand. There is no self-registration, and I have not built handling for an unregistered resident.
- Duplicate `/ack` messages create duplicate rows. Count unique `chat_id` per `notice_id`.
- Runs on a local machine with a tunnel, so it is offline when that machine is off.

## Why I built this

Most residential societies in India coordinate complaints, vendors and notices through a WhatsApp group and the secretary's memory. I wanted to see how much of that loop a small, approval-gated automation could cover without removing the secretary from the decisions.

## Sheet structure

- `Complaints`: ticket_id, flat_number, category, urgency, resident_message, status, timestamp, chat_id, resolved_at
- `Residents`: chat_id, flat_number, resident_name
- `Vendors`: category, vendor_name, phone
- `Notices`: notice_id, text, sent_at
- `Acks`: notice_id, chat_id, flat_number, resident_name, acked_at
