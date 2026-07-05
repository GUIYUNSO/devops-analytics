<div align="center">

# DevOps 智能分析引擎

### 一句话问出代码仓库的风险、根因和趋势 —— AI 自动采集数据、推理分析、给出行动建议

一个 Codex 插件，让软件团队用自然语言完成工程质量分析：风险评分、bug 根因追溯、趋势检测、周报生成，全部在一个对话里完成。

[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](./LICENSE)
[![Plugin: Codex](https://img.shields.io/badge/Plugin-Codex-purple?style=flat-square)](https://github.com/GUIYUNSO/devops-analytics)
[![Version: 4.0](https://img.shields.io/badge/Version-4.0-blue?style=flat-square)](https://github.com/GUIYUNSO/devops-analytics)

</div>

---

## 它是什么

EIA 是一个 Codex 插件，由 6 个独立 Skill 组成。你用自然语言提问，它自动完成以下流程：

1. **采集** — 从 git 历史和 GitHub API 拉取 commit、PR、issue、review 数据
2. **构建** — 将事件链接为生命周期时间线，构建实体关系图谱
3. **推理** — 风险评分、根因分析、趋势检测三种模式
4. **规划** — 将分析结果转化为按优先级排序的行动建议
5. **报告** — 用你的语言输出结论，附带具体数据证据

不需要写代码，不需要配 dashboard，直接问就行。

---

## 谁适合用

| 角色 | 使用场景 |
|------|---------|
| **开发者** | 「这个 PR 风险高吗？」「为什么 payment 模块最近老出 bug？」 |
| **Tech Lead** | 「哪个模块代码质量最差？」「哪些 PR 需要重点 review？」 |
| **DevOps** | 「CI/CD 流水线有没有瓶颈？」「部署频率正常吗？」 |
| **PM / 管理层** | 「这周工程效率怎么样？」「有哪些阻塞交付的风险？」 |
| **QA** | 「测试覆盖率够吗？」「哪些模块 bug 最集中？」 |

---

## 快速安装

```bash
git clone https://github.com/GUIYUNSO/devops-analytics.git
cp -r devops-analytics/plugins/devops-analytics ~/.codex/plugins/
rm -rf devops-analytics
```

然后在 `~/.agents/plugins/marketplace.json` 的 `plugins` 数组末尾加：

```json
{
  "name": "devops-analytics",
  "source": { "source": "local", "path": ".codex/plugins/devops-analytics" },
  "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
  "category": "Productivity"
}
```

重启 Codex 即可使用。

<details>
<summary>其他安装方式</summary>

**Codex 命令安装：**
```bash
codex plugin marketplace add GUIYUNSO/devops-analytics
codex plugin add devops-analytics@guiyunso
```

**macOS/Linux 一行脚本（含自动注册）：**
```bash
git clone https://github.com/GUIYUNSO/devops-analytics.git /tmp/eia && cp -r /tmp/eia/plugins/devops-analytics ~/.codex/plugins/ && rm -rf /tmp/eia && mkdir -p ~/.agents/plugins && python3 -c "import json,os;f=os.path.expanduser('~/.agents/plugins/marketplace.json');e={'name':'devops-analytics','source':{'source':'local','path':'.codex/plugins/devops-analytics'},'policy':{'installation':'AVAILABLE','authentication':'ON_INSTALL'},'category':'Productivity'};d=json.load(open(f)) if os.path.exists(f) else {'name':'personal','interface':{'displayName':'Personal'},'plugins':[]};d.setdefault('plugins',[]);[d['plugins'].append(e) for _ in [1] if not any(p['name']=='devops-analytics' for p in d['plugins'])];json.dump(d,open(f,'w'),indent=2);print('Done')" && echo "重启 Codex 即可"
```

</details>

---

## 6 个 Skill 详解

### eia — 编排器

**职责**：接收用户问题，判断意图，选择分析路径，编排后续 skill 执行。

**具体能力**：
- 自动识别用户角色（开发者 / Tech Lead / PM / QA），根据角色调整输出详略
- 自动检测语言，中文提问中文回答，英文提问英文回答
- 根据问题关键词分类为 6 种意图之一，每种意图走不同 pipeline
- 读取历史记忆（`~/.eia/memory.md`），跨会话积累分析上下文

**意图路由规则**：

| 意图 | 触发词 | 执行的 pipeline |
|------|--------|----------------|
| debug | 出bug了、error、crash、失败 | collect → build → 根因分析 → report |
| investigate | 为什么、why、分析、根因、trace | collect → build → 全部推理 → plan → report |
| monitor | 状态、status、health、看看 | collect → build → 风险评分 → report |
| predict | 预测、risk、风险、会不会 | collect → build → 风险+趋势 → plan → report |
| report | 报告、周报、月报、汇总 | collect → build → 趋势分析 → plan → report |
| optimize | 优化、改进、怎么办、improve | collect → build → 全部推理 → plan → report |

---

### eia-collect — 数据采集

**职责**：从 git 和 GitHub API 拉取原始数据，统一为标准事件格式。

**具体能力**：
- **git log**：自动采集 commit 历史（作者、时间、消息、文件变更、增删行数）
- **GitHub API**：拉取 PR（标题、作者、reviewers、状态、大小）、issue、review 评论
- **事件归一化**：不同来源的数据统一为 `{type, actor, repo, timestamp, payload}` 格式
- **模块识别**：按文件路径前缀自动归类模块（如 `src/payment/` → payment 模块）
- **Churn 计算**：统计每个文件在时间窗口内的变更频率
- **Ownership 识别**：按 commit 数量识别每个模块的主要贡献者

**输出示例**：
```json
{
  "events": [...],
  "modules": {
    "src/payment": { "commits": 12, "contributors": ["alice"], "churn": 0.85, "bug_rate": 3.2 }
  }
}
```

---

### eia-build — 知识构建

**职责**：将采集到的事件构建为两种结构化知识。

**时间线构建**：
- 通过 commit message 中的 issue key、PR 中的 commit SHA 将事件链接为生命周期链
- 计算每个链的 Lead Time（issue 创建到部署）、Cycle Time（首次提交到部署）、Review Time
- 输出瓶颈阶段（哪个环节耗时最长）

**知识图谱构建**：
- 实体：Developer、Module、PR、Commit、Issue、Bug、Deploy
- 关系：authored、opened、reviewed、owns、touched、resolved、introduced、caused_by
- 支持查询：给定实体找所有关联实体、两个实体之间的路径、模块的 bug 数量

---

### eia-reason — 统一推理

**职责**：根据意图选择分析模式，输出结构化发现。

#### 模式 1：风险评分

为每个 PR 和模块打分（0-1），评分因子：

| 因子 | 权重 | 评分规则 |
|------|------|---------|
| PR 大小（LOC） | 高 | >400 行=0.7，>800 行=0.9 |
| 代码 Churn | 高 | 直接取 churn score |
| Review 深度 | 中 | <10 分钟=0.8，<1 小时=0.5 |
| Reviewer 数量 | 中 | 仅 1 人=0.7 |
| 历史缺陷 | 高 | 模块 bug 率 >2/100 commits=0.8 |
| Bus Factor | 中 | 仅 1 人维护=0.6 |

**上下文修正**：bot 提交（dependabot）风险减半；纯文档 PR 置为 0.1。

输出风险等级：Low（0-0.3）/ Medium（0.3-0.6）/ High（0.6-0.8）/ Critical（0.8-1.0）

#### 模式 2：根因分析

对每个 bug 执行反向追溯：

```
Bug → 哪个 deploy 引入的？→ 哪个 PR 在那个 deploy 里？→ 哪个 commit 改了相关代码？→ 根因分类
```

根因分类：code_defect / config_change / dependency / infrastructure / process_gap

**链路断裂处理**：每一步都有 fallback，链路不完整时标记 `data_gap`，不猜测，降级输出已知部分。

#### 模式 3：趋势分析

对比当前周期 vs 上一周期：

| 指标 | 计算方式 | 显著性阈值 |
|------|---------|-----------|
| Bug 数量 | 本期 vs 上期 | >50% 变化=critical |
| Lead Time | 中位数对比 | >50% 变化=critical |
| PR 大小 | 中位数对比 | >50% 变化=critical |
| 部署频率 | 每周部署次数 | — |
| Review 时间 | PR 从打开到批准的中位数 | — |

**Review 文化分析**：对比 open PR 和已 merged PR 的 review 覆盖率，区分"临时积压"和"长期无 review 文化"。

---

### eia-planner — 行动规划

**职责**：将推理结果转化为按优先级排序的行动建议。

**优先级定义**：
- **[P0]** — 阻塞发布或安全风险，本周内处理
- **[P1]** — 降低质量或效率，本迭代处理
- **[P2]** — 改进机会，放 backlog

**行动规则库**（部分）：

| 发现 | 建议 |
|------|------|
| PR > 500 LOC | 拆分为更小的 PR |
| 单人 review 高风险 PR | 增加 reviewer |
| review 时间 < 5 分钟 | 重新 review |
| 模块 churn > 10 次/周 | 计划重构 |
| Bus factor = 1 | 知识转移 |
| code_defect 根因 | 添加回归测试 |
| Bug 数量 critical 上升 | 检查 PR 大小，增加测试覆盖 |

**角色适配**：developer 看到文件级建议，PM 看到时间线建议，CTO 看到团队级建议。

---

### eia-report — 报告生成

**职责**：将所有上游结果组织为用户可读的输出。

**两种输出格式**：

**对话模式**（默认）：
```
[一句话结论]

[关键数据 — 3-5 个，带数字]

[P0/P1/P2 行动项]

[一个跟进问题]
```

**报告模式**（report 意图时）：
```markdown
# 工程效率报告 — {time_range}

## 概况
一段话总结

## 关键指标
| 指标 | 本期 | 上期 | 变化 | 状态 |

## 发现
1. **标题** — 说明 [evidence: PR #xxx]

## 行动建议
[P0] 建议内容
[P1] 建议内容
[P2] 建议内容
```

**输出规则**：
- 系统性表述，不针对个人（「项目存在单点活跃风险」而非「某人有 50% PR」）
- 每个数字带上下文（「86% unreviewed」而非「很多 PR 没 review」）
- 每个发现配行动建议
- 末尾追加 pipeline trace 和记忆写入

---

## 使用示例

### 调试问题
```
你：为什么支付模块最近总是超时？
AI：自动执行 collect → build → 根因分析 → report
输出：定位到 PR #89 的 commit abc123 改了 validator.go:142，缺少 API timeout 的 nil check
```

### 风险评估
```
你：这个模块风险怎么样？
AI：自动执行 collect → build → 风险评分 + 趋势分析 → plan → report
输出：payment 模块 risk_score=0.78（High），主要因子：churn=0.85、bug_rate=3.2、bus_factor=1
```

### 周报生成
```
你：给这周的工程效率做个报告
AI：自动执行 collect → build → 趋势分析 → plan → report
输出：Markdown 格式的完整工程效率报告，含指标对比、发现、行动建议
```

### 根因追溯
```
你：为什么最近线上 bug 越来越多？
AI：自动执行 collect → build → 全部推理 → plan → report
输出：bug +50% QoQ，主要集中在 payment 模块，根因是 3 个大 PR 缺少 review
```

---

## 配置

| 变量 | 是否必须 | 说明 |
|------|---------|------|
| `GITHUB_TOKEN` | 可选 | 启用 GitHub API，采集 PR/issue/review 数据。没有时仅用 git 数据 |

记忆存储在 `~/.eia/memory.md`，append-only，可随时查看或删除。

---

## 工作原理

```
用户提问
    │
    ▼
┌─ eia ────────────────┐
│  推断角色、意图、语言   │
└──────────┬───────────┘
           ▼
    collect → build → reason → plan → report
    │         │        │         │       │
    采集     构建     推理      规划    输出
    git     时间线   风险/RCA  行动项  报告
    API     图谱     趋势      优先级  对话
```

6 个 Skill 独立运行，通过 `previous_output` 传递数据，每个有严格 IO contract。

---

## FAQ

**需要联网吗？**

Git 数据完全本地采集。GitHub API 需要 `GITHUB_TOKEN`，仅用于拉取 PR/issue/review 数据，不做其他网络请求。

**没有 GITHUB_TOKEN 能用吗？**

能。Git 数据开箱即用。没有 token 时跳过 GitHub API，分析范围限于本地 git history。核心功能（风险评分、根因分析、趋势检测）仍然可用。

**记忆存在哪？安全吗？**

`~/.eia/memory.md`，纯文本 append-only 文件。只记录分析摘要（如「payment 模块 30 天 3 个 bug」），不记录代码内容。你可以随时查看、编辑或删除。

**支持哪些意图？**

6 种：debug（调试）、investigate（调查）、monitor（监控）、predict（预测）、report（报告）、optimize（优化）。AI 根据你的问题自动选择，也可以在提问中明确指定。

**能分析 monorepo 吗？**

目前按单仓库设计。多仓库场景可以通过指定不同 repo 多次运行。

**输出可以导出吗？**

报告直接以 Markdown 格式输出在对话中，可以复制粘贴到任何地方。也可以让 AI 写入文件：「把报告写到 report.md」。

---

## 许可证

[MIT](./LICENSE) — 拿去随便用，记得改成自己仓库名。

---

<div align="center">

**用过觉得有用？给个 ⭐ 是对作者最大的鼓励。**

[⬆ 回到顶部](#devops-智能分析引擎) · [📥 安装](#快速安装) · [💬 提 Issue](https://github.com/GUIYUNSO/devops-analytics/issues)

</div>
