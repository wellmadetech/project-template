---
name: bug-fix
description: Automate complete bug lifecycle - triage, reproduce, investigate, plan, fix, verify, document RCA
args: issue_number [--dry-run]
---

# Bug Fix Automation Skill

Automates: triage → reproduce → investigate → plan → fix → verify → document.

**Invocation:**
- `/bug-fix 278` - Full execution
- `/bug-fix 278 --dry-run` - Simulation mode (no writes, no PR, no comments)

---

## Dry Run Mode

When `--dry-run` is passed:

| Action | Normal | Dry Run |
|--------|--------|---------|
| Fetch issue | Execute | Execute (read-only) |
| Navigate browser | Execute | Execute (read-only) |
| Take screenshots | Execute | Execute (save locally only) |
| Capture console/network | Execute | Execute |
| Write state file | Execute | **SKIP** - print what would be written |
| Post issue comment | Execute | **SKIP** - print comment content |
| Add labels | Execute | **SKIP** - print labels |
| Create plan file | Execute | **SKIP** - print content |
| Create worktree | Execute | **SKIP** - print command |
| Write code changes | Execute | **SKIP** - print diff |
| Create PR | Execute | **SKIP** - print PR body |
| Merge PR | Execute | **SKIP** |
| Create RCA | Execute | **SKIP** - print content |
| Close issue | Execute | **SKIP** |

**Dry run output format:**
```
[DRY-RUN] Would execute: gh issue edit 278 --add-label "severity:high,area:aperture"
[DRY-RUN] Would create file: docs/plans/2026-01-28-fix-278-aperture.md
[DRY-RUN] Content:
---
{file content here}
---
```

To test: `/bug-fix 278 --dry-run`

---

## Phase 1: Triage

### 1.1 Fetch Issue
```bash
gh issue view {issue_number} --repo wellmadetech/m360web --json title,body,labels,comments,state
```

### 1.2 Initialize State
Create `.bug-fix/state.json`:
```json
{
  "issue_number": {issue_number},
  "repo": "wellmadetech/m360web",
  "original_text": "{VERBATIM_ISSUE_BODY}",
  "current_phase": "triage",
  "phases": {}
}
```

**CRITICAL:** Save `original_text` verbatim. Never modify it.

### 1.3 Parse Issue
Extract from body:
- **Affected area:** aperture|products|forge|auth|surveys|config
- **Steps to reproduce:** numbered list or description
- **Expected vs actual:** behavior difference
- **Environment:** staging|production|local

### 1.4 Assess Severity
| Condition | Severity |
|-----------|----------|
| Data loss, security, production down | critical |
| Core feature broken, no workaround | high |
| Feature broken, workaround exists | medium |
| Minor UI, edge case | low |

### 1.5 Decision Point
If issue lacks details:
> "Issue lacks reproduction steps. Options: (1) Request more info via comment, (2) Attempt discovery. Choose?"

Update state:
```json
"phases.triage": {
  "status": "done",
  "affected_area": "{AREA}",
  "severity": "{SEVERITY}",
  "has_repro_steps": true|false
}
```

---

## Phase 2: Reproduction

### 2.1 Load URL Map
Read `references/url-map.json` to get:
- `_base_urls.staging` - Main staging URL
- `_base_urls.pr_preview` - PR preview URL pattern
- Area paths (e.g., `aperture.main` → `/aperture`)

### 2.2 Select Base URL
**Priority order:**
1. **Main staging:** Use `_base_urls.staging` from url-map.json
2. **PR preview:** Only if testing a specific PR

**IMPORTANT:** Do NOT guess staging URLs. Always read from `references/url-map.json`.

### 2.3 Navigate to Area
```
mcp__plugin_playwright_playwright__browser_navigate url={BASE_URL}{PATH}
```

### 2.4 Handle Authentication

If login screen appears (magic link flow):

