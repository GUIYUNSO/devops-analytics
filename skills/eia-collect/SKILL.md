---
name: eia-collect
description: "Collect development events from git history and GitHub API. Normalizes into a unified event array."
---

# EIA Collect

Gather development events for the specified scope.

## Input

From orchestrator:
- `repos`: list of repos (default: current directory)
- `time_range`: start/end or relative (default: 30 days)
- `event_types`: which events to collect (default: all available)
- `urgency`: realtime / high / medium / low (default: medium)

## Data Sources

### Tier 1: Git (always works)

Replace `{time_range}` with the actual value from input (e.g., "30 days ago", "7 days ago", "2026-06-01").

```bash
# Commits with file stats
git log --since="{time_range}" --pretty=format:"%H|%an|%ai|%s" --numstat

# Contributor summary
git shortlog -sn --since="{time_range}"

# File churn (times each file was changed)
git log --since="{time_range}" --format="" --name-only | sort | uniq -c | sort -rn
```

If `urgency=realtime`, limit to `--max-count=20` on the git log commands.

### Tier 2: GitHub (needs GITHUB_TOKEN)

```bash
# PRs
gh api repos/{owner}/{repo}/pulls?state=all&sort=updated&per_page=100

# Issues
gh api repos/{owner}/{repo}/issues?state=all&sort=updated&per_page=100

# Reviews (for a specific PR)
gh api repos/{owner}/{repo}/pulls/{number}/reviews
```

### Tier 3: External (only if tokens configured)

Jira, Sentry — collect only if explicitly configured.

## Unified Event Schema

Every event becomes:

```
{
  id:        string   — unique ID
  type:      string   — commit | pr | issue | review | build | deploy | bug
  actor:     string   — who
  repo:      string   — which repo
  timestamp: string   — ISO8601
  payload:   object   — type-specific fields
}
```

### Payload Fields

- **commit**: `{ sha, message, files_changed, additions, deletions }`
- **pr**: `{ number, title, state, author, reviewers[], size_loc, opened_at, merged_at }`
- **issue**: `{ key, title, state, assignee, priority, created_at, resolved_at }`
- **review**: `{ pr_number, reviewer, state, comment_count }`

## Enrichment (inline, not a separate skill)

While collecting, also compute:

- **Module mapping**: file path prefix → module name (`src/payment/handler.go` → `src/payment`)
- **Churn score**: per-file change frequency in the time range
- **Ownership**: per-module top contributors by commit count
- **Bug rate**: per-module bugs per 100 commits

## Output

```json
{
  "events": [...],
  "modules": {
    "src/payment": { "commits": 12, "contributors": ["alice"], "churn": 0.85, "bug_rate": 3.2 }
  },
  "developers": {
    "alice": { "commits": 45, "prs": 12, "bugs_introduced": 2 }
  },
  "summary": {
    "total_events": 234,
    "time_range": "2026-06-05 to 2026-07-05",
    "repos": ["backend"]
  }
}
```

Pass to `eia-build`.
