---
name: agent-research
description: 为「设计一个新 agent」做联网调研 —— 在 GitHub 与官方文档里找同类实现、核实仓库真实存在与活跃度、学别人的 description 与工具边界怎么写，产出「重叠 / 差异 / 可借用」对照表。当你要为某个 agent 设计找参考，或想知道「别人怎么做的 / 有没有现成的」时使用。**不生成 agent 定义**（那用 agent-designer）；**不做校验**（那用 agent-rubric）。
---

# Agent Research

## 铁律：核实纪律

**这一条是全部价值所在。** AI 生成的 agent 定义最常翻的车，就是**编造不存在的仓库 / API / 字段名**。

1. **每个引用的仓库，必须打开页面或走 API 确认存在。** 优先 `gh api repos/<owner>/<repo>` —— 一行拿到星数、归档状态、最后 push 时间。
2. **每个状态标注（活跃 / 归档 / 已改名）都要有出处。**
3. **标「已核实 / 未核实」**：打开过原文的标已核实；只有搜索摘要的标未核实，**并明说不要引用其细节**。
4. **找不到就写「未找到」。** 空缺是结论，编造不是。
5. **别漏。** 列一个项目的内容时，**把它的子目录列全**（`gh api .../git/trees/HEAD?recursive=1`），不要只挑自己想要的。**选择性调研和编造一样违反纪律。**
6. **别用「唯一 / 首个 / 没有之一」这类全称否定** —— 除非你能证明。换成「在我检索到的范围内」。
7. **仓库改名 / 归档是常态，别凭记忆。** 已核实的先例：`RooVetGit/Roo-Code` → `RooCodeInc/Roo-Code`（**已归档**）、`All-Hands-AI/OpenHands` → `OpenHands/OpenHands`、`microsoft/autogen` → 已进 maintenance mode、官方导向 Microsoft Agent Framework。

## 搜什么

| 目标 | 查询模板 |
|------|---------|
| 官方规范 | `<harness> docs subagent frontmatter` / `<harness> skills SKILL.md` |
| 同类 agent | `<task> agent claude` / `awesome-claude-code <category>` |
| 有没有人做过 | `"agent creator" claude` / `meta-agent generator` |
| 看别人实现细节 | `gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1` 直接列目录 |

**优先用 `gh`，不要只靠搜索**：`gh api` / `gh search repos` / `gh repo view` 给的是一手元数据，搜索引擎给的是二手摘要。

## 对照表格式（固定）

| owner/repo | ★ | 活跃 | 说明 | 重叠 | 差异 / 可借用 |
|---|---|---|---|---|---|

「差异 / 可借用」这一列是**重点不是装饰**：写清「它有什么我们没有」「我们有什么它没有」。

## 已知的参考起点

> 下表的元数据取自 **2026-09-30 的 `gh api` 核实**；「需要看什么」是**判断，不是事实**。用之前请自己再核一遍。

| owner/repo | ★ | 活跃 | 与「设计 agent」的关系 | 需要看什么 |
|---|---|---|---|---|
| `anthropics/claude-plugins-official` → `plugins/plugin-dev` | 37.2k（父仓库） | ✅ 未归档 | 官方 `agent-creator`（`tools` 只有 Write/Read）+ `agent-development` skill + **`agents/plugin-validator.md`** + **`scripts/validate-agent.sh`** | 官方**已有**校验件，只是没接进 `agent-creator` —— 先看它做了什么，再决定自己做什么 |
| 同上 → `plugins/skill-creator` | —— | ✅ 未归档 | **有完整 eval 回环**：`run_eval.py` / `run_loop.py` / `aggregate_benchmark.py` / `improve_description.py` | 这是「有回环」的范例；agent 侧缺的就是它 |
| `wshobson/agents` | 40.1k | ✅ 未归档 | 「单一源 → 多 harness 产物」的生成管线 | 稳定层 / 易变层怎么拆、怎么生成多变体 |
| `PDgit12/harnage` | 1 | ✅ 未归档 | 在我检索到的范围内，**把 `verify → repair` 写得最清楚的公开管线**（输出是 TS 项目，非 `.claude/agents/`） | 生成管线里的校验回环怎么设计 |

**核实用法**：`gh api repos/<owner>/<repo>` · `gh api repos/<owner>/<repo>/git/trees/HEAD?recursive=1`

## 输出

一段「调研摘要」，含：

1. 对照表（用上面的固定格式，含 ★ 与活跃列）
2. 一句结论：**有没有现成可用的？如果没有，空隙具体在哪？**
3. 所有 URL，**标已核实 / 未核实**