1. **Ask user for email:**
   > "Authentication required. What email should I use for the magic link?"

   (Default suggestion from `_auth.suggested_email` in url-map.json)

2. **Request magic link:**
   - Enter the user's email
   - Click "Send Magic Link"

3. **Prompt user for link:**
   > "Magic link sent to {email}. Please check your email and paste the link here:"

4. **Navigate to magic link URL:**
   ```
   mcp__plugin_playwright_playwright__browser_navigate url={PASTED_LINK}
   ```

5. **Verify authentication:**
   - Check for user menu / logout button
   - If still on login, retry once then escalate

**Note:** This requires two user interactions: (1) provide email, (2) paste magic link.

### 2.5 Follow Reproduction Steps
For each step:
1. Perform action (click, type, navigate)
2. Capture screenshot: `mcp__plugin_playwright_playwright__browser_take_screenshot`
3. Save to `.bug-fix/screenshots/step-{N}.png`

### 2.6 Capture Diagnostics
**Console messages (all levels):**
```
mcp__plugin_playwright_playwright__browser_console_messages level="debug"
```

**Network requests (include failures):**
```
mcp__plugin_playwright_playwright__browser_network_requests includeStatic=false
```

Look for:
- Console errors/warnings
- Failed network requests (4xx, 5xx)
- Slow responses (>2s)
- Missing telemetry/logging endpoints

### 2.7 Document Observability Gaps
Note what would help future debugging:
- Missing error boundaries
- No click tracking
- No performance metrics
- Silent failures (catch blocks without logging)

### 2.8 Retry Logic
- Max 3 reproduction attempts
- Vary timing, refresh between attempts
- If failed 3x: escalate

**Decision Point (failed 3x):**
> "Cannot reproduce after 3 attempts. Options: (1) Escalate to reporter, (2) Try different environment, (3) Mark as cannot-reproduce. Choose?"

Update state:
```json
"phases.reproduction": {
  "status": "done",
  "attempts": 2,
  "reproduced": true,
  "screenshots": ["step-1.png", "step-2.png"],
  "console_errors": [...],
  "network_failures": [...],
  "observability_gaps": ["No click tracking on sidebar"]
}
```

---

## Phase 3: Update Issue

**IMPORTANT:** Update the issue title AND body directly, not just add a comment.

### 3.1 Update Issue Title
Create a clear, descriptive title:
```bash
gh issue edit {issue_number} --title "{AREA}: {CLEAR_DESCRIPTION} (was: {ORIGINAL_TITLE})"
```

Example:
- Original: "Aperture redirection"
- Updated: "Aperture: Clicking image redirects to /products instead of opening viewer (was: Aperture redirection)"

### 3.2 Update Issue Body
Replace the body with structured content while preserving the original:
```bash
gh issue edit {issue_number} --body-file .bug-fix/issue-body.md
```

The body should include:
1. **Summary** - Clear 1-2 sentence description
2. **Original Report** - In `<details>` block (verbatim from state)
3. **Reproduction Steps** - Numbered steps
4. **Expected vs Actual** - Clear comparison
5. **Diagnostics** - Console errors, network failures
6. **Classification** - Severity, area, type
7. **Screenshots** - Embedded if possible

Use `templates/issue-update.md` as the body template.

### 3.3 Add Labels
```bash
gh issue edit {issue_number} --add-label "severity:{SEVERITY},area:{AREA},type:bug"
```

Note: If labels don't exist in the repo, skip this step.

### 3.4 Post Investigation Comment (Optional)
If additional context is needed beyond the body update:
```bash
gh issue comment {issue_number} --body "Additional diagnostics: ..."
```

Update state:
```json
"phases.issue_update": {
  "status": "done",
  "title_updated": true,
  "body_updated": true,
  "labels_added": ["severity:high", "area:aperture", "type:bug"]
}
```

---

## Phase 4: Investigation

### 4.1 Identify Entry Points
Based on affected area, find:
- Route definitions
- Component files
- API endpoints
- Event handlers

