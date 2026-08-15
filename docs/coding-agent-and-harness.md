# Coding Agent 与 Agent Harness 学习指南

> 目标：把 OpenAI Agents SDK / Codex、Claude Code、OpenCode 当作“生产级 Agent 系统样本”来研究，而不只是当成 AI 编码工具使用。
>
> 核心问题：一个 Agent 从“模型会回答问题”走到“能够在真实代码仓库里持续完成任务”，中间究竟多了哪些 runtime / harness 能力？

---

## 一、为什么 Coding Agent 值得单独学习

普通 ReAct Demo 通常只有：

`用户输入 -> LLM -> Tool Call -> Observation -> LLM -> 输出`

而真正的 Coding Agent 还要处理：

- 大型代码仓库探索；
- 文件系统与 Shell；
- 搜索、编辑、补丁、构建、测试；
- 长任务状态维护；
- context compaction；
- 权限与人工审批；
- sandbox / workspace isolation；
- persistent instructions；
- Skills；
- subagents / agent teams；
- hooks；
- MCP；
- 错误恢复与重试；
- Git / PR 等软件工程工作流；
- 可观测性、成本与安全。

这层把“Agent 算法”变成“可运行的 Agent 系统”。在 2026 年国内部分 Agent / AI Coding JD 中，Claude Code、Harness Engineering、Skills、MCP、AgentOps 等已经直接进入岗位要求，因此应该纳入学习主线，而不是只在求职前临时体验工具。

---

## 二、先建立统一分析框架

学习任何 Coding Agent 时，不先背命令，而是按下面 10 个问题分析。

### 1. Agent loop

- 一次任务的主循环是什么？
- 模型什么时候继续思考，什么时候调用工具？
- 如何判断任务完成？
- 如何避免无限循环？

### 2. Tools

- 有哪些内置工具？
- 文件、Shell、搜索、网络分别如何暴露给模型？
- 工具 schema 如何设计？
- 工具返回过长怎么处理？

### 3. Workspace / Sandbox

- Agent 在哪里执行命令？
- 能访问哪些文件？
- 是否有网络权限？
- 是否隔离高风险操作？

### 4. Permissions / Human-in-the-loop

- 哪些操作自动执行？
- 哪些操作需要确认？
- 权限是否可以按工具、命令或目录配置？
- 如何防止 prompt injection 导致破坏性操作？

### 5. Context Engineering

- 代码库如何进入上下文？
- 是否先搜索再读取？
- 长任务如何压缩历史？
- Tool result 如何清理？

### 6. Persistent Instructions

- 项目级规则如何长期保存？
- 指令如何分层、覆盖和继承？
- 如何避免规则越来越长导致上下文污染？

典型形式包括 `AGENTS.md`、`CLAUDE.md` 等。

### 7. Skills

- Skill 和普通 Prompt 有什么区别？
- Skill 什么时候加载？
- 是否包含脚本 / 资源 / 参考资料？
- Skill 如何组合、复用和版本化？

### 8. Subagents / Multi-Agent

- 子 Agent 是否拥有独立上下文？
- 如何委派任务？
- 是否并行执行？
- 主 Agent 如何汇总结果？
- 使用子 Agent 带来的 token / latency / coordination cost 是多少？

### 9. Hooks / Lifecycle

- 能否在 tool call 前后执行确定性代码？
- 能否做审计、格式化、测试、阻断危险操作？
- Agent 的 session / lifecycle 有哪些事件？

### 10. Evaluation / Recovery

- 如何判断“完成任务”而不是“生成了代码”？
- 是否自动运行 test / lint / build？
- 失败后如何定位和修复？
- 长任务中断后能否恢复？

---

## 三、OpenAI：Agents SDK 与 Codex

OpenAI 这一条线建议拆成两层学习。

### 3.1 OpenAI Agents SDK：通用 Agent runtime

官方资料：

- Agents SDK Python：<https://openai.github.io/openai-agents-python/>
- Agents：<https://openai.github.io/openai-agents-python/agents/>
- Running agents：<https://openai.github.io/openai-agents-python/running_agents/>
- Tools：<https://openai.github.io/openai-agents-python/tools/>
- Guardrails：<https://openai.github.io/openai-agents-python/guardrails/>
- Handoffs：<https://openai.github.io/openai-agents-python/handoffs/>
- Tracing：<https://openai.github.io/openai-agents-python/tracing/>
- Multi-agent orchestration：<https://openai.github.io/openai-agents-python/multi_agent/>

重点不是记 API，而是研究：

1. Agent、Runner 与 agent loop 如何分工；
2. function tools 如何进入 loop；
3. agent-as-tool 与 handoff 两种多 Agent 编排方式有什么差异；
4. session / conversation state 如何管理；
5. input / output / tool guardrails 分别拦什么；
6. tracing 能看到哪些运行信息；
7. MCP 如何进入 Agent runtime；
8. SDK 帮你隐藏了哪些自己手写 Agent loop 时必须处理的细节。

