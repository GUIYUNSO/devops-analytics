---
name: eia-report
description: "Compose final response in user's language. Supports conversation and report formats."
---

# EIA Report

Format upstream results into a clean, concise, technically precise answer.

## Input

- Orchestrator: user language, role, detail level
- Pipeline: events, structures, findings, actions

## Language

Output matches input language. Chinese in → Chinese out. English in → English out.

---

## Conversation Format (default)

For debug / investigate / monitor / predict:

```
[一句话结论]

[关键数据 — 3 到 5 个，带数字]

[行动项 — if planner ran]

[跟进问题 — 1 个可选的下一步提问]
```

**示例（中文）**：

```
PR review 覆盖率断档是当前首要风险：22 个 open PR 中 19 个无人 review。

核心指标：
• 86% open PR 缺乏 human review（历史 merged PR review 率 60%，属临时积压）
• 4 个 PR > 400 LOC，最大 #270 达 1538 LOC
• tools/video churn 最高（30 天 5 次变更），无 CODEOWNERS

建议：
[P0] 为 #270、#297 分配 human reviewer
[P1] 为 tools/video 添加 CODEOWNERS

还想深入哪个方向？
```

**示例（English）**：

```
Review coverage gap is the primary risk: 19 of 22 open PRs lack human review.

Key metrics:
• 86% open PRs unreviewed (merged PR review rate is 60% — this is a bandwidth issue, not cultural)
• 4 PRs exceed 400 LOC; largest is #270 at 1,538 LOC
• tools/video has highest churn (5 changes in 30 days), no CODEOWNERS

Actions:
[P0] Assign human reviewers to #270 and #297
[P1] Add CODEOWNERS for tools/video

What would you like to dig into next?
```

---

## Report Format

For report intent or explicit request:

```markdown
# {repo} Engineering Report — {time_range}

## Summary
{One paragraph: overall health, top concern, key shift. Max 3 sentences.}

## Metrics

| Metric | Current | Prior | Δ | Status |
|--------|---------|-------|---|--------|
| Open PRs | 22 | — | — | — |
| Review coverage | 4.5% | 60% (merged) | -55.5pp | ⚠ Regression |
| PRs > 400 LOC | 4 | — | — | ⚠ |
| Top churn module | tools/video (5) | — | — | — |

## Findings

1. **Review bandwidth deficit** — 86% of open PRs lack human review. Merged PRs show 60% review rate, indicating this is a temporary capacity issue, not a cultural norm. [evidence: PR #270, #297, #262]

2. **Large PR concentration** — 4 PRs exceed 400 LOC. #270 (1,538 LOC) and #262 (1,367 LOC) are from external contributors with zero human review. [evidence: risk scores 0.82, 0.78]

3. **Missing infrastructure** — No bug tracker configured. RCA analysis unavailable. [data_gap: no_bug_tracker]

## Actions

| Priority | Action | Rationale |
|----------|--------|-----------|
| [P0] | Assign reviewers to #270, #297 | Risk score > 0.75, zero human review |
| [P1] | Add CODEOWNERS for tools/video, tools/audio | Highest churn modules, no ownership |
| [P1] | Prioritize review of #262 (security hardening) | External contributor + subprocess safety |
| [P2] | Set 400 LOC PR limit | Reduce batch size, improve review quality |
| [P2] | Integrate GitHub Issues or Sentry | Enable RCA analysis |
```

---

## Detail Levels

| Level | Output |
|-------|--------|
| **brief** | One sentence + top action |
| **normal** | Conversation format above |
| **detailed** | Report format + raw data appendix |

## Role Adjustments

Use orchestrator Step 4 include/exclude rules as single source of truth. Additional style notes:

| Role | Tone |
|------|------|
| developer | Precise: file paths, SHAs, line numbers |
| devops | Systemic: pipeline stages, deploy metrics |
| pm | Outcome-oriented: timelines, milestones, velocity |
| cto | Strategic: cross-repo patterns, team risks |
| qa | Quality-focused: coverage, escape rate, gates |

## Footer

Append a one-line trace (collapsed if possible):

```
── eia: collect → build → reason → plan → report │ role={role} intent={intent} ──
```

## Priority Levels

| Level | Meaning | Timeframe |
|-------|---------|-----------|
| [P0] | Blocks release or introduces security risk | This week |
| [P1] | Degrades quality or velocity | This sprint |
| [P2] | Improvement opportunity | Backlog |

## Wording Principles

- **Systemic, not personal** — describe risk patterns, not individual blame
- **Precise, not vague** — "86% unreviewed" not "many PRs lack review"
- **Actionable, not descriptive** — every finding pairs with a concrete next step
- **Honest about gaps** — "no bug tracker configured" not "data unavailable"

| Instead of | Write |
|-----------|-------|
| "某人有很多 PR" | "41% open PR 集中在单一贡献者，存在 bus factor 风险" |
| "bug 越来越多" | "bug count +50% QoQ，主要集中在 payment 模块" |
| "代码质量不好" | "模块 churn score 0.85，高于团队基线 3x" |

## Memory

Append after response:

```
- [{date}] repo={repo} intent={intent} finding="{one-line}" gaps=["{gap}"]
```

## Rules

1. Lead with conclusion, not process
2. Every number has context (not just "5 bugs" but "5 bugs, +67% vs prior period")
3. Tables for comparisons, bullets for lists
4. Never fabricate data — state gaps explicitly
5. Max 5 findings, max 5 actions — prioritize, don't enumerate
