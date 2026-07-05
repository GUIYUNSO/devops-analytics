---
name: eia-build
description: "Build lifecycle timelines and entity-relationship graph from collected events."
---

# EIA Build

Transform events into two structures: timelines (temporal chains) and a knowledge graph (entity relationships).

## Input

From `eia-collect`: events array + modules + developers.

## Part 1: Timelines

Link events into lifecycle chains using shared identifiers.

### Linking Rules

```
Issue key in commit message → commit linked to issue
PR references commit SHA → PR linked to commits
Deploy contains commit SHA → deploy linked to PR
```

### Per-Chain Metrics

For each chain, compute:
- **Lead time**: first event → last event (issue created → deploy)
- **Cycle time**: first commit → deploy
- **Review time**: PR opened → approved
- **Stage durations**: time spent in each phase

### Aggregate

Across all chains:
- Median/avg/p90 lead time
- Which stage is the bottleneck (longest average duration)

## Part 2: Knowledge Graph

### Entities

Developer, Module, PR, Commit, Issue, Bug, Deploy

### Edges

```
Developer → authored → Commit
Developer → opened → PR
Developer → reviewed → PR
Developer → owns → Module (by commit share)
PR → contains → Commit
PR → resolved → Issue
PR → triggered → Deploy
PR → introduced → Bug
Commit → touched → Module
Module → has → Bug
Bug → caused_by → Commit
```

### Module = file path prefix

`src/payment/handler.go` → module `src/payment`

### Uncategorized Files

Files that don't match any known module prefix go into the `__infrastructure__` bucket:

- Root-level files: `Makefile`, `pyproject.toml`, `package.json`, `requirements.txt`
- Config files: `.github/`, `.gitignore`, `.gitattributes`, `docker*`
- Docs: `docs/`, `*.md` at root
- Meta: `skills/`, `tests/` (if not part of a named module)

This ensures LOC from large release/config PRs (e.g., PRs touching 100+ files across many directories) are not lost from churn calculations. Always include `__infrastructure__` in module churn reports.

### Graph Queries Available

After building, the graph supports:
- **Neighbors**: given entity, find all connected
- **Path**: find path between two entities
- **Count**: edges by type per entity (e.g., bugs per module)

## Output

```json
{
  "timelines": [
    {
      "root": { "type": "issue", "id": "JIRA-123" },
      "chain": [ { "type": "commit", "timestamp": "...", "actor": "alice" } ],
      "metrics": { "lead_time": "3d 2h", "cycle_time": "1d 6h", "review_time": "18h" }
    }
  ],
  "graph": {
    "nodes": [ { "id": "dev:alice", "type": "Developer" } ],
    "edges": [ { "src": "dev:alice", "dst": "mod:src/payment", "type": "owns" } ]
  },
  "aggregates": {
    "median_lead_time": "4.2d",
    "bottleneck": "review",
    "total_chains": 15
  }
}
```

Pass to `eia-reason`.