### 3.2 Codex：Coding Agent 系统样本

官方资料：

- Codex 文档入口：<https://developers.openai.com/codex/>
- Codex CLI：<https://developers.openai.com/codex/cli/>
- Codex Skills：<https://developers.openai.com/codex/skills/>
- Codex MCP：<https://developers.openai.com/codex/mcp/>
- Codex GitHub：<https://github.com/openai/codex>

建议重点观察：

- Codex 如何理解一个陌生 repository；
- 如何读取、修改、运行代码；
- approval / sandbox 的边界；
- `AGENTS.md` 等项目级指令如何影响行为；
- Skills 与 MCP 如何扩展能力；
- 本地 CLI 与远程 / 长任务 Agent 的差异；
- Agent 如何把“修改代码”闭环到 build / test / review。

### 3.3 OpenAI 实践任务

不要只看文档，做三个对照实验：

1. **手写 Agent loop vs Agents SDK**：实现同一个三工具任务，对照 SDK 替你处理了哪些状态和异常。
2. **Codex 仓库任务**：给一个陌生小仓库修复真实 bug，记录它的搜索、读取、修改和测试路径。
3. **权限实验**：分别允许 / 禁止 Shell、网络和高风险操作，观察 agent plan 如何变化。

---

## 四、Anthropic：Claude Code 与 Agent SDK

Claude Code 很适合研究 Agent Harness，因为它把 Skills、Hooks、Subagents、MCP、persistent instructions、permissions 等机制暴露得比较清楚。

官方资料：

- Claude Code Overview：<https://code.claude.com/docs/en/overview>
- Claude Code 扩展机制：<https://code.claude.com/docs/en/features-overview>
- Skills：<https://code.claude.com/docs/en/skills>
- Subagents：<https://code.claude.com/docs/en/sub-agents>
- Hooks：<https://code.claude.com/docs/en/hooks-guide>
- MCP：<https://code.claude.com/docs/en/mcp>
- Memory / CLAUDE.md：<https://code.claude.com/docs/en/memory>
- Permissions：<https://code.claude.com/docs/en/permissions>
- Agent SDK：<https://platform.claude.com/docs/en/agent-sdk/overview>

### 4.1 Claude Code 要重点研究什么

#### CLAUDE.md / Memory

理解 persistent instructions 与普通 Prompt 的差异：

- 为什么项目规范应该外置；
- 如何做 repository-level instructions；
- 规则冲突与上下文膨胀如何处理。

#### Skills

重点理解 progressive disclosure：Agent 不需要把所有能力说明一次性塞进 context，而是在需要时加载 Skill。

观察：

- `SKILL.md` 的角色；
- description 如何影响调用；
- Skill 能否附带脚本、模板和资料；
- Skill 与 MCP Tool 的边界。

#### Subagents

观察：

- 独立上下文是否减少主 Agent 污染；
- 什么任务适合委派；
- 并行 Agent 的成本与结果聚合问题；
- 子 Agent 的工具 / 权限如何隔离。

#### Hooks

Hooks 是理解“LLM 决策 + deterministic control”混合系统的好例子。

研究它如何用于：

- tool call 前阻断危险操作；
- 自动格式化；
- 自动测试；
- 审计；
- 注入环境信息；
- 生命周期控制。

#### Permissions

不要把 permissions 当设置页功能，而要把它理解成 Agent 安全模型的一部分。

要能回答：

> 为什么一个能够执行 Shell 的 Agent 必须把 capability、policy、approval 和 sandbox 分开设计？

### 4.2 Claude Agent SDK

把 Claude Code 看成产品，把 Agent SDK 看成可编程 runtime。

重点观察 SDK 暴露的：

- Read / Write / Edit / Bash / Glob / Grep；
- WebSearch / WebFetch；
- Hooks；
- Subagents；
- MCP；
- Permissions；
- Sessions。

目标是理解：如果自己要做一个“某领域的 Claude Code”，需要哪些系统组件。

---

## 五、OpenCode：开源 Coding Agent 样本

OpenCode 的价值在于：它是开源 Coding Agent，可以把“使用体验”进一步推进到“源码阅读与架构分析”。

官方资料：

- 官方文档：<https://opencode.ai/docs>
- Agents：<https://opencode.ai/docs/agents>
- Permissions：<https://opencode.ai/docs/permissions>
- MCP Servers：<https://opencode.ai/docs/mcp-servers>
- Rules / AGENTS.md：<https://opencode.ai/docs/rules>
- GitHub：<https://github.com/anomalyco/opencode>

### 5.1 重点机制

观察 OpenCode 的：

- Build / Plan 等 primary agent；
- Explore / General 等 subagent；
- hidden compaction / summary agent；
- tool permissions；
- MCP tool 权限；
- `AGENTS.md` 项目规则；
- session / provider abstraction；
- compaction；
- 多模型 / 多 provider 支持。

