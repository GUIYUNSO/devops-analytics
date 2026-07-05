# EIA V4 Architecture Reference

## Design Philosophy

**V3 planning + V1 execution.**

- Decision layer: intent routing, profile inference, constrained pipeline selection (V3 style)
- Execution layer: linear skill chain, direct data sources, markdown memory (V1 style)

## Pipeline (linear, not DAG)

```
User question
    │
    ▼
eia (orchestrator)
  ├─ infer profile (role, urgency, language)
  ├─ classify intent (debug/investigate/monitor/predict/report/optimize)
  ├─ read memory (~/.eia/memory.md, last 20 lines)
  ├─ select pipeline based on intent
    │
    ▼
eia-collect
  ├─ git log / gh api
  ├─ normalize events
  ├─ compute modules, ownership, churn
    │
    ▼
eia-build
  ├─ link events into timelines
  ├─ build entity graph
  ├─ compute aggregates (lead time, bottleneck)
    │
    ▼
eia-reason (mode selected by intent)
  ├─ risk: score PRs/modules
  ├─ rca: trace bug → deploy → PR → commit
  ├─ trend: compare periods, detect shifts
    │
    ▼ (if intent requires actions)
eia-planner
  ├─ match findings to action rules
  ├─ prioritize by impact/effort
  ├─ tailor to user role
    │
    ▼
eia-report
  ├─ compose in user's language
  ├─ format (conversation or report)
  ├─ append pipeline trace
  └─ write memory line
```

## Intent → Pipeline Mapping

| Intent | Skills Called | Skip |
|--------|-------------|------|
| debug | collect → build → reason(rca) → report | planner |
| investigate | collect → build → reason(all) → planner → report | — |
| monitor | collect → build → reason(risk) → report | planner (unless issues found) |
| predict | collect → build → reason(risk+trend) → planner → report | — |
| report | collect → build → reason(trend) → planner → report | — |
| optimize | collect → build → reason(all) → planner → report | — |

## Urgency Modifiers

| Urgency | Modification |
|---------|-------------|
| realtime | Skip memory read, skip enrich details, minimal report |
| high | Skip memory, standard pipeline |
| medium | Full pipeline |
| low | Full pipeline, detailed report |

## IO Contracts

Every skill has explicit input/output:

### eia-collect
```
Input:  { repos[], time_range, event_types[] }
Output: { events[], modules{}, developers{}, summary{} }
```

### eia-build
```
Input:  { events[], modules{}, developers{} }
Output: { timelines[], graph{ nodes[], edges[] }, aggregates{} }
```

### eia-reason
```
Input:  { timelines[], graph{}, aggregates{}, intent }
Output: { risk{ findings[], summary }, rca{ findings[], summary }, trend{ findings[], summary } }
```

### eia-planner
```
Input:  { risk findings, rca findings, trend findings, user role }
Output: { summary, quick_wins[], actions[] }
```

### eia-report
```
Input:  { all pipeline outputs, user language, role, detail_level }
Output: { formatted response (string) }
```

## Memory Format

File: `~/.eia/memory.md`

```markdown
- [2026-07-05] repo=backend intent=investigate finding="payment module 3 bugs in 30 days"
- [2026-07-04] repo=frontend intent=report finding="lead time improved 15%"
```

- Append-only during session
- Read last 20 lines before analysis
- One line per analysis

## Profile Inference

Orchestrator infers from query vocabulary:

| Signal | Role |
|--------|------|
| PR, review, commit, merge | developer |
| deploy, pipeline, CI, infra | devops |
| sprint, backlog, milestone | pm |
| architecture, system, scale | cto |
| test, coverage, QA | qa |

Override: user says "as CTO" / "以PM视角" → use directly.

## Extension Pattern

To add a new analysis type (e.g., security):

1. Add mode to `eia-reason` SKILL.md (new section under "Mode: Security")
2. Add intent keywords to `eia` SKILL.md routing table
3. Add IO contract for the new mode
4. No other skill changes needed
