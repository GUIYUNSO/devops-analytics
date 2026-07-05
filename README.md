<div align="center">

# 工程智能代理（EIA）

**Codex 自适应 DevOps 分析插件**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Plugin: Codex](https://img.shields.io/badge/Plugin-Codex-purple.svg)](https://github.com/GUIYUNSO/devops-analytics)
[![Version: 4.0](https://img.shields.io/badge/Version-4.0-green.svg)](https://github.com/GUIYUNSO/devops-analytics)

输入一个问题，自动完成：采集 → 构建 → 推理 → 规划 → 报告

</div>

---

## 快速开始

**安装**

```bash
# 添加 Git marketplace
codex plugin marketplace add GUIYUNSO/devops-analytics

# 安装插件
codex plugin add devops-analytics@guiyunso
```

**直接提问**

> *「分析这个仓库的主要风险」*
> *「为什么最近 bug 越来越多？」*
> *「给这周的工程效率做个报告」*

---

## 工作流程

```
用户提问
    │
    ▼
┌─ eia ────────────────┐
│  意图分类 + 角色推断    │
└──────────┬───────────┘
           │
    ┌──────┴──────┐
    │             │
    ▼             ▼
┌─────────┐ ┌─────────┐
│ collect │→│  build  │
│ 数据采集 │ │ 知识构建 │
└─────────┘ └─────────┘
           │
    ┌──────┴──────┐
    │             │
    ▼             ▼
┌─────────┐ ┌─────────┐
│  risk   │ │   rca   │
│ 风险评分 │ │ 根因分析 │
└─────────┘ └─────────┘
    │             │
    └──────┬──────┘
           ▼
    ┌──────────┐
    │ planner  │
    │ 行动规划  │
    └────┬─────┘
         ▼
    ┌──────────┐
    │  report  │
    │ 结果输出  │
    └──────────┘
```

---

## 核心能力

| 能力 | 说明 |
|------|------|
| **意图路由** | 根据问题自动选择分析路径，6 种意图覆盖全部场景 |
| **风险评分** | 基于 PR 大小、代码 churn、review 深度、历史缺陷多维打分 |
| **根因分析** | bug → deploy → PR → commit 反向追溯，定位引入变更 |
| **趋势检测** | 周期对比，识别指标异常波动并生成假设 |
| **行动规划** | 按 [P0/P1/P2] 优先级输出可执行建议 |
| **跨语言输出** | 中文提问中文回答，英文提问英文回答 |

---

## 意图路由

| 意图 | Pipeline | Planner |
|------|----------|---------|
| debug | collect → build → rca → report | 跳过 |
| investigate | collect → build → reason → plan → report | 执行 |
| monitor | collect → build → risk → report | risk > 0.6 时 |
| predict | collect → build → risk+trend → plan → report | 执行 |
| report | collect → build → trend → plan → report | 执行 |
| optimize | collect → build → reason → plan → report | 执行 |

冲突优先级：investigate > debug > optimize > predict > report > monitor

---

## 配置

| 变量 | 用途 |
|------|------|
| `GITHUB_TOKEN` | 启用 GitHub API 采集 PR/issue/review 数据 |

Git 数据开箱即用。记忆存储在 `~/.eia/memory.md`。

---

## 示例

| 提问 | Pipeline |
|------|----------|
| 「为什么支付模块最近总是超时？」 | rca → report |
| 「Why have PR review times increased?」 | reason → plan → report |
| 「生成本周工程效率报告」 | trend → plan → report |
| 「这个模块风险怎么样？」 | risk+trend → plan → report |

---

## 架构设计

**V3 规划 + V1 执行**：决策层用抽象能力（意图路由、角色推断、记忆），执行层用确定性（git 命令、GitHub API、线性 pipeline）。

- **6 个独立 Skill**：每个可独立调用，通过 `previous_output` 传递数据
- **显式 IO Contract**：每个 skill 定义输入输出格式，无隐式依赖
- **Markdown Memory**：跨会话记忆存储在 `~/.eia/memory.md`，append-only
- **冲突规则硬编码**：意图分类、优先级排序、跳过规则全部写死，无模糊判断

---

## 许可证

[MIT](LICENSE)
