# 工程智能代理（EIA）

**V3 规划 + V1 执行** —— Codex 自适应 DevOps 分析插件。

EIA 将原始开发事件转化为可执行的工程洞察。输入一个关于代码仓库的问题，它会自动采集数据、构建时间线和知识图谱、多维推理、规划行动，并以简洁的语言输出结果——全部在一个线性 pipeline 中完成。

---

## 快速开始

### 安装

```bash
codex plugin add devops-analytics@personal
```

安装后启动新会话，直接提问：

> *「分析这个仓库的主要风险」*
> *「为什么最近 bug 越来越多？」*
> *「给这周的工程效率做个报告」*

### 从 GitHub 安装

将 marketplace 指向 `GUIYUNSO/devops-analytics`，然后执行：

```bash
codex plugin add devops-analytics@<marketplace-name>
```

---

## 架构

```
用户提问
  │
  ▼
┌─ eia（编排器）───────────────────────────────────┐
│  推断用户角色、紧急度、语言                          │
│  分类意图（6 种类型）                               │
│  读取记忆 → 选择 pipeline                          │
└──────────────────────────────────────────────────┘
  │
  ▼
┌─ eia-collect（采集）─────────────────────────────┐
│  git log / GitHub API                             │
│  统一事件 Schema                                   │
│  计算模块归属、代码 ownership、变更频率               │
└──────────────────────────────────────────────────┘
  │
  ▼
┌─ eia-build（构建）───────────────────────────────┐
│  将事件链接为生命周期时间线                          │
│  构建实体关系图谱                                   │
│  计算聚合指标（lead time、瓶颈定位）                  │
└──────────────────────────────────────────────────┘
  │
  ▼
┌─ eia-reason（推理）──────────────────────────────┐
│  风险评分：为 PR 和模块打分                          │
│  根因分析：bug → deploy → PR → commit 反向追溯      │
│  趋势分析：周期对比、异常检测                         │
└──────────────────────────────────────────────────┘
  │
  ▼
┌─ eia-planner（规划）─────────────────────────────┐
│  将发现匹配为行动规则                                │
│  按影响/成本比排序                                   │
│  根据用户角色裁剪                                    │
└──────────────────────────────────────────────────┘
  │
  ▼
┌─ eia-report（报告）──────────────────────────────┐
│  用用户语言组织输出                                   │
│  对话模式或报告模式                                   │
│  追加 pipeline trace + 写入记忆                      │
└──────────────────────────────────────────────────┘
```

---

## 6 个 Skill

| Skill | 职责 |
|-------|------|
| `eia` | 编排器 —— 意图路由、用户画像推断、pipeline 选择 |
| `eia-collect` | 数据采集 —— git/GitHub API、事件归一化、模块 enrichment |
| `eia-build` | 知识构建 —— 时间线链接、实体图谱、聚合指标 |
| `eia-reason` | 统一推理 —— 风险评分、根因分析、趋势检测 |
| `eia-planner` | 行动规划 —— 发现→建议、优先级排序、角色适配 |
| `eia-report` | 报告生成 —— 多语言输出、格式化、记忆写入 |

---

## 意图路由

| 意图 | Pipeline | 是否调用 planner |
|------|----------|-----------------|
| debug（调试） | collect → build → reason(rca) → report | 跳过 |
| investigate（调查） | collect → build → reason(all) → plan → report | 执行 |
| monitor（监控） | collect → build → reason(risk) → [plan →] report | risk > 0.6 时执行 |
| predict（预测） | collect → build → reason(risk+trend) → plan → report | 执行 |
| report（报告） | collect → build → reason(trend) → plan → report | 执行 |
| optimize（优化） | collect → build → reason(all) → plan → report | 执行 |

**冲突优先级**：investigate > debug > optimize > predict > report > monitor

---

## 配置

| 变量 | 用途 |
|------|------|
| `GITHUB_TOKEN` | 启用 GitHub API 采集 PR/issue/review 数据 |

Git 数据开箱即用，无需配置。记忆存储在 `~/.eia/memory.md`（自动创建）。

---

## 使用示例

| 提问 | 执行的 pipeline |
|------|----------------|
| 「为什么支付模块最近总是超时？」 | collect → build → reason(rca) → report |
| 「Why have PR review times increased?」 | collect → build → reason(all) → plan → report |
| 「生成本周工程效率报告」 | collect → build → reason(trend) → plan → report |
| 「这个模块风险怎么样？」 | collect → build → reason(risk+trend) → plan → report |
| 「优化我们的开发流程」 | collect → build → reason(all) → plan → report |

---

## 许可证

MIT