Use `references/url-map.json` and `references/selectors.json` for hints.

### 4.2 Trace Code Path
```bash
# Find relevant files
Grep pattern="{COMPONENT_NAME}" path="src/web"
Grep pattern="{API_ENDPOINT}" path="src/api-nest"

# Read and trace
Read file_path="{FILE}"
```

### 4.3 Find Root Cause
Look for:
- Missing null checks
- Race conditions
- Incorrect state updates
- API contract mismatches
- Missing error handling

### 4.4 Document Affected Files
List all files that need changes:
```json
"phases.investigation": {
  "status": "done",
  "root_cause": "Missing onClick handler on sidebar product cards",
  "affected_files": [
    "src/web/src/features/aperture/ImageViewer.tsx",
    "src/web/src/features/aperture/components/ProductSidebar.tsx"
  ],
  "missing_error_handling": ["ProductSidebar.tsx:45 - no error boundary"]
}
```

---

## Phase 5: Planning

### 5.1 Create Plan File
Path: `docs/plans/{YYYY-MM-DD}-fix-{issue_number}-{slug}.md`

Include:
- Issue link
- Root cause summary
- Files to modify
- Implementation steps
- Test requirements

### 5.2 Multi-Service Check
If fix spans multiple services (web, api-nest, api):
- Create separate implementation issues
- Plan separate PRs per service
- Define merge order

Update state:
```json
"phases.planning": {
  "status": "done",
  "plan_file": "docs/plans/2026-01-28-fix-278-aperture-redirect.md",
  "implementation_issues": [],
  "services_affected": ["web"]
}
```

---

## Phase 6: Implementation

### 6.1 Create Worktree
```bash
git worktree add ../m360web-fix-{issue_number} -b fix/{issue_number}-{slug}
cd ../m360web-fix-{issue_number}
```

### 6.2 Implementation Review Cycle

**FOLLOW THIS CYCLE EXACTLY:**

```
┌─────────────────────────────────────────────────┐
│  IMPLEMENTATION REVIEW CYCLE                    │
├─────────────────────────────────────────────────┤
│  1. Write fix code                              │
│  2. Codex CLI review (`codex review`)           │
│  3. Fix Codex issues                            │
│  4. Claude review (code patterns)               │
│  5. Fix Claude issues                           │
│  6. Codex review (second pass)                  │
│  7. Claude review (second pass)                 │
│  8. Greptile skill (`/review-pr` local)         │
│  9. Fix Greptile skill issues                   │
│ 10. Local E2E test (`npm run test:headed`)      │
│ 11. ──► If issues, repeat from step 1           │
│ 12. Push PR                                     │
│ 13. Wait for deployment                         │
│ 14. Check Railway deployment logs               │
│ 15. Check E2E CI logs                           │
│ 16. Wait for Greptile GitHub app review         │
│ 17. Respond to Greptile app comments            │
│ 18. Respond to all other PR comments            │
│ 19. Resolve all conversations                   │
│ 20. ══► STOP: Wait for human PR review          │
│ 21. After approval: Merge                       │
└─────────────────────────────────────────────────┘
```

### 6.3 Review Commands

**Codex review:**
```bash
codex review
```

**Claude review:** Self-review for:
- CLAUDE.md coding standards
- Project patterns
- Security (OWASP top 10)
- Type safety

**Local E2E:**
```bash
cd src/web && npm run test:headed
```

### 6.4 Create PR
```bash
gh pr create --title "fix(area): description (#issue_number)" \
  --body "Fixes #issue_number\n\n## Summary\n{SUMMARY}\n\n## Test plan\n- [ ] E2E passes\n- [ ] Manual verification"
```

### 6.5 Post-Push Verification
```bash
# Wait for deployment
gh pr view {PR#} --json statusCheckRollup

# Check Railway logs
railway logs --build

# Check E2E CI
gh run list --workflow=e2e.yml --limit=1
```

