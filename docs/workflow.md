# Development Workflow

## Lifecycle

```
PRD → Phases → Waves → Issues → Branches → PRs → Review → Main → Tag → Production
```

## Step 1: Write the PRD

- Create a GitHub Issue using `docs/prd-template.md`
- Every implementable unit gets a row in the phase table
- Resolve all Open Questions before proceeding
- Get team sign-off on Technical Decisions

## Step 2: Break Phases into Waves

- Wave 1 is always foundation (entities, schemas, module scaffold)
- Subsequent waves group 1-4 related units
- Wave N+1 can depend on Wave N within a phase
- Phases are independent and can run in parallel

### Wave Sizing

| Wave Type | Max Issues | Typical Scope |
|-----------|-----------|---------------|
| Foundation | 1 | Module scaffold, entities, config |
| Read operations | 2-4 | GET routes, list/detail views |
| Write operations | 2-3 | Create/update/delete + validation |
| UI screens | 1-2 | Screen + components + hooks |
| Cross-cutting | 1-2 | Auth, analytics, notifications |

## Step 3: Create GitHub Issues

- One issue per implementable unit (1:1 with PRD table rows)
- Title format: `feat(scope): Wave Na - METHOD /path (Phase N, Wave N)`
- Assign to GitHub Project for the phase
- Set Wave field value

## Step 4: Write Wave Plans

- File: `docs/plans/YYYY-MM-DD-phaseN-waveN-description.md`
- Contains: implementation approach, key decisions, edge cases
- Skip for foundation waves (Wave 1) -- pattern is self-evident
- Keep plans concise -- bullets over prose

## Step 5: Implement

- One branch per wave: `feat/pNwN-slug`
- Use git worktrees for isolation when working on multiple phases
- Follow CLAUDE.md standards -- no exceptions
- Run lint + typecheck + tests before pushing

### AI-Assisted Implementation

When using Claude Code or similar AI coding tools:

1. **System prompt**: Append the wave plan as system context
2. **Autonomous mode**: AI implements all issues in the wave, commits, pushes, opens PR
3. **Human review**: AI never merges its own PRs
4. **One wave at a time**: Within a phase, complete and merge one wave before starting the next

#### Prompt Pattern for Autonomous Execution

```
You are implementing Phase N, Wave M.

Rules:
1. Do not ask questions -- make decisions based on codebase patterns
2. Follow CLAUDE.md standards strictly
3. Complete all issues before stopping
4. Commit, push, and open a PR when done
5. Do NOT merge the PR

Issues:
- #NNN: [title]
- #NNN: [title]

Plan:
[contents of wave plan file]
```

## Step 6: Open PR

- One PR per wave (not per issue)
- PR closes all issues in that wave
- Max ~300 lines changed (see CLAUDE.md for size limits)
- Use the PR template (`.github/PULL_REQUEST_TEMPLATE.md`)

## Step 7: Review and Merge

- CI must pass (lint, test, security, build)
- Human reviews code quality and correctness
- Reviewer merges to main
- Verify deployment health after merge

## Step 8: Advance to Next Wave

- Previous wave's PR merged to main
- Next wave branches from updated main
- Repeat Steps 5-7 until phase complete

## Step 9: Release

- Main HEAD auto-deploys to staging
- Production: `git tag vN.N.N && git push origin vN.N.N`
- Rollback: re-deploy previous tag

---

## Rules

### Absolute Musts

- Never skip CI checks
- Never merge your own PR without review
- Never mix unrelated changes in one PR
- Never put implementation details in the PRD -- use plan files
- Always resolve Open Questions before starting a phase
- Always run the build locally before pushing
- Always verify deployment health after merge

### Branching Model

```
main ─── auto-deploys to staging
  ├── feat/*, fix/*, chore/* branches (short-lived, PR to main)
  └── v1.0.0, v1.1.0, ... tags ─── deploy to production
```

- All PRs target `main`
- Staging runs latest `main` HEAD
- Production promotion: `git tag vN.N.N && git push origin vN.N.N`
- Rollback: re-deploy previous tag

### Wave Completion Criteria

A wave is complete when:
- [ ] All issues in wave implemented
- [ ] CI checks pass (lint, test, security, build)
- [ ] Deployment services healthy
- [ ] Review comments addressed
- [ ] PR rebased on main (for stacked waves)

### PR Breakdown Patterns

**Vertical Slice (preferred)**: Ship thin end-to-end slices.
```
Feature: "Add product tags"
PR 1: entities + migration (foundation)
PR 2: repository + service + GET endpoint (read path)
PR 3: POST/PUT/DELETE endpoints (write path)
PR 4: UI components for tag display (frontend foundation)
PR 5: integration with existing views (frontend complete)
PR 6: filtering by tags (cross-cutting)
```

**Horizontal Slice**: Layer by layer when vertical not feasible.
```
Feature: "Bulk actions"
PR 1: shared types + schemas
PR 2: database layer
PR 3: service layer + tests
PR 4: API routes
PR 5: frontend hooks + state
PR 6: UI components
```
