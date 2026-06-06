# ai-career-ops
Inspired by Santifer https://github.com/santifer/career-ops

# AI Career Ops Agent

Customized AI-powered career operations system for:
- Data Engineering
- AI Engineering
- GenAI roles
- C2C consulting opportunities

Built using and inspired by:
https://github.com/santifer/career-ops

Enhanced with:
- Token optimization
- Resume tailoring workflows
- ATS optimization
- AI recruiting automation
- Multi-agent job search strategy

## Automated job application pipeline

A four-agent team (`job-scout`, `resume-tailor`, `application-submitter`,
`application-tracker`, defined in `.claude/agents/`) discovers roles,
tailors materials, drafts applications, and tracks everything in
`dashboard/DASHBOARD.md`. See [WORKFLOW.md](WORKFLOW.md) for the full
pipeline, setup steps (drop your real resume into `resume/base_resume.md`
and edit `resume/criteria.json`), and the human-confirmation gate before
anything is ever sent.
