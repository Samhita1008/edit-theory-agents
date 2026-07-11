# Email Digest Agent

Status: ✅ Live workflow

Triages your inbox every morning before you open Gmail — categorizes each email by urgency and delivers a single clean summary to Telegram.

## What it does
- Pulls every email from the last 24 hours (not just unread)
- Filters out newsletter/promo noise before it hits the AI, to save tokens and keep signal clean
- Groq categorizes each email: 🔴 needs reply today, 🟡 FYI, 🟢 can wait
- Writes a one-line summary per email, not the full thread
- Sends one sorted, formatted digest to Telegram every morning

## Stack
n8n · Groq (Llama 3.3 70B) · Gmail · Telegram

## Note
Full workflow file available on request. Contact samhitatavutu@gmail.com or book a call: https://topmate.io/samhita_tavutu
