# Automated Job Application Pipeline

A team of four Claude Code subagents (in `.claude/agents/`) runs the
end-to-end job search as a pipeline. Each stage hands off to the next
via `dashboard/applications.json`, the single source of truth.

## Setup (one-time, required before running)

1. Replace `resume/base_resume.md` with your real resume — agents only
   reorder/re-emphasize this content, they never invent experience.
2. Edit `resume/criteria.json` — target titles, locations, comp floor,
   keyword filters, and which boards to search.

## Pipeline stages

```
job-scout  ─────►  resume-tailor  ─────►  application-submitter  ─────►  application-tracker
(discover &           (tailor resume          (draft — never auto-          (record status,
 score matches)        + cover letter)          send — for confirmation)     render dashboard)
```

| Stage | Agent | Reads | Writes | Status transition |
|---|---|---|---|---|
| 1. Discover | `job-scout` | `resume/criteria.json`, `resume/base_resume.md`, job board APIs | `dashboard/applications.json` (new entries) | → `discovered` |
| 2. Tailor | `resume-tailor` | `resume/base_resume.md`, posting | `dashboard/materials/<id>-resume.md`, `<id>-cover-letter.md`, a tailored Google Doc (Drive) | `discovered` → `materials_ready` |
| 3. Prepare submission | `application-submitter` | tailored materials | Gmail draft or `<id>-submission-checklist.md` | `materials_ready` → `ready_to_submit` |
| 4. Track & notify | `application-tracker` | `dashboard/applications.json` | `dashboard/applications.json`, `dashboard/DASHBOARD.md`, Gmail digest draft | any → next (user-confirmed) |

This mirrors the pattern from Abhijay Vuyyuru's "Build an AI agent that
automates your entire job hunt process" (scrape → score → tailor →
Google Doc → email digest), adapted into Claude Code subagents with an
explicit human-confirmation gate before anything is ever sent externally.

## Human-in-the-loop gate

`application-submitter` only ever produces **drafts** or **checklists**.
Nothing is sent to an employer or recruiter, and no status is ever set
to `submitted`, without you explicitly confirming that specific
application in conversation. This is intentional — job applications are
external, hard-to-reverse actions.

## Running it

Ask Claude Code to run a stage by name, e.g.:
- "Run job-scout to find new matches"
- "Tailor materials for the top 3 discovered roles"
- "Prepare submissions for everything that's materials_ready"
- "Show me the dashboard" / "Mark the Acme Corp application as submitted"

Or ask for the full pipeline ("run the whole job application pipeline")
and Claude Code will chain the agents in order, stopping at the
submission gate for your go-ahead on each application.

## Viewing status

Open `dashboard/DASHBOARD.md` for the human-readable board, or
`dashboard/applications.json` for the raw data.
