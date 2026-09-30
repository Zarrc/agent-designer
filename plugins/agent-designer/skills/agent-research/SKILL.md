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
5. **仓库改名 / 归档是常态，别凭记忆。** 已核实的先例：`RooVetGit/Roo-Code` → `RooCodeInc/Roo-Code`（**已归档**）、`All-Hands-AI/OpenHands` → `OpenHands/OpenHands`、`microsoft/autogen` → 已进 maintenance mode、官方导向 Microsoft Agent Framework。

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

## 已知的参考起点（仍需自行核实，别直接抄）

| 对象 | 为什么值得看 |
|------|------------|
| `anthropics/claude-plugins-official` → `plugin-dev` | 官方 `agent-creator`（`tools` 只有 Write/Read，**无校验回环**）+ `agent-development` skill 的三份 reference |
| 同仓库 `skill-creator` | **有完整 eval 回环**的范例：草稿 → 造测试 prompt → 跑 eval → 量化 → 重写 |
| `wshobson/agents` | 「单一源 → 多 harness 产物」的生成管线，最成熟的现实参考 |
| `PDgit12/harnage` | 唯一把 `verify → repair` 写清楚的公开管线（但输出是 TS 项目，且星数极低） |

## 输出

一段「调研摘要」，含：

1. 对照表
2. 一句结论：**有没有现成可用的？如果没有，空隙具体在哪？**
3. 所有 URL，**标已核实 / 未核实**
