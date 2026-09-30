# 🧬 Agent Designer

> **An agent that designs agents — with the discipline wired into the pipeline.**
> **一个设计 agent 的 agent —— 把核查纪律内联进流水线：先判该不该建，再联网核实，最后必须验，验不过回炉。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-plugin-blueviolet)](https://code.claude.com/docs/en/plugin-marketplaces)

---

## 它和官方的东西差在哪

Anthropic 官方**已经有**一个 `agent-creator`（在 `anthropics/claude-plugins-official` → `plugins/plugin-dev`），而且**同一个插件下还有校验件**：`agents/plugin-validator.md` 和 `skills/agent-development/scripts/validate-agent.sh`。

所以准确的差异不是「官方没有校验」，而是：

| | 官方 `agent-creator` | 本插件 |
|---|---|---|
| **它自己能调动的工具** | `tools: ["Write", "Read"]` —— **调不动**同插件里的校验件 | 带 `Bash` / `Grep` / `WebFetch`，能**真跑**校验 |
| **校验接在哪** | 校验件存在，但**没接进 `agent-creator` 这条流程** | 校验**内联在同一 agent 的流水线里**，不过就回炉（≤3 轮） |
| **准入闸门** | 无 —— 你说要建它就建 | 先判「**该不该建**」；**最常见的正确输出是「不用建」** |
| **自身规范一致性** | 它的 `description` 违反了同仓库 `references/` 的两条硬规则（要求扁平散文、禁止引号复述用户原话 —— 它两条都犯） | 显式**选边并写明**，见下方「选边声明」 |

一句话：**官方缺的不是「校验这件事」，是「把校验接进 agent-creator 这一个 agent」。**

> 顺带一提：官方已在 v2.1.198 **移除** `/agents` 交互式创建向导，退回到「让 Claude 直接写文件」。
> 也就是说，这个位置官方自己也没收敛。

## 五阶段流水线

| # | 阶段 | 做什么 | 产出 |
|---|------|--------|------|
| 0 | **准入闸门** | 只有满足「需要独立评判」或「需要隔离上下文」之一才建；都不满足 → 劝退，改推荐 Skill / CLAUDE.md / Hook / 直接用 `tools` 字段 | 建或不建的判定 + 理由 |
| 1 | **需求澄清** | 八问：谁触发、输入、输出、落点、**不做什么**、失败模式、验收判据、写区边界 | 需求卡 |
| 2 | **调研** | 在 GitHub / 官方文档找同类实现，**逐条打开页面核实**，标「已核实 / 未核实」，并**把目录列全**（不许选择性引用） | 同类实现对照表 |
| 3 | **设计** | 按房子风格产出 frontmatter 四件套 + `description` 三段式 + 正文骨架 + 最小权限白名单 | agent 草稿 + 设计说明 |
| 4 | **校验** | 跑 rubric：路由测试、**tools 与边界一致性**、锚点/路径存在性、静默失败陷阱 | 校验报告，不过则回炉 |

> **第 0 阶段是这个项目最值钱的部分。**
> 大多数「agent 生成器」你说要它就生成，结果是 agent 泛滥。
> 这个反过来——**它最常见的正确输出是「不用建」**。

## 特性

| 特性 | 说明 |
|------|------|
| 🚧 **准入闸门** | 建之前先问「该不该建」；不满足就劝退并给替代方案（含「只是要限制工具 → 用 `tools` 字段就够」） |
| 🔍 **联网调研 + 核实纪律** | 每个引用的仓库都打开页面确认，标「已核实 / 未核实」；**目录要列全**；找不到就写「未找到」，不许编 |
| 🩺 **校验回环** | `claude plugin validate` + `Grep` 扫引用逐条核对 + **静默失败陷阱**（4 项不报错 + 1 项硬拒绝），不过回炉 |
| 🧱 **稳定层 / 易变层分离** | 人设与准则（稳定）和机器/版本/技术栈（易变）分开写，换机器只改一处 |
| 📐 **房子风格标准** | 可复用的 agent 写作规范，见 `CLAUDE.md` |
| ⚖️ **选边声明** | 凡官方自身有矛盾或本插件有取舍的地方，**显式写明是选边**，不装作没有 |
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

会话内也可以用斜杠命令（`/plugin install` 会**打开插件面板**让你确认，不是直接装）：

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
  ▸ ③ 调研：找到 N 个同类实现，逐个核实并列出全部子目录（对照表，标注已核实/未核实）
  ▸ ④ 设计：产出 .claude/agents/resume-auditor.md
  ▸ ⑤ 校验：跑 rubric → 发现 tools 里有 Write 但描述说「只读」→ 回炉收窄权限

  → 交付：定义文件 + 设计说明（六节）
```

## 选边声明（这个插件有意做出了这些取舍）

官方自身在两个地方是矛盾的，本插件不装作没有，而是**显式选边**：

| 分歧 | 本插件的选择 |
|------|------------|
| `description` 范式：官方 reference 要求**扁平散文**、禁止引号复述用户原话；官方 `agent-creator` 自己却用**引号 + `<example>` 块** | 选**「单行 + 「」引号触发语」** —— 认同按用户原话命中更准（中文尤甚），但不采用块标量形态（会吃掉常驻预算） |
| `agent-creator` 有校验件却不调用 | 把**校验内联进流水线**，写死「不许声称验证过而没真跑」 |
| 准入判据 | 「需要隔离上下文」**有官方依据**；「需要独立评判」是**本插件的取舍**（源自 harness 工程里生成者/评判者分离的实践），**官方 sub-agents 文档未明列** |

## 与官方的关系

本插件**不重复造**官方已有的东西，而是补它没做的：

- `/doctor prompt-audit`（v2.1.283+）已会查 `.claude/` 下 subagents/skills 的「引用了不存在的文件或命令」「文件互相矛盾」→ `agent-rubric` **直接告诉你先跑它**，自己只做官方不做的维度（路由语义 / 权限一致性 / 结构 / 选边）。
- 官方 `plugin-validator` agent 与 `validate-agent.sh` 已在做结构校验 → 同样先跑它。
- 官方 `skill-creator` 有完整 eval 回环（`run_eval.py` / `run_loop.py` / `aggregate_benchmark.py` / `improve_description.py`）→ agent 侧缺的就是这个回环，本插件把它补在 agent 流程里。

## 关键事实的核实状态

本文档中关于官方的每条事实，均在 **2026-09-30 打开原始页面 / 调 API** 核实：

| 事实 | 核实方式 |
|------|---------|
| `agent-creator` 的 `tools: ["Write", "Read"]` | 读 `anthropics/claude-plugins-official` 的 `plugins/plugin-dev/agents/agent-creator.md` |
| 官方 `agent-creator` 的 `description` 与同仓库 reference 矛盾 | 对读 `agents/agent-creator.md` 与 `skills/agent-development/references/triggering-examples.md` |
| `skill-creator` 的 eval 脚本存在 | 列 `plugins/skill-creator/skills/skill-creator/scripts/` 目录 |
| v2.1.198 移除 `/agents` 向导 | `code.claude.com/docs/en/commands` |
| `prompt-audit` 需 v2.1.283+ 及其检查项 | `code.claude.com/docs/en/memory` 与 `/commands` |
| 静默失败陷阱、frontmatter 字段表 | `code.claude.com/docs/en/sub-agents` 与 `/skills` |
| 未核实项 | 本文档**不含**未标注的未核实内容；`agent-research` 里那张参考表已注明「用前请自己再核一遍」 |

## 许可

[MIT](LICENSE) © 2026 Zarrc
