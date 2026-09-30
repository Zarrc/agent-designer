# 🧬 Agent Designer

> **An agent that designs agents — with the verification loop the official one lacks.**
> **一个设计 agent 的 agent —— 补上官方 `agent-creator` 缺失的那一环：验证。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-plugin-blueviolet)](https://code.claude.com/docs/en/plugin-marketplaces)

---

## 为什么不是「又一个 agent 生成器」

Anthropic 官方**已经有**一个 `agent-creator`（在 `anthropics/claude-plugins-official` → `plugins/plugin-dev`）。

两者的差别只有一条，但它是决定性的：

| | 官方 `agent-creator` | 本插件 |
|---|---|---|
| **能否执行校验** | ❌ `tools: ["Write", "Read"]` —— 写完无法执行任何东西 | ✅ 带 `Bash`，能真跑校验 |
| **校验回环** | 无 | 跑 `claude plugin validate` + 逐条存在性校验，**不过就回炉，最多 3 轮** |
| **准入闸门** | 无 —— 你说要建它就建 | 先判「**该不该建**」，最常见的正确输出是「不用建」 |
| **自身规范一致性** | 它的 `description` 违反了同仓库 `references/` 的两条硬规则（要求扁平散文、禁止引号复述用户原话，它自己两条都犯） | 显式选边并写明理由 |

一句话：**官方那个只会写、不会验；这个写完必须验。**

> 顺带一提：官方已在 v2.1.198 **移除** `/agents` 交互式创建向导，退回到「让 Claude 直接写文件」。
> 也就是说，这个位置官方自己也没收敛。

## 五阶段流水线

| # | 阶段 | 做什么 | 产出 |
|---|------|--------|------|
| 0 | **准入闸门** | subagent 只有两个理由值得建：**需要独立评判** / **需要隔离上下文**。两者都不满足 → 劝退，改推荐 Skill / CLAUDE.md / Hook | 建或不建的判定 + 理由 |
| 1 | **需求澄清** | 八问：谁触发、输入、输出、落点、**不做什么**、失败模式、验收判据、写区边界 | 需求卡 |
| 2 | **调研** | 在 GitHub / 官方文档找同类实现，**逐条打开页面核实**，标「已核实 / 未核实」 | 同类实现对照表 |
| 3 | **设计** | 按房子风格产出 frontmatter 四件套 + `description` 三段式 + 正文骨架 + 最小权限白名单 | agent 草稿 + 设计说明 |
| 4 | **校验** | 跑 rubric：路由测试、**tools 与边界一致性**、锚点/路径存在性、静默失败四项 | 校验报告，不过则回炉 |

> **第 0 阶段是这个项目最值钱的部分。**
> 市面上大多数「agent 生成器」你说要它就生成，结果是 agent 泛滥。
> 这个反过来——**它最常见的正确输出是「不用建」**。

## 特性

| 特性 | 说明 |
|------|------|
| 🚧 **准入闸门** | 建之前先问「该不该建」；不满足条件就劝退并给替代方案 |
| 🔍 **联网调研 + 核实纪律** | 每个引用的仓库都打开页面确认，标「已核实 / 未核实」；找不到就写「未找到」，**不许编** |
| 🩺 **校验回环** | `claude plugin validate` + 存在性校验 + **静默失败四项**，不过回炉（最多 3 轮） |
| 🧱 **稳定层 / 易变层分离** | 人设与准则（稳定）和机器/版本/技术栈（易变）分开写，换机器只改一处 |
| 📐 **房子风格标准** | 可复用的 agent 写作规范，见 `CLAUDE.md` |
| 🔌 **插件 + 手动两种装法** | 一键 `/plugin marketplace add`，或直接 copy 文件 |
| 🪶 **低常驻开销** | 全书约 **520 tokens** 常驻 —— 只有 `description` 常驻，正文按需加载 |

## 包含什么

```
agent-designer/                          ← 仓库根 = 市场（marketplace）
├── .claude-plugin/
│   └── marketplace.json                 ← 市场清单
├── plugins/
│   └── agent-designer/                  ← 插件本体
│       ├── .claude-plugin/plugin.json
│       ├── agents/
│       │   └── agent-designer.md        ← 主角
│       └── skills/
│           ├── agent-anatomy/           ← 取舍与格式
│           │   ├── SKILL.md
│           │   └── references/
│           │       ├── frontmatter-reference.md   ← 全字段表 + 静默失败陷阱
│           │       └── examples.md                ← 好/坏范例对照
│           ├── agent-research/          ← 联网调研的方法与核实纪律
│           │   └── SKILL.md
│           └── agent-rubric/            ← 校验清单
│               └── SKILL.md
├── CLAUDE.md                            ← 房子风格标准（可复用资产）
├── LICENSE · README.md · .gitignore
```

**三个 skill 分工**：`agent-anatomy` 管「做哪种载体、长什么样」· `agent-research` 管「别人怎么做的」· `agent-rubric` 管「我做的对不对」。

## 安装

### A. 作为插件（推荐）

```bash
claude plugin marketplace add Zarrc/agent-designer
claude plugin install agent-designer@agent-designer
```

在会话内也可以用斜杠命令：

```
/plugin marketplace add Zarrc/agent-designer
/plugin install agent-designer@agent-designer
```

### B. 手动复制（不用插件机制）

```bash
# 项目级（写进你的仓库）
cp    plugins/agent-designer/agents/*   <你的项目>/.claude/agents/
cp -r plugins/agent-designer/skills/*   <你的项目>/.claude/skills/

# 或者用户级（所有项目都能用）
cp    plugins/agent-designer/agents/*   ~/.claude/agents/
cp -r plugins/agent-designer/skills/*   ~/.claude/skills/
```

> ⚠️ 注意路径差异：**插件**用 `<插件根>/agents/`；**手动安装**用 `.claude/agents/`。
> 放错位置**不会报错**，只是不生效。

### 验证安装

```bash
claude plugin list
claude plugin details agent-designer
```

实测输出（Claude Code v2.1.201，2026-09-30）：

```
Component inventory
  Skills (3)  agent-anatomy, agent-research, agent-rubric
  Agents (1)  agent-designer
```

## 使用例

```text
你：帮我设计一个 agent，专门检查我的简历里有没有夸大或编造的数字
Claude：（派 agent-designer）

  ▸ ① 准入闸门：这需要「独立评判」—— 写简历的和查简历的不能是同一个 → 通过
  ▸ ② 需求澄清：输入=简历文件 / 输出=问题清单 / 不做什么=不改简历 / 写区=只读
  ▸ ③ 调研：找到 N 个同类实现，逐个核实（对照表，标注已核实/未核实）
  ▸ ④ 设计：产出 .claude/agents/resume-auditor.md
  ▸ ⑤ 校验：跑 rubric → 发现 tools 里有 Write 但描述说「只读」→ 回炉收窄权限

  → 交付：定义文件 + 设计说明（六节）
```

## 三个 Skill 各自负责什么

| Skill | 回答的问题 |
|-------|-----------|
| `agent-anatomy` | 「这需求该做成 subagent / Skill / CLAUDE.md / Hook？」「定义文件该长什么样？」 |
| `agent-research` | 「别人怎么做的？」「有没有现成的？」「怎么核实一条引用是真的？」 |
| `agent-rubric` | 「我写的这个对不对？」「哪里会静默失效？」 |

## 与官方的关系

本插件**不重复造**官方已有的东西，而是补它没做的：

- 官方的 `/doctor prompt-audit`（v2.1.283+）已经会查 `.claude/` 下 subagents/skills 的「**引用了不存在的文件或命令**」「**文件互相矛盾**」→ `agent-rubric` **直接告诉你先跑它**，自己只做官方不做的维度（结构 / 权限一致性 / 路由 / 语义门禁）。
- 官方 `skill-creator` 有完整的 eval 回环；官方 `agent-creator` 没有 → 本插件把回环补在 agent 一侧。
- 官方 reference 与官方 `agent-creator` 的 `description` 范式互相矛盾 → 本插件**显式选边并写明**，不装作没有这回事。

## 许可

[MIT](LICENSE) © 2026 Zarrc

本仓库的规范部分参考了 Claude Code 官方文档（`code.claude.com/docs`）与 `anthropics/claude-plugins-official` 的公开内容；所有引用均在 2026-09-30 核实，核实状态在文档内逐条标注。
