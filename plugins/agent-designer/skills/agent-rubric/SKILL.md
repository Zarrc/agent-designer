---
name: agent-rubric
description: Agent / Skill 定义的校验清单 —— description 路由测试、职责单一性、tools 与边界是否一致、路径与锚点是否真实存在、稳定层与易变层是否混淆、外部引用是否编造，外加会静默失败的字段级陷阱。当你写完或改完一个 agent / skill 定义需要自检，或要体检现有 `.claude/` 目录时使用。它只出问题清单与修法，**不负责重写定义**（那用 agent-designer）。
---

# Agent Rubric

## 先跑官方的，别重复造

官方**已经有**这些校验件 —— 先跑它们，再去补它们不做的：

| 官方件 | 覆盖什么 |
|--------|---------|
| `/doctor prompt-audit`（v2.1.283+） | `.claude/` 下 CLAUDE.md / skills / commands / **subagents** 的「为老模型写的指令」「**引用了不存在的文件或命令**」「**文件互相矛盾**」 |
| `plugins/plugin-dev/agents/plugin-validator.md` | 插件组件校验，description 明写「**proactively** after user creates or modifies plugin components」 |
| `plugins/plugin-dev/skills/agent-development/scripts/validate-agent.sh` | agent md 的 frontmatter 结构、字段格式、system prompt |
| `claude plugin validate <dir>` | marketplace.json / plugin.json 的 JSON 语法与必填字段 |

→ **本 skill 只覆盖它们不做的维度**：路由语义、权限一致性、结构、选边声明。

## 校验维度

### A. 路由（description）

- [ ] 给 **3 句「用户真的会说的话」**，能否命中？命中不了 → 触发语写得太抽象
- [ ] 是否有**边界句**（它不做什么 / 归谁）？
- [ ] 有没有把**只在正文定义的占位符**（如 `{xxx 根}`）写进 `description`？—— 常驻时拿不到定义
- [ ] skill 的 `description` + `when_to_use` 合并后 **≤ 1536 字符**（超了在 listing 里被截断）

### B. 边界与权限一致性 ⭐ 最常翻车

- [ ] `description` **声称的边界**与 `tools` **是否一致**？
  - 反例：description 写「只读 vault，不写简历」，`tools` 里却有 `Write, Edit`
- [ ] **逐个工具问「正文哪一步用到它？」** —— 答不上来的就删
- [ ] 写区是否显式列出「可写 / 不可写」？
- [ ] 能机械化拦截的约束，是不是该转成 **hook** 或 **`permissions.deny`**？（自然语言护栏 **harness 不强制**）

### C. 结构

- [ ] 职责是否**单一**？一个 agent 塞 4-5 块职责 → 该拆
- [ ] 是否把**稳定层**（人设 / 准则）和**易变层**（机器 / 版本 / 技术栈）混在一起？
- [ ] **锚点是不是真的 Markdown 标题**？（写成 `> ### 标题` 会让 `[[file#锚点]]` 指不到）
- [ ] 引用的路径 / 锚点 / 工具名**真的存在吗**？用 `Grep` 全量扫出来逐条核对，不要凭印象
- [ ] 骨架若有偏离（如用「流程」代替「职责 / 模式」），**声明了吗**？

### D. 引用真实性

- [ ] 每个外部仓库 / URL 都核实过吗？有没有编造？
- [ ] 未核实的，是否明确标了「未核实」？
- [ ] 有没有把**作者本地的私人路径 / 笔记**当依据写进要分发的东西里？（读者无法核实）
- [ ] 有没有用「唯一 / 首个 / 没有之一」这类**全称否定**却没给证明？

### E. 选边声明

- [ ] `description` 范式（扁平散文 vs 引号触发语）是**明确选了**还是随手写的？
- [ ] `model` 是显式决策还是照抄 `inherit`？
- [ ] 引用的判据里，哪些是**官方规定**、哪些是**本项目的取舍**？分开了吗？

## 静默失败陷阱

### 4 项**不报错**的（最危险）

| 检查 | 后果 |
|------|------|
| frontmatter 字段名拼写 / 大小写（是 `maxTurns` 不是 `maxturns`） | **静默忽略，不报错** |
| `SKILL.md` 的第一行是不是 `---` | 否则整个文件（含 `---`）被当成正文 |
| `SKILL.md` 的 YAML 能否解析 | 解析失败 → skill 仍加载，但**所有字段为空** |
| `description` + `when_to_use` 是否超 1,536 字符 | 被截断 → 路由依据丢失 |

### 1 项**会硬拒绝**的

| 检查 | 后果 |
|------|------|
| `name` 是否含 `:` | v2.1.218 起**直接被拒绝** |

## 能跑的校验，就跑

| 场景 | 命令 |
|------|------|
| 插件 | `claude plugin validate <dir>` |
| 官方提示词体检 | `/doctor prompt-audit` |
| 路径存在性 | `Grep` 扫引用 + 逐条 `ls` / `Read` |

## 输出格式

```
## 校验结果：<定义文件名>

### 🔴 阻断（必须改）
### 🟡 建议
### ✅ 通过
```

**不许写「看起来没问题」。** 每条通过项都要说清**你实际查了什么**。