### 5.2 为什么要读 OpenCode 源码

只使用闭源 Coding Agent，你能看到“行为”；读一个开源 Coding Agent，则可以追到：

`用户任务 -> system prompt -> agent state -> model call -> tool registry -> permission -> tool execution -> message history -> compaction -> 下一轮 model call`

建议至少追一次完整调用链，并画出自己的架构图。

不要求第一次就读完整项目，优先回答五个问题：

1. Agent loop 在哪里？
2. Tool 如何注册和调用？
3. Permission 在哪一层判断？
4. Context 超限后谁负责 compaction？
5. Primary agent 和 subagent 如何切换？

---

## 六、三套系统横向比较时应该比较什么

不要比较“谁更聪明”，而比较系统设计。

| 维度 | OpenAI Agents SDK / Codex | Claude Code / Agent SDK | OpenCode |
| --- | --- | --- | --- |
| 学习价值 | 通用 Agent runtime + Coding Agent | Harness 机制暴露清晰 | 开源、适合源码追踪 |
| Agent loop | 重点理解 Runner / loop | 产品 + SDK 两层观察 | 可直接从源码追踪 |
| Persistent instructions | Codex 项目规则 / AGENTS.md | CLAUDE.md / Memory | AGENTS.md / Rules |
| Tool 扩展 | Function tools / MCP | Built-in tools / MCP | Tools / MCP |
| Skills | Codex Skills | Claude Code Skills | 观察其 Agent / command / extension 机制 |
| Subagents | agents-as-tools / handoffs 等 | Subagents / Agent teams | Primary / subagents |
| Hooks | 按 OpenAI runtime 能力理解 | Claude Code Hooks 很适合重点研究 | 结合插件 / runtime 源码分析 |
| Permissions / Sandbox | Codex approval / sandbox | Claude Code permissions | Permission rules，源码可追踪 |
| Context 管理 | Agent / Codex runtime | Memory / compaction / subagent context | compaction / summary agents |
| 最适合的学习方式 | SDK 实验 + Codex 实际任务 | 功能机制实验 | 源码阅读 |

注意：产品功能会持续变化，表格的目的不是长期记录“谁有 / 没有某功能”，而是给出当前学习观察框架。具体能力以各项目最新官方文档为准。

---

## 七、建议学习顺序

### 阶段 1：会用，但不陷入工具细节

1. 分别用 Codex、Claude Code、OpenCode 完成同一个小型代码任务；
2. 记录它们如何搜索代码、调用 Shell、修改文件、执行测试；
3. 重点记录失败路径，不比较主观“回答质量”。

### 阶段 2：拆机制

4. 学 persistent instructions；
5. 学 permissions / approvals；
6. 学 MCP；
7. 学 Skills；
8. 学 subagents；
9. 学 hooks / deterministic control；
10. 学 compaction / long-running context。

### 阶段 3：源码与 SDK

11. 用 OpenAI Agents SDK 或 Claude Agent SDK 写一个最简 coding agent；
12. 阅读 OpenCode 的 agent loop 与 tool execution 路径；
13. 自己实现最小 permission layer；
14. 加入 test-before-finish；
15. 加入失败恢复和 trace。

### 阶段 4：形成面试表达

最终应能回答：

- Coding Agent 和普通 Chat Agent 的核心系统差异是什么？
- Agent Harness 到底负责什么？
- 为什么 Skills 不等于 Tools？
- 为什么 Hooks 仍然必要，不能所有行为都交给 LLM？
- 为什么 subagent 既能缓解 context pressure，也会增加成本和协调失败？
- permission 与 sandbox 有什么区别？
- 如何验证 Coding Agent 真正修好了代码，而不是只生成了看起来合理的 patch？

---

## 八、建议产出

这一模块不需要再做一个大型项目，建议留下四份产物：

1. `notes/coding-agent-comparison.md`：三套系统机制横向比较；
2. `notes/opencode-source-walkthrough.md`：OpenCode 一次完整调用链源码走读；
3. `experiments/minimal-coding-agent/`：自己实现的最小 Coding Agent；
4. `experiments/agent-permission-eval/`：权限 / sandbox / prompt injection 小实验。

这些产物比“我平时经常用 Claude Code / Codex”更能证明你真正理解 Agent Engineering。

---

## 九、学习边界

这部分容易无限扩张，需要主动限制：

- 不追每一个 CLI 命令；
- 不追每个产品版本的新功能；
- 不以“同时精通三个工具”为目标；
- 不把产品使用技巧替代 Agent 原理；
- 不为了 Multi-Agent 而 Multi-Agent；
- 不需要复刻完整 Claude Code / Codex。

学习的终点是：

> **能够从一个成熟 Coding Agent 反推 Agent runtime、harness、tooling、context、permission、evaluation 的工程设计，并把这些原则迁移到自己的 Agent 项目。**
