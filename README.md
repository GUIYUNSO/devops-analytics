# Engineering Intelligence Agent (EIA)

**V3 planning + V1 execution** — an adaptive DevOps analytics plugin for Codex.

EIA transforms raw development events into actionable engineering intelligence. Given a question about your codebase, it collects data, builds timelines and knowledge graphs, reasons across multiple dimensions, plans actions, and delivers a concise answer — all in a single linear pipeline.

---

## Quick Start

### Install

`ash
codex plugin add devops-analytics@personal
`

Then start a new thread and ask:

> *"分析这个仓库的主要风险"*
> *"Why have bugs been increasing lately?"*
> *"给这周的工程效率做个报告"*

### Install from GitHub

Add a marketplace pointing to \GUIYUNSO/devops-analytics\, then:

`ash
codex plugin add devops-analytics@<marketplace-name>
`

---

## Architecture

\\\
User question
    │
    ▼
┌─ eia (orchestrator) ──────────────────────────────┐
│  infer profile (role, urgency, language)           │
│  classify intent (6 types)                         │
│  read memory → select pipeline                     │
└────────────────────────────────────────────────────┘
    │
    ▼
┌─ eia-collect ─────────────────────────────────────┐
│  git log / gh api                                  │
│  normalize to unified event schema                 │
│  compute modules, ownership, churn                 │
└────────────────────────────────────────────────────┘
    │
    ▼
┌─ eia-build ───────────────────────────────────────┐
│  link events into lifecycle timelines              │
│  build entity-relationship graph                   │
│  compute aggregates (lead time, bottleneck)        │
└────────────────────────────────────────────────────┘
    │
    ▼
┌─ eia-reason ──────────────────────────────────────┐
│  risk: score PRs & modules                         │
│  rca:  trace bug → deploy → PR → commit            │
│  trend: compare periods, detect shifts              │
└────────────────────────────────────────────────────┘
    │
    ▼
┌─ eia-planner ─────────────────────────────────────┐
│  match findings to action rules                     │
│  prioritize by impact/effort                        │
│  tailor to user role                                │
└────────────────────────────────────────────────────┘
    │
    ▼
┌─ eia-report ──────────────────────────────────────┐
│  compose in user's language                         │
│  conversation or report format                      │
│  append pipeline trace + write memory               │
└────────────────────────────────────────────────────┘
\\\

---

## 6 Skills

| Skill | Role |
|-------|------|
| eia | Orchestrator — intent routing, profile inference, pipeline selection |
| eia-collect | Data collection from git/GitHub, event normalization |
| eia-build | Timeline construction, knowledge graph generation |
| eia-reason | Unified reasoning: risk scoring, RCA, trend detection |
| eia-planner | Action planning from analysis findings |
| eia-report | Response composition in user's language |

### Intent Routing

| Intent | Pipeline | Plan? |
|--------|----------|-------|
| debug | collect → build → reason(rca) → report | Skip |
| investigate | collect → build → reason(all) → plan → report | Run |
| monitor | collect → build → reason(risk) → [plan →] report | If risk > 0.6 |
| predict | collect → build → reason(risk+trend) → plan → report | Run |
| report | collect → build → reason(trend) → plan → report | Run |
| optimize | collect → build → reason(all) → plan → report | Run |

Conflict: investigate > debug > optimize > predict > report > monitor

---

## Configuration

| Variable | Purpose |
|----------|---------|
| \GITHUB_TOKEN\ | Enables GitHub API for PR/issue/review data |

Git data works out of the box. Memory is stored at \~/.eia/memory.md\ (auto-created).

---

## Usage

| Example | Pipeline |
|---------|----------|
| \为什么支付模块最近总是超时？\ | collect → build → reason(rca) → report |
| \Why have PR review times increased?\ | collect → build → reason(all) → plan → report |
| \Generate a weekly engineering report\ | collect → build → reason(trend) → plan → report |

---

## License

MIT
