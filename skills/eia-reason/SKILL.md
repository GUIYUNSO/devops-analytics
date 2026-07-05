---
name: eia-reason
description: "Unified reasoning: risk scoring, root cause analysis, trend detection. Select mode based on intent."
---

# EIA Reason

The analysis engine. Runs one or more analysis modes based on the user's intent.

## Input

From `eia-build`: timelines + graph + aggregates.
From orchestrator: intent (determines which modes to run).

## Mode Selection

| Intent | Run These Modes |
|--------|----------------|
| debug | rca |
| investigate | rca + trend + risk |
| monitor | risk |
| predict | risk + trend |
| report | trend |
| optimize | all three |

---

## Mode: Risk

Score each PR and module for failure probability.

### Factors

| Factor | Source | How to Score |
|--------|--------|-------------|
| PR size | graph: PR.size_loc | >400 LOC = 0.7, >800 = 0.9 |
| Churn | modules: churn_score | Direct value |
| Review depth | timelines: review_time | <10 min = 0.8, <1h = 0.5 |
| Reviewer count | graph: PR.reviewers | 1 reviewer = 0.7 |
| History | modules: bug_rate | >2/100 commits = 0.8 |
| Bus factor | modules: ownership | 1 person = 0.6 |

### Context Adjustment (mandatory)

Before computing final score, apply these adjustments:

| Condition | Adjustment |
|-----------|-----------|
| Author is bot (dependabot, renovate, github-actions) | Multiply all scores by 0.5 |
| PR is auto-generated (title starts with "chore(deps)" or "build(deps)") | Multiply all scores by 0.5 |
| PR is docs-only (only .md/.txt/.yaml changes) | Set score to 0.1 regardless of size |
| Module has 0 commits in last 90 days | Set churn factor to 0 |

### Score

Weighted average → risk_score (0–1)

| Score | Level | Meaning |
|-------|-------|---------|
| 0–0.3 | Low | Safe to merge |
| 0.3–0.6 | Medium | Consider extra review |
| 0.6–0.8 | High | Require 2+ reviewers |
| 0.8–1.0 | Critical | Block or senior review required |

### Output

For each high/critical target:
- risk_score, risk_level
- top 2–3 contributing factors
- possible failure scenario
- suggested action

---

## Mode: Root Cause Analysis

For each bug/alert in the dataset, trace backward.

### Steps (with fallback at each step)

**Step 1**: Find the deploy where the bug first appeared.
- Fallback: If no deploy data available, use the commit timestamp as proxy. Set `deploy_link: false`.

**Step 2**: List PRs merged between previous deploy and this deploy.
- Fallback: If no deploy chain, list all PRs merged in the 7 days before the bug first_seen. Set `deploy_chain: approximate`.

**Step 3**: Find which PR touched the same module as the bug.
- Fallback: If no module match found, list all PRs by the same author in the same time window. Set `match_confidence: low`.

**Step 4**: Identify the specific commit that changed the relevant code.
- Fallback: If no commit linked, identify the PR as the most likely source without pinpointing a commit. Set `commit_link: false`.

**Step 5**: Classify root cause.

| Category | Signal |
|----------|--------|
| code_defect | Logic error, missing check |
| config_change | Env var, feature flag changed |
| dependency | Third-party lib update |
| infrastructure | Resource, network, disk |
| process_gap | No review, no test, no monitoring |

If classification is uncertain, use `code_defect` as default and note the uncertainty.

### Chain Break Rules

At any step, if the trace breaks:
1. **Do not guess** the missing link
2. Output what you found with explicit `data_gap` fields
3. Mark `confidence: low` on the root cause
4. In the summary, state: "Trace incomplete — missing {what's missing}. Root cause is probable, not confirmed."

### Output

For each analyzed bug:
- root_cause_commit (or null + data_gap)
- root_cause_pr (or null + data_gap)
- root_cause_author
- category, explanation (1–2 sentences)
- contributing_factors (what allowed this to happen)
- time_to_detect, time_to_fix
- confidence: high | medium | low

---

## Mode: Trend

Compare current period vs previous period.

### Metrics

| Metric | Compute |
|--------|---------|
| Bug count | Total bugs this period vs last |
| Lead time | Median lead time this period vs last |
| PR size | Median LOC this period vs last |
| Deploy frequency | Deploys per week |
| Review time | Median time from PR open to approve |

### Significance

- <10% change → stable (no action needed)
- 10–50% change → notable (flag for attention)
- >50% change → critical (triggers planner)

### Hypothesis

For notable/critical degradations, use the graph to find:
- Which modules are driving the change?
- Is it correlated with specific developers or team changes?
- Did a process change coincide?

### Merged PR Review History

When trend analysis finds a critical review metric (e.g., review coverage = 0%), also check **already merged PRs** for the same pattern:

1. Query merged PRs in the same time range
2. Check if they had reviews before merge
3. Compare: is the current pattern (no reviews) a new regression, or a long-standing project culture?

This distinction matters:
- **New regression** → "Review coverage dropped this week, likely due to sprint pressure" → action: add reviewers
- **Long-standing culture** → "Project has historically merged without reviews, this is not new" → action: establish review process from scratch

Include this comparison in the trend output as a `review_culture` field:
```json
"review_culture": {
  "open_prs_review_rate": 0.0,
  "merged_prs_review_rate": 0.0,
  "assessment": "long_standing | new_regression | insufficient_data"
}
```

### Planner Trigger

If ANY metric has significance = critical (>50% change), set `triggers_planner: true` in the output. The orchestrator uses this to decide whether to call planner for monitor intent.

### Output

- metrics with current/previous/delta/direction/significance
- triggers_planner: true | false
- top 3 insights (natural language)
- DORA comparison if delivery metrics available

---

## Output

All modes produce:

```json
{
  "modes_run": ["risk", "rca", "trend"],
  "risk": { "findings": [...], "summary": "..." },
  "rca": { "findings": [...], "summary": "..." },
  "trend": { "findings": [...], "summary": "...", "triggers_planner": false }
}
```

Pass to `eia-planner` (if intent requires actions) or `eia-report`.
