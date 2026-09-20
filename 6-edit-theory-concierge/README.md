# Edit Theory Concierge — Backend

Status: 🟡 Demo mode (architecture production-ready, two integrations pending real credentials)

The frontend lives in its own repo: **[edit-theory-concierge](https://github.com/Samhita1008/edit-theory-concierge)**. This document covers the n8n backend, pipeline, and engineering notes.

A universal booking assistant for local businesses, restaurants, gyms, salons, parlours, travel agencies, that finds a venue, takes a booking, and follows up automatically. Full-stack build: React/Tailwind frontend, n8n backend, Google Sheets as the system of record.

**This is the flagship build in this repo** — the first agent designed as a sellable product rather than an internal automation, with a frontend a real customer would actually use.

## What it does

1. **Discover** — user picks a category and searches ("italian in chennai"), the system geocodes the query, finds matching venues, and returns a clean ranked list
2. **Book** — user picks a venue and submits booking details; the system validates required fields (category-specific), logs the booking, and responds immediately with a booking ID
3. **Confirm** — in the background, the venue is contacted and their reply (confirm/decline) updates the booking status
4. **Notify** — the customer gets a confirmation email with their booking reference
5. **Status** — the frontend polls for live status updates until the booking is resolved

## Pipeline

```
DISCOVER
Webhook → Extract Input → Groq (extract location from free text) → Nominatim (geocode)
→ Overpass (find venues by category near coordinates) → Format Results → Respond

BOOK
Webhook → Config → Extract Input → Validate Required Fields → [invalid → 400 + missing fields]
→ Generate Booking ID → Append to Google Sheets → Respond (200, pending)
                                ↓ (background, parallel)
                          Contact Venue → Update Status → [confirmed] → Send Confirmation Email

STATUS
Webhook → Extract Booking ID (query param) → Find Booking in Sheet → Respond (status + details)
```

## Stack
React · Tailwind · n8n · Groq (Llama 3.3 70B / GPT-OSS 120B) · OpenStreetMap (Nominatim + Overpass) · Google Sheets · Gmail

## Demo mode, disclosed

Two integrations are built and wired but intentionally run in simulation for the public demo, to avoid real costs and because going live requires a real business partner:

- **Venue discovery** currently runs on free OpenStreetMap data (Nominatim + Overpass) instead of Google Places, to avoid Google Cloud billing during development. A fully-built Google Places node sits on the same n8n canvas, disconnected, with a placeholder credential, plug in a real API key and rewire it in place of the OSM chain to get production-grade venue data and coverage.
- **Venue contact** is simulated: a background branch waits ~25 seconds and randomly confirms or declines, then updates the same Sheet a real integration would. A fully-built WhatsApp Cloud API node sits on the same canvas, same pattern, waiting on a client's WhatsApp Business credentials (phone number ID, access token) and an approved message template. Swap it in place of the wait/random-outcome branch to make venue contact real.

Both swaps are structurally identical: the downstream logic (status update, email, response shape) doesn't change, only the middle step does. This was intentional, so activating either integration later is a rewire, not a rebuild.

## Why I built this
Most of my earlier agents automate a business's own internal process. This one's customer-facing, an actual product someone could hand to a client and say "your customers book through this." Building the demo-mode disclosure directly into the architecture (rather than faking real integrations) was a deliberate choice, the product should be honest about what's live and what activates on setup, the same way any legitimate SaaS tool with a "connect your account" step works.

## Engineering Notes

Real problems hit while building this, not just the happy path:

- **Google Places API (New) requires billing even within free tier**, and a demo/portfolio project shouldn't carry live billing risk. After an autopay scare during setup, switched Discover entirely to free OpenStreetMap tooling. Learned the hard way that Nominatim is a geocoder, not a POI search, "italian restaurant chennai" returns nothing, it can only resolve real place names/addresses. Fix: a two-step chain, Groq extracts just the location from the free-text query, Nominatim geocodes that location to coordinates, then Overpass does the actual category search (`amenity=restaurant`, `leisure=fitness_centre`, etc.) within a radius.
- **A Set node's field type silently stringified nested objects.** `bookingDetails` and `selectedPlace` were coming through as literal stringified JSON (`"{\"name\":...}"`) instead of real objects, so every downstream field lookup returned `undefined`, every booking failed validation even with all fields filled in correctly. Traced by comparing the Set node's output against the Code node's input line by line. Fix: explicit Object type on those fields instead of letting it default to String.
- **A webhook path with a `:param` gets a UUID silently prepended to its production URL.** `status/:bookingId` doesn't resolve to a clean `/webhook/status/:bookingId`, n8n prepends the node's internal webhook ID to avoid route collisions on dynamic paths, and it's not a settings toggle, it's how n8n handles parameterized webhook paths. Fix: dropped the path param entirely, `bookingId` goes in as a query string instead (`/webhook/status?bookingId=...`), clean URL, no UUID.
- **Google Sheets Append silently skips rows it considers non-empty**, even when only one column (a pre-filled `=ROW()` formula) has content and everything else is blank. Caused a booking to land several rows further down than expected. Fix: write the `Row ID` formula at append time via the node's own column mapping, instead of pre-filling the formula down hundreds of rows in advance.
- **Groq model strings go stale.** `llama-3.3-70b-versatile` returned a flat "model does not exist" error mid-build, Groq's available models shift over time independent of anything in this workflow. Fix: cross-checked against a model string known to be currently live in another workflow, since there's no way to trust a hardcoded model name stays valid indefinitely.
- **PowerShell's `curl` alias isn't real curl.** Early manual testing kept failing with cryptic parameter-binding errors, `Invoke-WebRequest` (what `curl` aliases to in PowerShell) doesn't parse `-X`/`-H`/`-d` the way real curl does. Fix: either `curl.exe` explicitly, or native `Invoke-RestMethod` with a PowerShell object piped through `ConvertTo-Json`, the latter is far more reliable for anything with nested JSON.

## Access
Full n8n workflow export is not published in this repo to protect the underlying prompt engineering and implementation logic. Available on request for serious inquiries, see contact below.
