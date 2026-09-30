# frontmatter 全字段参考

> 来源：Claude Code 官方文档 `code.claude.com/docs/en/sub-agents` 与 `/skills`（2026-09-30 核实）。
> 字段名**必须逐字符匹配** —— 拼错的字段会被**静默忽略，不报错**。

---

## 一、Subagent（`.claude/agents/*.md` 或插件 `agents/*.md`）

### 必填（只有这两个）

| 字段 | 说明 |
|------|------|
| `name` | 身份**只由它决定**，文件名不参与判定。**不能含 `:`**（预留给 `my-plugin:reviewer` 这类插件作用域 ID），v2.1.218 起含 `:` 的名字直接被拒绝并记日志 |
| `description` | 何时该派这个 agent —— 即**路由信号** |

### 可选（16 项）

| 字段 | 说明 |
|------|------|
| `tools` | 工具白名单。逗号分隔字符串或 YAML list。**省略 = 继承子代理可用的全部工具**。支持 `mcp__<server>`、`mcp__<server>__*`；主线程模式下可写 `Agent(worker, researcher)` 限制可派生的子代理类型（旧的 `Task(agent_type)` 别名仍可用） |
| `disallowedTools` | 从继承 / 指定列表中移除。**先应用 deny，再解析 allow** |
| `model` | 别名（`sonnet` / `opus` / `haiku` / `fable`）、完整 ID、或 `inherit` |
| `permissionMode` | `default` `acceptEdits` `auto` `dontAsk` `bypassPermissions` `plan`（`manual` 是 `default` 的别名） |
| `maxTurns` | 最大 agentic 轮数 |
| `skills` | 启动时预载入上下文的 skill |
| `mcpServers` | 该子代理可用的 MCP |
| `hooks` | 该子代理作用域的生命周期钩子 |
| `memory` | `user` / `project` / `local` |
| `background` | `true` 时保持后台 |
| `omitClaudeMd` | 不加载任何 CLAUDE.md |
| `effort` | `low` `medium` `high` `xhigh` `max` |
| `isolation` | `worktree` → 临时 git worktree |
| `color` | `red` `blue` `green` `yellow` `purple` `orange` `pink` `cyan` |
| `initialPrompt` | 作为主会话 agent 时自动提交的首个用户轮 |
| `experimental` | 实验项 map，如 `cacheTtl: 5m` / `1h` |

> ⚠️ **插件里的 subagent 有三个字段不生效**：`hooks`、`mcpServers`、`permissionMode` —— 从插件加载时**被忽略**。若要这三个，必须装到 `.claude/agents/` 或 `~/.claude/agents/`。
>
> ⚠️ **部分字段有版本门槛**（`name` 含 `:` 的拒绝、`omitClaudeMd`、`experimental`、`manual` 别名等）。**升级或排错时以官方文档为准**，别照抄二手版本号。

### `model` 解析优先级

1. 调用时的 `model` 参数
2. frontmatter 的 `model`（`inherit` 取主会话模型）
3. `CLAUDE_CODE_SUBAGENT_MODEL` 环境变量
4. 主会话模型

### 文件位置与优先级

Managed settings > `--agents` CLI flag > `.claude/agents/` > `~/.claude/agents/` > 插件的 `agents/` 目录。
项目 / 用户目录**递归扫描**。

---

## 二、Skill（`.claude/skills/<name>/SKILL.md` 或插件 `skills/<name>/SKILL.md`）

**没有任何字段是硬性必填**；`description` 标为 Recommended。
命名规则：**小写单词用连字符连接**，唯一例外是 `when_to_use`。

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | No | `/` 菜单里的命令名，默认取目录名 |
| `description` | **Recommended** | Claude 据此判断何时调用；省略则取正文第一非空行 |
| `when_to_use` | No | 追加到 `description`，**合并后 1,536 字符截断** |
| `argument-hint` / `arguments` | No | 自动补全提示 / 命名位置参数（`$name` 替换） |
| `disable-model-invocation` | No | `true` = 禁止 Claude 自动加载，同时阻止被 preload 进子代理 |
| `user-invocable` | No | `false` = 只允许 Claude 调用，从 `/` 菜单隐藏 |
| `allowed-tools` | No | 调用轮次内免权限提示；**下一条消息即失效** |
| `disallowed-tools` | No | 该 skill 激活期间移出工具池 |
| `model` / `effort` | No | 仅当前轮生效，不写入 settings |
| `context` | No | `fork` = 在 fork 出的子代理上下文中运行 |
| `agent` | No | `context: fork` 时用哪个子代理类型（默认 `general-purpose`） |
| `background` | No | 仅 `context: fork` 生效，默认 `true` |
| `hooks` / `paths` / `shell` | No | 钩子 / 路径 glob 限定自动激活 / `bash`\|`powershell` |
| `metadata` / `license` / `compatibility` | No | 三者均属 **Agent Skills 规范**（跨工具兼容子集），Claude Code 接受但不作用 |

布尔值自 v2.1.218 起接受 `yes/no/on/off/1/0`（任意大小写）。

### 跨工具兼容子集

上传到 claude.ai / Skills API / 经 `package_skill.py` 打包时，**只认这六个**：
`name`、`description`、`license`、`compatibility`、`metadata`、`allowed-tools`。
其余字段会报错，例如：`Unexpected key(s) in SKILL.md frontmatter: argument-hint.`

---

## 三、静默失败陷阱

### 4 项**不报错**的（最危险）

| 检查 | 后果 |
|------|------|
| subagent frontmatter 字段名拼错 / 大小写错（是 `maxTurns` 不是 `maxturns`） | **静默忽略，不报错**，行为莫名降级 |
| `SKILL.md` 的第一行是不是 `---` | 否则整个文件（含 `---`）被当成 skill 正文 |
| `SKILL.md` 里的 YAML 能否解析 | 解析失败 → skill 仍加载，但**所有字段为空** |
| `description` + `when_to_use` 是否超 1,536 字符 | 超了在 listing 中**被截断** → 路由依据丢失 |

### 1 项**会硬拒绝**的（会报错，但仍列在此便于一起查）

| 检查 | 后果 |
|------|------|
| `name` 是否含 `:` | v2.1.218 起**直接被拒绝**（`:` 预留给插件作用域 ID） |

---

## 四、目录名速查（写错不报错，只是不生效）

| 载体 | 项目级 | 用户级 | 插件 |
|------|--------|--------|------|
| Agent | `.claude/agents/` | `~/.claude/agents/` | `<插件根>/agents/` |
| Skill | `.claude/skills/<name>/` | `~/.claude/skills/<name>/` | `<插件根>/skills/<name>/` |

⚠️ 插件里**没有** `.claude/` 前缀 —— 这是最容易照印象写错的地方。
