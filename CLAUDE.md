# [Project Name]

[One-line description of what this project does.]

## Tech Stack

**Frontend**: [e.g., React 18, TypeScript 5, Vite 5, Tailwind 3]
**Backend**: [e.g., FastAPI 0.109, Python 3.12, SQLAlchemy 2.0, Pydantic v2]
**Database**: [e.g., PostgreSQL 15]
**Infrastructure**: [e.g., Docker Compose, Railway]

## Architecture

```
project-root/
├── src/
│   ├── api/           # Backend service
│   ├── web/           # Frontend app
│   └── ...
├── docs/              # Plans, PRD template, workflow
├── docker-compose.yml # Service orchestration
└── .env.example       # Configuration
```

**Pattern**: [e.g., DDD, MVC, Clean Architecture]
**Separation**: [How services relate to each other]

## Quick Start

```bash
# Full stack
docker-compose up

# Backend only
cd src/api && [start command]

# Frontend only
cd src/web && npm install && npm run dev
```

## Coding Standards (ENFORCE)

**These standards apply to ALL new code and refactors. Existing code is legacy.**

### Do's
- Be concise, sacrifice grammar if needed
- Label unverified claims: [Speculation] [Unverified]
- Full variable names (no single-letter variables except loop indices)
- All imports at top of file
- Small focused PRs with brief summary

### Don'ts
- Add comments that can be inferred from the code
- Add a library if a simple function can do the job
- Inline imports or imports inside try blocks
- Single letter variables
- Test against real databases - ALWAYS mock database interactions in tests

### [Language 1] (ENFORCE)
- [Style rule 1]
- [Style rule 2]
- [Import convention]
- [Naming convention]

### [Language 2] (ENFORCE)
- [Style rule 1]
- [Style rule 2]

### General (ENFORCE)
- No comments inferable from code
- No library if simple function suffices (<10 lines)
- Small focused PRs with summary

**Code Review**: MUST reject PRs violating these standards. No exceptions.

## Testing Philosophy

- ALWAYS mock database interactions in tests
- Unit tests for business logic, integration tests for API routes
- [Coverage target, e.g., 80%+ for new code]
- [Framework, e.g., pytest, vitest, jest]

## Commands

```bash
# [Language 1] formatting
[format command, e.g., uv run black path/to/file.py]

# [Language 1] tests
[test command, e.g., uv run pytest]

# [Language 2] type check
[typecheck command, e.g., npm run tsc --noEmit path/to/file.tsx]

# [Language 2] format
[format command, e.g., npm run prettier --write path/to/file.tsx]

# [Language 2] lint
[lint command, e.g., npm run eslint --fix path/to/file.tsx]
```

## Safety & Permissions

**Allowed without prompt**:
- Read/list files
- Format single files
- Single test runs

**Ask first**:
- Package installs
- git push
- File deletion, chmod
- Full build/e2e suites

## PR Checklist

- Title: `feat|fix|refactor|docs|test|chore(scope): description`
- Lint, type check, tests green before commit
- Small focused diffs with brief summary
- Remove excessive logs/comments
- **Max ~300 lines changed**
- **Link to parent issue/epic if part of larger feature**

## PR Size Guidelines

| PR Type | Max Lines | Max Files |
|---------|-----------|-----------|
| Bug fix | ~100 | 3 |
| Refactor | ~200 | 5 |
| Feature slice | ~300 | 8 |
| Migration-only | any | 1-2 |

### Always Separate PRs
- Database migrations (own PR, merge first)
- Dependency additions (own PR with justification)
- Large refactors (before or after feature, not mixed)
- Test infrastructure changes

### Anti-Patterns (REJECT)
- "Added entire feature" with 1000+ lines
- Mixing unrelated changes
- Refactor + feature in same PR
- Multiple independent features in one PR

## Wave Deployment Checklist

### Pre-Push
1. Local build succeeds
2. Tests pass
3. Lint/format/typecheck clean
4. Types match across layers (entity, DTO, service)

### Post-Push
5. CI checks pass
6. Deployment health endpoints return 200
7. Review comments addressed

### Wave Complete When
- [ ] All CI checks pass
- [ ] All deployment services healthy
- [ ] No build errors
- [ ] Review comments addressed
- [ ] PR rebased onto previous wave (for stacked PRs)

## Context Files

**Backend**: See `src/api/CLAUDE.md` for backend-specific standards
**Frontend**: See `src/web/CLAUDE.md` for frontend-specific standards
