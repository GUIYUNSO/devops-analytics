---
name: eia-planner
description: "Convert analysis findings into prioritized, actionable recommendations."
---

# EIA Action Planner

Turn reasoning output into concrete actions. Only called when the user wants recommendations (optimize, report, investigate, predict intents) or when monitor intent has risk_score > 0.6 or trend triggers_planner = true.

## Input

From `eia-reason`: findings from risk/rca/trend analysis.
From orchestrator: user role (to tailor recommendations).

## Action Rules

Match findings to actions using these exact tables. Do not invent actions outside these tables.

### From Risk Findings

| Finding | Action |
|---------|--------|
| PR too large (>500 LOC) | Split into smaller PRs, each focused on one concern |
| Single reviewer on high-risk PR | Add reviewer with module expertise |
| Review too fast (<5 min) | Request re-review with specific focus areas |
| Module churn too high (>10 commits/week) | Plan module refactor or split |
| Low bus factor (1 person owns module) | Knowledge transfer, add second owner |

### From RCA Findings

| Finding | Action |
|---------|--------|
| code_defect | Add regression test for the specific scenario |
| config_change | Add config validation, canary deployment |
| dependency | Pin dependency version, add compatibility test |
| infrastructure | Add monitoring alert for the specific resource |
| process_gap (no review) | Add CODEOWNERS, require reviews |
| process_gap (no test) | Add integration test for the affected path |

### From Trend Findings (only for critical significance)

| Finding | Action |
|---------|--------|
| Bug count critical increase (>50%) | Review PR sizes, increase test coverage for top bug modules |
| Lead time critical increase (>50%) | Identify bottleneck stage, remove queue |
| PR size critical increase (>50%) | Enforce 400 LOC PR limit |
| Deploy frequency critical decrease (>50%) | Automate deployment pipeline |
| Review time critical increase (>50%) | Add reviewers, set 24h review SLA |

### From Data Gaps

When reason output contains `data_gap` fields, generate infrastructure actions:

| Data Gap | Action | Tier |
|----------|--------|------|
| no_bug_tracker | Integrate GitHub Issues or external error tracker (e.g., Sentry) to enable RCA analysis | later |
| no_deploy_data | Add deployment tracking (e.g., GitHub Actions deployment status, ArgoCD) to enable lead time calculation | later |
| no_review_data | Ensure PR reviews are recorded (check if reviews exist but aren't captured by API) | important |
| no_ci_data | Add CI status tracking to correlate build failures with commits | later |

These actions address root cause (missing infrastructure) rather than symptoms. Always include data gap actions in the output, even if other actions exist.

## Prioritization (exact rules)

Categorize each action into exactly one tier:

| Tier | Criteria | Action |
|------|----------|--------|
| **quick_win** | effort=low AND impact=high | Do first this week |
| **important** | effort=medium OR impact=high | Plan for this sprint |
| **later** | effort=high AND impact=medium/low | Backlog |

Impact assessment:
- **high**: addresses a finding with risk_score > 0.8 OR trend significance = critical
- **medium**: addresses a finding with risk_score 0.6–0.8 OR trend significance = notable
- **low**: addresses a finding with risk_score < 0.6

Effort assessment:
- **low**: < 1 hour of work (add reviewer, add CODEOWNERS, split PR)
- **medium**: 1–8 hours (add integration test, refactor small module)
- **high**: > 8 hours (full module refactor, pipeline rewrite)

## Role-Based Filter

Remove actions that don't match the user's role:

| Role | Include | Exclude |
|------|---------|---------|
| developer | PR splits, code reviews, test additions, CODEOWNERS | pipeline automation, infra changes |
| devops | pipeline automation, deploy frequency, infra monitoring | individual PR advice, code-level fixes |
| pm | timeline improvements, bottleneck removal, sprint process | technical implementation details |
| cto | team-level patterns, cross-repo strategic actions | specific file/commit recommendations |

## Output

```json
{
  "summary": "one-line overall assessment",
  "quick_wins": [
    "action + evidence"
  ],
  "important": [
    "action + evidence"
  ],
  "later": [
    "action + evidence"
  ],
  "all_actions": [
    {
      "title": "action title",
      "tier": "quick_win | important | later",
      "category": "pr | test | code | process",
      "why": "evidence from analysis",
      "impact": "high | medium | low",
      "effort": "low | medium | high"
    }
  ]
}
```

Pass to `eia-report`.
