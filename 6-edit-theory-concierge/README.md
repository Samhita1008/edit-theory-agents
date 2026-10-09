# Edit Theory Concierge (Backend)

A booking assistant for local businesses: restaurants, gyms, salons, parlours and travel agencies. It finds a venue, takes a booking and follows up automatically.

**Status:** Demo mode. The core flow works end to end, and two integrations are built but not connected to real accounts (see [Demo mode](#demo-mode)).

The React frontend is in its own repo: [edit-theory-concierge](https://github.com/Samhita1008/edit-theory-concierge). This README covers the n8n backend. Google Sheets is the system of record.

This is the biggest build in the repo. Unlike my other agents, which automate a business's internal process, this one is customer-facing: something a business could hand to its customers to book through.

## What it does

1. **Discover:** the user picks a category and searches (for example "italian in chennai"). The system finds the location, finds matching venues nearby, and returns a clean list.
2. **Book:** the user picks a venue and submits details. The system checks the required fields for that category, logs the booking and replies right away with a booking ID.
3. **Confirm:** in the background, the venue is contacted and its confirm or decline updates the booking status.
4. **Notify:** the customer gets a confirmation email with their booking reference.
5. **Status:** the frontend polls until the booking is resolved.

## Pipeline

```
DISCOVER
Webhook → Groq (extract location from text) → Nominatim (geocode)
→ Overpass (venues by category) → Format results → Respond

BOOK
Webhook → Validate required fields → [invalid: 400 + missing fields]
→ Generate booking ID → Append to Sheets → Respond (pending)
        ↓ (background)
   Contact venue → Update status → [confirmed] → Confirmation email

STATUS
Webhook → Find booking in Sheet (by ID) → Respond (status + details)
```

## Stack

React · Tailwind · n8n · Groq · OpenStreetMap (Nominatim + Overpass) · Google Sheets · Gmail

## Demo mode

Two integrations are built and wired up but run in simulation for the public demo. This avoids billing risk, and going live needs a real business partner.

- **Venue discovery** uses free OpenStreetMap data instead of Google Places. A finished Google Places node sits on the same canvas, disconnected, waiting for an API key.
- **Venue contact** is simulated: the system waits about 25 seconds, then randomly confirms or declines. A finished WhatsApp Cloud API node sits next to it, waiting for a business's WhatsApp credentials and an approved message template.

Both swaps only replace the middle step. The status update, email and response format stay the same, so switching one on is a rewire, not a rebuild.

## Engineering notes

- **Google Places needs billing even on the free tier.** After a billing scare during setup, I switched Discover to OpenStreetMap. Nominatim only geocodes real place names and addresses, so "italian restaurant chennai" returns nothing. Fix: Groq pulls out the location, Nominatim turns it into coordinates, and Overpass does the category search within a radius.
- **A Set node turned objects into strings.** `bookingDetails` and `selectedPlace` arrived as stringified JSON, so every field lookup returned `undefined` and every booking failed validation. Fix: set those fields' type to Object explicitly.
- **A `:param` in a webhook path adds a UUID to the production URL.** n8n does this for parameterized paths to avoid route collisions. Fix: drop the path parameter and send `bookingId` as a query string instead.
- **Sheets Append skipped rows it considered non-empty.** A pre-filled `=ROW()` formula made rows look occupied, so a booking landed several rows down. Fix: write the Row ID at append time through the node's column mapping.
- **Groq model names go stale.** `llama-3.3-70b-versatile` suddenly returned "model does not exist" mid-build. Fix: check against a model string known to work in another workflow, and don't assume a hardcoded name stays valid.
- **PowerShell's `curl` isn't curl.** It's an alias for `Invoke-WebRequest`, which doesn't accept `-X`, `-H` or `-d` the same way. Fix: use `curl.exe` or `Invoke-RestMethod`.

## Why I built this

I wanted to build something a business could hand to its customers, not just run internally. I also chose to disclose demo mode in the README and the architecture instead of faking real integrations. The product should be clear about what's live and what turns on once an account is connected.

## Access

The workflow export isn't published because it contains my prompts and logic. Email me if you'd like a walkthrough: samhitatavutu@gmail.com
