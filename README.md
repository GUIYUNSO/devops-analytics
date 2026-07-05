<div align="center">

# 工程智能代理（EIA）

### 输入一个问题，AI 自动完成：采集 → 构建 → 推理 → 规划 → 报告

一个 Codex 插件，让你用自然语言分析代码仓库的风险、根因、趋势，并给出可执行建议。

[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](./LICENSE)
[![Plugin: Codex](https://img.shields.io/badge/Plugin-Codex-purple?style=flat-square)](https://github.com/GUIYUNSO/devops-analytics)
[![Version: 4.0](https://img.shields.io/badge/Version-4.0-blue?style=flat-square)](https://github.com/GUIYUNSO/devops-analytics)

</div>

---

## ✨ 它能做什么

| 能力 | 说明 |
|------|------|
| **风险评分** | 基于 PR 大小、代码 churn、review 深度、历史缺陷多维打分 |
| **根因分析** | bug → deploy → PR → commit 反向追溯，定位引入变更 |
| **趋势检测** | 周期对比，识别指标异常波动并生成假设 |
| **行动规划** | 按 [P0/P1/P2] 优先级输出可执行建议 |
| **跨语言** | 中文提问中文回答，英文提问英文回答 |
| **记忆** | 跨会话记忆，自动积累分析上下文 |

---

## ⚡ 30 秒装上

```bash
git clone https://github.com/GUIYUNSO/devops-analytics.git
cp -r devops-analytics/plugins/devops-analytics ~/.codex/plugins/
rm -rf devops-analytics
```

然后在 `~/.agents/plugins/marketplace.json` 的 `plugins` 数组末尾加一段：

```json
{
  "name": "devops-analytics",
  "source": { "source": "local", "path": ".codex/plugins/devops-analytics" },
  "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
  "category": "Productivity"
}
```

重启 Codex 即可。

<details>
<summary>其他安装方式</summary>

**Codex 命令安装：**

```bash
codex plugin marketplace add GUIYUNSO/devops-analytics
codex plugin add devops-analytics@guiyunso
```

**macOS / Linux 一行脚本（含自动注册 marketplace）：**

```bash
git clone https://github.com/GUIYUNSO/devops-analytics.git /tmp/eia && cp -r /tmp/eia/plugins/devops-analytics ~/.codex/plugins/ && rm -rf /tmp/eia && mkdir -p ~/.agents/plugins && python3 -c "import json,os;f=os.path.expanduser('~/.agents/plugins/marketplace.json');e={'name':'devops-analytics','source':{'source':'local','path':'.codex/plugins/devops-analytics'},'policy':{'installation':'AVAILABLE','authentication':'ON_INSTALL'},'category':'Productivity'};d=json.load(open(f)) if os.path.exists(f) else {'name':'personal','interface':{'displayName':'Personal'},'plugins':[]};d.setdefault('plugins',[]);[d['plugins'].append(e) for _ in [1] if not any(p['name']=='devops-analytics' for p in d['plugins'])];json.dump(d,open(f,'w'),indent=2);print('Done')" && echo "重启 Codex 即可"
```

</details>

---

## 🎬 怎么用

安装后在 Codex 对话里直接提问：

- *「分析这个仓库的主要风险」*
- *「为什么最近 bug 越来越多？」*
- *「给这周的工程效率做个报告」*
- *「这个模块风险怎么样？」*
- *「优化我们的开发流程」*

---

## 🏗️ 架构

```
用户提问
    │
    ▼
┌─ eia ────────────────┐
│  意图分类 + 角色推断    │
└──────────┬───────────┘
           ▼
    collect → build → reason → plan → report
```

6 个独立 Skill，通过 `previous_output` 传递数据，每个有严格 IO contract。

| Skill | 职责 |
|-------|------|
| `eia` | 编排器 — 意图路由、用户画像推断、pipeline 选择 |
| `eia-collect` | 数据采集 — git/GitHub API、事件归一化 |
| `eia-build` | 知识构建 — 时间线链接、实体图谱 |
| `eia-reason` | 统一推理 — 风险评分、根因分析、趋势检测 |
| `eia-planner` | 行动规划 — 发现→建议、优先级排序 |
| `eia-report` | 报告生成 — 多语言输出、格式化 |

---

## ❓ FAQ

<details>
<summary><b>需要联网吗？</b></summary>

Git 数据完全本地。GitHub API 需要 `GITHUB_TOKEN`（可选），仅用于拉取 PR/issue/review 数据。

</details>

<details>
<summary><b>没有 GITHUB_TOKEN 能用吗？</b></summary>

能。Git 数据开箱即用。没有 token 时跳过 GitHub API，分析范围限于本地 git history。

</details>

<details>
<summary><b>记忆存在哪？</b></summary>

`~/.eia/memory.md`，append-only markdown 文件。你可以随时查看、编辑或删除。

</details>

<details>
<summary><b>支持哪些意图？</b></summary>

6 种：debug（调试）、investigate（调查）、monitor（监控）、predict（预测）、report（报告）、optimize（优化）。AI 根据你的问题自动选择。

</details>

<details>
<summary><b>能分析 monorepo 吗？</b></summary>

目前按单仓库设计。多仓库场景可以通过指定不同 repo 多次运行。

</details>

---

## 📜 协议

[MIT](./LICENSE) — 拿去随便用，记得改成自己仓库名。

---

<div align="center">

**用过觉得有用？给个 ⭐ 是对作者最大的鼓励。**

[⬆ 回到顶部](#工程智能代理eia) · [📥 安装](#-30-秒装上) · [💬 提 Issue](https://github.com/GUIYUNSO/devops-analytics/issues)

</div>