### 6.6 Greptile GitHub App
After push, Greptile GitHub app reviews automatically.
- Wait for review comments
- Address each comment
- Resolve conversations

### 6.7 Human Review Gate

**STOP HERE. DO NOT PROCEED UNTIL HUMAN APPROVES.**

> "PR #{PR_NUMBER} ready for human review. Waiting for approval before merge."

Update state:
```json
"phases.implementation": {
  "status": "awaiting_review",
  "prs": [{"service": "web", "number": 290}],
  "review_cycle": {
    "iteration": 1,
    "codex_review_1": "done",
    "claude_review_1": "done",
    "codex_review_2": "done",
    "claude_review_2": "done",
    "greptile_skill": "done",
    "local_e2e": "done",
    "pushed": true,
    "deployment_logs_checked": true,
    "ci_e2e_checked": true,
    "greptile_app_review": "done",
    "comments_resolved": true,
    "human_review": "pending"
  }
}
```

### 6.8 After Approval
```bash
gh pr merge {PR#} --squash --delete-branch
```

---

## Phase 7: Post-Merge Verification

### 7.1 Verify on Staging
Navigate to staging URL and verify bug no longer reproduces:
```
mcp__plugin_playwright_playwright__browser_navigate url="https://proxy-m360web-pr-{PR#}.up.railway.app{PATH}"
```

Re-run reproduction steps. Capture "fixed" screenshots.

### 7.2 Create RCA Document
Path: `docs/rca/{YYYY-MM-DD}-issue-{issue_number}.md`

Use `templates/rca-document.md`:
- Timeline from report to resolution
- Root cause analysis
- Fix summary
- Prevention measures
- Observability improvements made

### 7.3 Post RCA as Issue Comment
Post FULL RCA text to issue (not just a link):
```bash
gh issue comment {issue_number} --body-file docs/rca/{RCA_FILE}
```

### 7.4 Close Issue
```bash
gh issue close {issue_number} --comment "Fixed in PR #{PR_NUMBER}. See RCA above."
```

Update state:
```json
"phases.verification": {
  "status": "done",
  "staging_verified": true,
  "rca_file": "docs/rca/2026-01-28-issue-278.md",
  "rca_comment_posted": true,
  "issue_closed": true
}
```

---

## State Management

### State File Location
`.bug-fix/state.json` in repo root.

### Locking
Create `.bug-fix/state.lock` before updates. Remove after.

### Resume Support
On skill invocation, check for existing state:
- If exists and same issue: resume from `current_phase`
- If exists and different issue: prompt to reset or abort
- If no state: start fresh

---

## Error Handling

| Error | Action |
|-------|--------|
| Issue not found | Exit with error message |
| Cannot reproduce | Escalate after 3 attempts |
| PR CI fails | Fix and re-push (stay in implementation) |
| Deployment fails | Check Railway logs, fix, redeploy |
| Greptile timeout | Proceed, note in state |

---

## Quick Reference

### Commands
```bash
# Triage
gh issue view {N} --repo wellmadetech/m360web --json title,body,labels

# Labels
gh issue edit {N} --add-label "severity:high,area:aperture"

# Comment
gh issue comment {N} --body "message"

# PR
gh pr create --title "fix(area): desc" --body "Fixes #N"
gh pr view {N} --json statusCheckRollup
gh pr merge {N} --squash

# Railway
railway logs --build
railway logs
```

### URLs
- Main staging: `https://proxy-staging-2b4c.up.railway.app` (from url-map.json)
- PR preview: `https://proxy-m360web-pr-{PR#}.up.railway.app`
- **Always read from `references/url-map.json` - do not hardcode**

### File Paths
- State: `.bug-fix/state.json`
- Screenshots: `.bug-fix/screenshots/`
- Plans: `docs/plans/`
- RCAs: `docs/rca/`
