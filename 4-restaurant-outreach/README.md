# Restaurant Outreach AI System

Status: ✅ Live workflow

A single n8n canvas running 3 connected workflows for automated restaurant outreach:

1. **Lead draft generation** — audits each lead's website (Broken / Mediocre / Good) and skips sites that don't need outreach. Writes a personalized cold email based on the specific problems found. For restaurants with no website, generates a pitch from scratch.
2. **Human review + send** — waits for a manual status change in Google Sheets before sending, keeping a human in the loop before anything goes out.
3. **Reply tracking** — monitors Gmail for replies, classifies them (Interested / Meeting Request / Not Interested) via Groq, and fires an instant Telegram alert for warm replies.

## Stack
n8n · Groq (Llama 3.3 70B) · Google Sheets · Gmail · Telegram

## Note
Full workflow file available on request. Contact samhitatavutu@gmail.com or book a call: https://topmate.io/samhita_tavutu
