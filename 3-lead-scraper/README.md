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

*Being written up — real bugs from this build, not the happy path. Coming soon.*

## Note
Workflow file not published in this repo (IP protection — same pattern as Agents 2 and 4). Available on request. Contact samhitatavutu@gmail.com or book a call: https://topmate.io/samhita_tavutu
