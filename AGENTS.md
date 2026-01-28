# Repository Guidelines

This repository is a template for AI-assisted development. It is intentionally light on code and focuses on process, PRD structure, and workflow. Contributors should customize `CLAUDE.md` and add project-specific files before building features.

## Project Structure & Module Organization

- `README.md`: template overview and onboarding.
- `CLAUDE.md`: project-specific coding standards, commands, and PR rules (must be filled in).
- `docs/prd-template.md`: PRD format used for GitHub Issues.
- `docs/workflow.md`: phase → wave → PR lifecycle.
- `docs/plans/`: intended location for wave plans (not present yet).

No application source tree exists yet. When you add code, keep it under a clear top-level directory (e.g., `src/`), and document it in `CLAUDE.md`.

## Build, Test, and Development Commands

No build or test commands are defined in this template. Add canonical commands to `CLAUDE.md` once the tech stack is chosen. Example pattern:

```bash
# format
<format command>
# test
<test command>
# run
<dev server command>
```

## Coding Style & Naming Conventions

Coding standards are defined in `CLAUDE.md` and are intended to be project-specific. Update that file with:

- indentation and formatting rules
- lint/format tools
- naming conventions per language

Until updated, there is no enforced style beyond the template’s placeholders.

## Testing Guidelines

Testing philosophy and tooling must be defined in `CLAUDE.md`. The template expects mocked database interactions and clear unit/integration separation, but details are not yet set.

## Commit & Pull Request Guidelines

Commit message conventions are not documented in this repo. If you adopt one (e.g., Conventional Commits), record it in `CLAUDE.md`.

Pull request expectations are defined in `CLAUDE.md` and include:

- PR title format: `feat|fix|refactor|docs|test|chore(scope): description`
- small, focused PRs (generally ≤ ~300 lines changed)
- run lint/typecheck/tests before pushing
- use the PR template if/when `.github/PULL_REQUEST_TEMPLATE.md` is added

## Agent-Specific Instructions

When using an AI coding tool, follow `docs/workflow.md`:

- write PRDs using `docs/prd-template.md`
- break work into phases and waves
- write wave plans in `docs/plans/`
