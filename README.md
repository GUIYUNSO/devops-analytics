<div align="center">

# EIA — Engineering Intelligence Agent

### AI 驱动的 DevOps 分析插件，为 Codex 和 Claude Code 设计

**不是又一个看板。** 这个项目专注于跨工程事件的推理分析，而不是可视化指标。

[![许可证: MIT](https://img.shields.io/badge/许可证-MIT-green?style=flat-square)](./LICENSE)
[![插件: Codex](https://img.shields.io/badge/插件-Codex%20%7C%20Claude%20Code-purple?style=flat-square)](https://github.com/GUIYUNSO/devops-analytics)
[![版本: 4.0](https://img.shields.io/badge/版本-4.0-blue?style=flat-square)](https://github.com/GUIYUNSO/devops-analytics)

</div>

---

## 解决什么问题

软件团队每天产生大量工程数据：提交、合并请求、议题、评审、部署、Bug。但这些数据散落在不同平台，没有人把它们串起来回答真正的问题：

- 为什么 Bug 越来越多？
- 哪个模块风险最高？
- 哪些合并请求最值得重点评审？
- 什么拖慢了交付速度？

EIA 的答案是：**用自然语言提问，AI 自动采集数据、构建知识图谱、推理分析、给出行动建议。**

---

## 工作流程

```
用户提问
    │
    ▼
  采集（Collect）
    │  git log / GitHub API
    │  归一化为标准事件
    ▼
  构建（Build）
    │  时间线 + 知识图谱
    │  模块归属、变更频率、ownership
    ▼
  推理（Reason）
    │  风险评分 / 根因分析 / 趋势检测
    ▼
  规划（Plan）
    │  按 P0/P1/P2 排序行动建议
    ▼
  报告（Report）
    │  用你的语言输出结论
    ▼
  用户获得答案
```

---

## 统一事件模型

AI 不直接分析 GitHub 原始数据。而是先将所有事件归一化为标准格式：

```
提交（Commit）
    ↓
合并请求（Pull Request）
    ↓
议题（Issue）
    ↓
部署（Deployment）
    ↓
Bug / 告警（Incident）
```

每个事件统一为 `{type, actor, repo, timestamp, payload}`，跨平台、跨仓库通用。

---

## 设计理念

**为什么是 Collect → Build → Reason → Plan，而不是 Agent → Agent → Agent？**

因为工程分析需要**确定性**，不是自由发挥。

- **采集层**：确定性 — git 命令和 API 调用，结果可复现
- **构建层**：确定性 — 事件链接规则写死，不会漂移
- **推理层**：半确定 — 评分因子和阈值固定，LLM 做假设生成
- **规划层**：确定性 — 行动规则库匹配，优先级排序有公式
- **报告层**：灵活 — LLM 根据用户语言和角色组织输出

每个 Skill 有严格输入输出契约，通过 `previous_output` 传递数据，无隐式依赖。

---

## 谁适合用

| 角色 | 使用场景 |
|------|---------|
| **开发者** | 「这个合并请求风险高吗？」「为什么支付模块最近老出 Bug？」 |
| **技术负责人** | 「哪个模块代码质量最差？」「哪些 PR 需要重点评审？」 |
| **运维工程师** | 「持续集成流水线有没有瓶颈？」「部署频率正常吗？」 |
| **产品经理** | 「这周工程效率怎么样？」「有哪些阻塞交付的风险？」 |
| **测试工程师** | 「测试覆盖率够吗？」「哪些模块 Bug 最集中？」 |

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

## 使用示例

### 调试问题
```
你：为什么支付模块最近总是超时？
AI：自动执行 采集 → 构建 → 根因分析 → 报告
输出：定位到合并请求 #89 的提交 abc123 改了 validator.go:142，缺少接口超时的空值检查
```

### 风险评估
```
你：这个模块风险怎么样？
AI：自动执行 采集 → 构建 → 风险评分 + 趋势分析 → 规划 → 报告
输出：支付模块风险分=0.78（高），主要因子：变更频率=0.85、Bug 率=3.2、单点依赖=1
```

### 周报生成
```
你：给这周的工程效率做个报告
AI：自动执行 采集 → 构建 → 趋势分析 → 规划 → 报告
输出：Markdown 格式的完整工程效率报告，含指标对比、发现、行动建议
```

### 根因追溯
```
你：为什么最近线上 Bug 越来越多？
AI：自动执行 采集 → 构建 → 全部推理 → 规划 → 报告
输出：Bug +50% 环比，主要集中在支付模块，根因是 3 个大合并请求缺少评审
```

---

## 6 个技能详解

### eia — 编排器

**职责**：接收用户问题，判断意图，选择分析路径，编排后续技能执行。

- 自动识别用户角色（开发者 / 技术负责人 / 产品经理 / 测试），根据角色调整输出详略
- 自动检测语言，中文提问中文回答，英文提问英文回答
- 根据问题关键词分类为 6 种意图之一，每种意图走不同流程
- 读取历史记忆（Memory Adapter），跨会话积累分析上下文

**意图路由规则**：

| 意图 | 触发词 | 执行的流程 |
|------|--------|-----------|
| 调试 | 出bug了、error、crash、失败 | 采集 → 构建 → 根因分析 → 报告 |
| 调查 | 为什么、why、分析、根因、trace | 采集 → 构建 → 全部推理 → 规划 → 报告 |
| 监控 | 状态、status、health、看看 | 采集 → 构建 → 风险评分 → 报告 |
| 预测 | 预测、risk、风险、会不会 | 采集 → 构建 → 风险+趋势 → 规划 → 报告 |
| 报告 | 报告、周报、月报、汇总 | 采集 → 构建 → 趋势分析 → 规划 → 报告 |
| 优化 | 优化、改进、怎么办、improve | 采集 → 构建 → 全部推理 → 规划 → 报告 |

---

### eia-collect — 数据采集

**职责**：从 git 和 GitHub 接口拉取原始数据，归一化为标准事件。

- **git log**：采集提交历史（作者、时间、消息、文件变更、增删行数）
- **GitHub 接口**：拉取合并请求、议题、评审评论
- **事件归一化**：统一为 `{type, actor, repo, timestamp, payload}` 格式
- **模块识别**：按文件路径前缀自动归类（如 `src/payment/` → 支付模块）
- **变更频率**：统计每个文件在时间窗口内的变更次数
- **归属识别**：按提交数量识别每个模块的主要贡献者

---

### eia-build — 知识构建

**职责**：将归一化事件构建为两种结构化知识。

**时间线**：将事件链接为生命周期链（议题 → 提交 → 合并请求 → 部署 → Bug），计算交付周期、开发周期、评审时间，定位瓶颈。

**知识图谱**：构建实体（开发者、模块、合并请求、提交、议题、Bug、部署）和关系（编写、评审、归属、修改、解决、引入、导致），支持图查询。

---

### eia-reason — 统一推理

**职责**：根据意图选择分析模式，输出结构化发现。

**风险评分**：为每个合并请求和模块打分（0-1），因子包括 PR 大小、变更频率、评审深度、评审人数量、历史 Bug 率、单点依赖。输出等级：低 / 中 / 高 / 严重。

**根因分析**：对每个 Bug 反向追溯（Bug → 部署 → 合并请求 → 提交），分类根因：代码缺陷 / 配置变更 / 依赖问题 / 基础设施 / 流程缺失。链路断裂时降级输出，不猜测。

**趋势分析**：对比当前周期与上周期的 Bug 数量、交付周期、PR 大小、部署频率、评审时间。识别显著变化（>50% = 严重）并生成假设。

---

### eia-planner — 行动规划

**职责**：将推理结果转化为按优先级排序的行动建议。

- **[P0]** — 阻塞发布或安全风险，本周内处理
- **[P1]** — 降低质量或效率，本迭代处理
- **[P2]** — 改进机会，放入待办

行动规则库覆盖：PR 拆分、评审人分配、测试补充、模块重构、知识转移、流程改进等场景。根据用户角色裁剪建议粒度。

---

### eia-report — 报告生成

**职责**：将所有上游结果组织为用户可读的输出。

**对话模式**（默认）：一句话结论 → 关键数据 → 行动项 → 跟进问题。

**报告模式**：完整 Markdown 报告，含指标对比表、发现列表、行动建议。

输出规则：系统性表述（不针对个人）、每个数字带上下文、每个发现配行动建议。

---

## 记忆系统

**Memory Adapter**：默认后端为 Markdown 文件（`~/.eia/memory.md`），append-only，可随时查看或删除。仅记录分析摘要，不记录代码内容。

未来可扩展至 SQLite、PostgreSQL、向量数据库。

---

## 配置

| 变量 | 是否必须 | 说明 |
|------|---------|------|
| `GITHUB_TOKEN` | 可选 | 启用 GitHub 接口，采集合并请求/议题/评审数据。没有时仅用 git 数据 |

---

## 常见问题

**需要联网吗？** Git 数据完全本地采集。GitHub 接口需要 `GITHUB_TOKEN`，仅用于拉取数据，不做其他网络请求。

**没有 GITHUB_TOKEN 能用吗？** 能。Git 数据开箱即用，核心功能（风险评分、根因分析、趋势检测）仍然可用。

**支持哪些意图？** 6 种：调试、调查、监控、预测、报告、优化。AI 根据问题自动选择。

**能分析多仓库吗？** 目前按单仓库设计，多仓库可通过指定不同仓库多次运行。

**输出可以导出吗？** 报告以 Markdown 格式输出在对话中，可复制粘贴，也可让 AI 写入文件。

---

## 许可证

[MIT](./LICENSE) — 拿去随便用，记得改成自己仓库名。

---

<div align="center">

**用过觉得有用？给个 ⭐ 是对作者最大的鼓励。**

[⬆ 回到顶部](#eia--engineering-intelligence-agent) · [📥 安装](#快速安装) · [💬 提问题](https://github.com/GUIYUNSO/devops-analytics/issues)

</div>
