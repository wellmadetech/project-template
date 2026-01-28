# Project Template

GitHub template repository for AI-assisted development with Claude Code.

## What's Included

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Coding standards and AI agent instructions -- fill in per project |
| `docs/prd-template.md` | How to write PRDs that decompose into implementable waves |
| `docs/workflow.md` | Full lifecycle: PRD to phases to waves to PRs to production |
| `docs/plans/` | Wave plan files go here |
| `.github/PULL_REQUEST_TEMPLATE.md` | Standardized PR descriptions |
| `.claude/settings.local.json` | Claude Code permissions (safe defaults) |

## Getting Started

1. Click **"Use this template"** on GitHub to create a new repo
2. Fill in `CLAUDE.md` with your project's tech stack, architecture, and coding standards
3. Write your PRD as a GitHub Issue using `docs/prd-template.md`
4. Follow `docs/workflow.md` to go from PRD to shipped code

## Workflow Summary

```
PRD → Phases → Waves → Issues → Branches → PRs → Review → Main → Tag → Production
```

See `docs/workflow.md` for the full process.
