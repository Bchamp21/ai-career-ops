# Automated Job Search & Application

Inspired by the n8n template ["Automate job search with LinkedIn, Google Sheets &
AI"](https://n8n.io/workflows/9602-automate-job-search-with-linkedin-google-sheets-and-ai/),
but rebuilt on tools already connected to this account instead of a
self-hosted n8n instance: **Claude (search/scoring/drafting), Indeed +
ZipRecruiter + Dice (job discovery), Notion (tracker), Gmail (outreach),
Google Drive (resumes)**. No LinkedIn scraping — LinkedIn blocks and bans
that kind of automation, and none of the connected tools do it.

## What it actually does

1. **Discover** — searches Indeed, ZipRecruiter, and Dice against the query
   list in [`config/search-criteria.yaml`](../config/search-criteria.yaml).
2. **Filter** — drops listings that are: at the current employer, below the
   comp floor, outside the US, explicitly no-sponsorship (Dice exposes a
   `willingToSponsor` field; other boards are checked by reading the posting
   text), federal/GovCon roles (near-certain clearance/citizenship
   requirement), or already logged in the tracker.
3. **Score** — rates each surviving listing 0–5 against whichever resume
   archetype fits better (Data Engineer/Fabric/Databricks vs. Data
   Scientist/LLM-RAG), using the same scale already in use in the tracker.
4. **Log** — every scored listing (including discarded ones, with a reason)
   is written to the Notion tracker so the next run doesn't re-evaluate it.
5. **Outreach** — for listings scoring ≥ the `auto_send.min_score_to_send`
   threshold, **only if a real recruiter/hiring-contact email was found**
   in the posting or on the company's page, Claude drafts a tailored email
   with the matching resume and sends it via Gmail (capped at
   `auto_send.max_sends_per_run` per run). The Notion row is updated to
   `Applied` with the send logged in Notes.

## The honest limitation

Indeed, ZipRecruiter, and Dice only return **search results and an apply
link** — none of them expose an "apply" API, and there's no browser
automation tool connected here to fill out a third-party ATS form. So most
matches will **not** be auto-submitted; they land in the tracker as
`Evaluated` with the apply link ready for a one-click manual apply. In a
live test run (2026-07-03) across 3 boards, 0 of 9 relevant listings
exposed a contact email — auto-send is the exception, not the rule. If you
want true one-click auto-apply on ATS forms, that requires a browser
automation layer (e.g. Playwright) this system doesn't currently have —
worth adding deliberately, not silently, given how easy it is to submit a
bad application at scale.

## Resumes

Two tailored resume variants live in Google Drive and are treated as the
source of truth (see `config/search-criteria.yaml` for file IDs):

- **Data Engineer** — Microsoft Fabric / Databricks / Medallion
  Architecture / ADF / T-SQL
- **Data Scientist (LLM/RAG)** — agentic LLM workflows / Databricks /
  vector DBs / MLOps

New roles are scored against both and matched to whichever archetype fits
best. Per-application tailoring (adjusting bullets/summary for a specific
JD) is done on demand by Claude, not templated — the two variants are
starting points, not the final artifact for every application.

## Tracker

Notion database **"Bhuvan S — Job Application Tracker"**
(`config/search-criteria.yaml: tracker.database_id`). `Status` is the
pipeline stage (Evaluated → Applied → Responded → ... → Offer Accepted /
Rejected / Discarded / SKIP). `Stage` tracks resume-prep sub-status
(PDF Ready, etc.) and is only set once a tailored PDF exists for that
specific application.

## Schedule

A daily trigger runs the pipeline autonomously (see repo history / Claude
Code Remote triggers for the current schedule) and sends a push
notification summarizing what it found, what it sent, and anything that
needs a manual decision. Every auto-send is logged in Notion for audit —
nothing is sent silently.

## Changing the config

Edit `config/search-criteria.yaml`:
- Add/remove `search_queries` entries to change what's searched.
- Raise/lower `auto_send.min_score_to_send` to make sending more or less
  aggressive (0-5 scale).
- Set `auto_send.enabled: false` to fall back to discovery + logging only,
  with no emails ever sent automatically.
- Add companies to `candidate.current_employer_exclude` logic or extend
  `filters.exclude_locations_substring` as needed.
