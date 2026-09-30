---
name: agent-anatomy
description: Agent 定义的解剖参考 —— subagent / Skill / CLAUDE.md / Hook 的**选择树**、agent 与 skill 的**全字段 frontmatter 表**、`description` 的两套竞争范式与选边、正文骨架、安装位置与优先级。当你要判断「这个需求该做成哪种载体」或「定义文件该长什么样」时使用。只提供参考与骨架，**不负责走完整设计流程**（那用 agent-designer）；**不负责校验**（那用 agent-rubric）。
---

# Agent Anatomy

## 1. 选择树：这个需求该做成什么？

**按顺序问，第一个答「是」的就是答案：**

| 顺序 | 问题 | 是 → |
|---|---|---|
| 1 | 需要**独立评判**（生成者 ≠ 评判者）或**独立上下文**？ | **Subagent** |
| 2 | 只是要一段**可复用的知识 / 流程**，按需加载？ | **Skill** |
| 3 | 只是要**项目级常驻规则**，每次都该生效？ | **CLAUDE.md** |
| 4 | 需要**确定性拦截**某个动作（不能靠模型自觉）？ | **Hook** |
| 5 | 以上都不是 | **什么都别建** |

> 依据：「subagent 的触发条件只有两个：需要独立评判、需要隔离上下文」。
> 核心原则：**能机械执行的约束就别写进提示词**。「模型可能不遵守」的规则 → hook；「模型需要知道」的知识 → skill。

## 2. 关键分野：Skill vs Subagent

| 维度 | Skill | Subagent |
|------|-------|----------|
| 上下文 | 与主对话**共享** context window | **独立** context window |
| 正文进入方式 | 作为**一条消息**进入，后续轮次持续存在 | 成为该子代理的 system prompt |
| 后续轮次是否重读 | **不会重读** —— 要贯穿全程的指导须写成常驻指令 | n/a |
| 系统提示来源 | —— | 子代理 md 正文（**不是** Claude Code 的系统提示） |
| 适合 | 可复用知识 / 流程 | 隔离 + 独立权限 + 独立模型档 |

> ⚠️ `context: fork` 只对**有明确任务指令**的 skill 有意义。若 skill 通篇是「用这些 API 约定」这类指南而无任务，子代理会「returns without meaningful output」。

## 3. frontmatter

### 3.1 Subagent（`.claude/agents/*.md`，或插件根下的 `agents/*.md`）

**必填只有两个**：`name`、`description`。其余 17 个可选字段见
→ `references/frontmatter-reference.md`

常写的是「四件套」：

```yaml
---
name: kebab-case-不要含冒号
description: 三段式，见 §4
tools: Read, Glob, Grep, Bash, WebSearch, WebFetch   # 最小白名单；省略 = 继承全部
model: inherit                                        # 或 sonnet / opus / haiku / 完整 ID
---
```

### 3.2 Skill（`.claude/skills/<name>/SKILL.md`）

**没有硬性必填**；`description` 标为 Recommended。全字段表见
→ `references/frontmatter-reference.md`

## 4. `description` 的两套竞争范式（必须选边）

同一个官方仓库里并存两种写法，**互相打架**：

| | **A. 扁平散文**（官方 reference 规定） | **B. 引号触发语**（官方自己的 agent-creator 在用） |
|---|---|---|
| 形态 | 单行散文 | block scalar + `<example>` 块 |
| 触发语 | 名词短语，描述**情境** | 直接引**用户原话** |
| 官方态度 | reference 明确要求，并写明「**不要**用引号复述用户原话」 | 官方 `agent-creator` 自己违反了这条 |

**默认选边：B（引号触发语）** —— 中文场景下「用户会怎么说」比抽象情境更好命中。
**但必须在设计说明里写明这是「选边」，不是默认。**

房子风格的 B 写法（三段）：

```
<中文触发语，用「」引原话> —— <能力，破折号连接>。它<写区 / 能力边界>；<面向用户的动作>由<谁>完成。
```

## 5. 正文骨架（房子风格）

```
# <角色名>
> <一句话定位 + 与相似 agent 的差别>

## 📍 路径锚点          ← 真标题，不要写进 > 引用块
## 工作入口（先读什么）
## 职责 / 模式（编号枚举：何时用 + 做什么 + 产出）
## 🚨 硬性护栏           ← 用显式否定句
## 输出规范
```

⚠️ **锚点必须是真的 Markdown 标题。** 写成 `> ### 📍 路径锚点`（塞进引用块）会让 `[[file#锚点]]` 指不到 —— 文件里看着像有这一节，链接却永远失效。

## 6. 安装位置与优先级

| 载体 | 项目级 | 用户级 |
|------|--------|--------|
| Agent | `.claude/agents/` | `~/.claude/agents/` |
| Skill | `.claude/skills/<name>/SKILL.md` | `~/.claude/skills/<name>/SKILL.md` |
| 插件 | 插件根下 `agents/`、`skills/`（**无** `.claude/` 前缀） | —— |

Agent 解析优先级：Managed settings > `--agents` > 项目级 > 用户级 > 插件。
身份**只由 `name` frontmatter 决定**，文件名不参与判定。

## References

- `references/frontmatter-reference.md` —— 全字段表（agent 17 项 + skill 全项）+ 静默失败陷阱
- `references/examples.md` —— 好 / 坏范例对照
