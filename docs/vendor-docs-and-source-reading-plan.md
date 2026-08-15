# AI 厂商官方文档与源码阅读计划

> 目标：把“读官方文档 / 读源码”变成 Agent 学习路线中的正式方法，而不是零散收藏项目。
>
> 本文的优先级按 **当前国内 Agent / LLM 应用实习的学习收益、可迁移性、源码可读性与时间投入产出比** 排序，不代表厂商综合技术实力排名。
>
> Coding Agent 的专项机制学习另见 [`Coding Agent 与 Agent Harness 学习指南`](./coding-agent-and-harness.md)。

---

## 一、为什么要读厂商官方文档和源码

Agent 方向变化很快，仅靠教程容易出现三个问题：

1. 学到的是某个框架某个版本的 API，而不是可迁移的设计思想；
2. 教程通常只覆盖 happy path，看不到权限、上下文、失败恢复、评测与运行时约束；
3. 很多真正重要的新工程实践，往往先出现在厂商文档、SDK、CLI 和开源项目中，再慢慢沉淀为课程或教材。

因此源码阅读不是为了“把几十万行代码全部看完”，而是回答具体问题：

```text
用户任务如何进入 Agent？
模型在哪里被调用？
Tool 如何注册、选择和执行？
Tool result 如何回到上下文？
上下文什么时候压缩？
权限和 approval 在哪一层？
状态 / session 如何保存？
长任务失败后如何恢复？
Skills / subagents / hooks 如何接入？
trace / eval 如何观察系统？
```

如果读完一个项目仍然回答不了这些问题，就说明阅读停留在 README / API 表面。

---

## 二、三种阅读对象要分开

不要把所有厂商项目都当成同一类源码。

### 2.1 Agent Runtime / Harness

重点研究：

- Agent loop；
- Tool runtime；
- shell / filesystem / browser；
- sandbox；
- permissions / approval；
- context management；
- persistent instructions；
- Skills；
- subagents；
- hooks；
- long-running tasks；
- recovery / verification。

典型对象：Codex、Claude Code / Agent SDK、OpenCode、Kimi Code、Trae-Agent、DeerFlow。

### 2.2 通用 Agent Framework

重点研究：

- Agent 抽象；
- LLM adapter；
- Tool registry；
- memory；
- MCP；
- RAG；
- multi-agent orchestration；
- serving / evaluation。

典型对象：Qwen-Agent、AgentScope、OpenAI Agents SDK。

### 2.3 Agentic Model / Reasoning / 后训练

重点研究：

- 模型为什么更适合 reasoning / tool use / coding；
- long-horizon 行为；
- RL / post-training；
- inference；
- context；
- 模型能力与 Agent harness 的边界。

典型对象：Kimi K3、DeepSeek-R1 / V3、GLM-5、Seed。

这三类学习目标不同。不要因为 DeepSeek 很重要，就强行把 DeepSeek-R1 当作 Coding Agent runtime 来读。

---

## 三、优先级总表

| 优先级 | 厂商 / 项目 | 主要学习对象 | 最值得学的内容 | 推荐阅读方式 |
| --- | --- | --- | --- | --- |
| S | OpenAI | Agents SDK、Codex | Agent runtime、tools、handoffs、guardrails、sandbox / approval、Skills、MCP、trace / eval | 官方文档 + SDK / 可获得源码 + 对照实验 |
| S | Anthropic | Claude Code、Claude Agent SDK | Context Engineering、CLAUDE.md、Skills、Subagents、Hooks、Permissions、MCP、long-running harness | 官方文档优先 + SDK + 机制实验 |
| S | Moonshot / Kimi | Kimi K3、Kimi Code | Agentic model、long-horizon coding、tool use、session、approval、MCP / ACP、Coding Agent harness | 技术报告 + 官方仓库 + Kimi Code 源码 |
| S | Qwen | Qwen-Agent | 通用 Python Agent Framework、Tool Calling、MCP、RAG、Code Interpreter、Browser | **重点读源码** |
| S | ByteDance | Trae-Agent、DeerFlow | Coding Agent、SuperAgent、Sandbox、Memory、Skills、Subagents、长任务 | **重点读源码** |
| A+ | MiniMax | Mini-Agent | 小而完整的 Agent pipeline、production feature、源码组织 | **第一次完整源码阅读首选** |
| A | Z.ai / GLM | GLM-5、GLM-V | Agentic Engineering、long-horizon model、GUI / multimodal Agent | 技术报告 + 关键实现选读 |
| A | DeepSeek | DeepSeek-R1、DeepSeek-V3、awesome-deepseek-agent | reasoning / RL、模型架构、推理系统，以及第三方 Agent 如何适配 DeepSeek | 模型报告 / 关键实现为主 |
| A | AgentScope | AgentScope | Production Agent Framework、Memory、MCP / A2A、Eval、Serving、多 Agent | 架构与核心模块源码 |
| B | UI-TARS / GUI Agent | UI-TARS-desktop、GLM-V | Computer Use、GUI Agent、视觉-动作闭环 | 专项需要时再深入 |
| B | ByteDance Seed | Seed-1.8 等 | perception-reasoning-action、Agentic model / RL | 算法方向再深入 |

### 当前阶段的核心原则

**S 级不是都要求通读全部源码。**

真正要求“完整追调用链”的项目只选少数几个；其他项目围绕具体问题定向阅读。

---

## 四、S 级：当前最值得投入时间

## 4.1 OpenAI：Agent Runtime / Codex / Evals

### 为什么是 S 级

OpenAI 这条线适合把 Agent 的几个层次串起来：

```text
模型 API
  ↓
Function Calling / Tools
  ↓
Agents SDK
  ↓
Agent orchestration / tracing / guardrails
  ↓
Codex Coding Agent
  ↓
Sandbox / Approval / Skills / MCP / 软件工程闭环
```

它的价值不是学某一个 OpenAI API，而是观察从“调用模型”到“成熟 Agent Runtime / Coding Agent”之间增加了哪些工程层。

### 重点资料

- OpenAI Agents SDK：https://openai.github.io/openai-agents-python/
- Codex：https://developers.openai.com/codex/
- Function Calling：https://developers.openai.com/api/docs/guides/function-calling
- Production Best Practices：https://developers.openai.com/api/docs/guides/production-best-practices

### 要回答的问题

- Agents SDK 的 Agent / Runner / Tool / Handoff / Guardrail 各处于什么层？
- tracing 为什么要成为 Agent runtime 的一等能力？
- Codex 如何处理 repo exploration、shell、文件修改与验证？
- sandbox 和 approval 为什么不是普通聊天 Agent 的附属功能，而是执行型 Agent 的核心安全边界？
- Skills 与普通 system prompt / tool description 的差别是什么？

### 阅读深度

**高。**官方文档认真读；Agents SDK 追一条最小调用链；Codex 重点研究 runtime / tools / sandbox / Skills，不要求机械通读所有实现。

---

## 4.2 Anthropic：Claude Code / Context Engineering / Harness

### 为什么是 S 级

Anthropic 当前最值得学习的不是“Claude 模型怎么调用”，而是它围绕长程 Agent 工程形成的一套方法：

- context engineering；
- tool design；
- persistent instructions；
- Skills；
- subagents；
- hooks；
- permissions；
- long-running agent harness；
- eval。

Claude Code 是理解这些机制如何落地到真实 Coding Agent 的成熟样本。

### 重点资料

- Claude Code Docs：https://code.claude.com/docs/en/overview
- Anthropic Engineering：https://www.anthropic.com/engineering
- Claude Cookbook：https://platform.claude.com/cookbook

### 要回答的问题

- `CLAUDE.md` / persistent instructions 为什么需要独立于普通对话历史？
- Skill 与 Subagent 的边界是什么？
- Hook 是扩展点、控制点还是安全点？
- Permissions 如何限制真正会执行系统操作的 Agent？
- compaction 如何支撑长任务？
- 为什么 Context Engineering 往往比继续堆 prompt 更重要？

### 阅读深度

**高。**以官方文档和机制实验为主；对 Agent SDK / 可获得实现做定向源码阅读，不以“必须通读 Claude Code 全部实现”为目标。

---

## 4.3 Moonshot / Kimi：Kimi K3 + Kimi Code

### 为什么是 S 级

Kimi 这一条最有价值的地方是同时有：

```text
Agentic Model
    +
Coding Agent Harness
```

因此可以研究“模型能力”和“Agent runtime”是如何配合的。

### 重点项目

- Kimi K3：https://github.com/MoonshotAI/Kimi-K3
- Kimi Code：https://github.com/MoonshotAI/kimi-code

### Kimi K3 重点

不要把目标设成读完整模型训练代码，而是理解：

- 为什么模型面向 long-horizon coding / agentic tasks 设计；
- tool use；
- reasoning / preserved thinking；
- context；
- 模型能力如何影响 Agent loop；
- 模型和 Kimi Code harness 各负责什么。

### Kimi Code 重点

追踪：

```text
User Task
  ↓
Session / Context
  ↓
Model
  ↓
Tool Decision
  ↓
Filesystem / Shell / Search
  ↓
Approval / Runtime
  ↓
Observation
  ↓
Context Update
  ↓
Next Step
```

重点查：

- agent loop；
- session；
- tools；
- approval；
- MCP；
- ACP；
- context；
- command / editing workflow；
- failure recovery。

### 阅读深度

Kimi K3：**技术报告 + 关键实现，中高深度。**

Kimi Code：**重点源码阅读，高深度。**

---

## 4.4 Qwen：Qwen-Agent

### 为什么是 S 级

Qwen-Agent 很适合回答：

> 一个通用 Python Agent Framework 到底怎么组织？

相比直接进入大型 Coding Agent，它的抽象更传统、更容易看清：

- LLM wrapper；
- Agent abstraction；
- Tool Calling；
- Tool registry；
- MCP；
- RAG；
- Code Interpreter；
- Browser；
- Memory / messages。

### 官方项目

- Qwen-Agent：https://github.com/QwenLM/Qwen-Agent

### 源码阅读目标

至少完整追一次：

```text
User Message
  ↓
Agent.run()
  ↓
LLM Adapter
  ↓
Tool Call
  ↓
Tool Registry
  ↓
Tool Execution
  ↓
Observation
  ↓
Messages / Memory
  ↓
Next LLM Call
```

然后再定向看：

- MCP 如何接入；
- RAG 如何作为 Agent 能力组合；
- Code Interpreter 如何封装；
- Browser Agent 如何组织；
- 不同模型后端如何适配。

### 阅读深度

**非常高。**这是建议真正做源码笔记、架构图和调用链图的项目之一。

---

## 4.5 ByteDance：Trae-Agent + DeerFlow

### 为什么是 S 级

字节这条线适合把“Coding Agent”继续扩展到“SuperAgent / long-horizon harness”。

### 重点项目

- Trae-Agent：https://github.com/bytedance/trae-agent
- DeerFlow：https://github.com/bytedance/deer-flow

### Trae-Agent 学什么

- Coding Agent loop；
- repo / file / shell tool；
- 软件工程任务如何表达；
- edit -> test -> feedback -> repair；
- Coding Agent 和通用 Chat Agent 的区别。

### DeerFlow 学什么

- Sandbox；
- Memory；
- Tools；
- Skills；
- Subagents；
- long-running task；
- SuperAgent harness；
- 更复杂的任务拆解与执行。

### 阅读深度

Trae-Agent：**中高。**先追主调用链。

DeerFlow：**高，但后置。**等 Mini-Agent / Qwen-Agent / 一个 Coding Agent 已经读懂后再进入，否则容易被工程规模淹没。

---

## 五、A+：最适合作为第一次完整源码阅读

## 5.1 MiniMax Mini-Agent

### 为什么单独列 A+

它不一定是你最终最需要掌握的框架，但它对“学习源码”本身的投入产出比非常高。

官方项目：

- Mini-Agent：https://github.com/MiniMax-AI/Mini-Agent

相较大型 Agent 工程，它规模更可控，适合第一次建立完整源码阅读方法。

### 第一次阅读必须产出

1. 项目目录结构图；
2. Agent 主调用链；
3. Tool registry / execution 路径；
4. context / state 流转；
5. error handling；
6. 你认为可替换 / 可扩展的模块；
7. 和自己手写 Minimal Agent 的差异表。

### 阅读深度

**完整阅读核心源码。**

不是要求逐字符看所有文件，而是核心 Agent pipeline 必须完整追通。

---

## 六、A 级：高价值，但根据方向选读

## 6.1 Z.ai / GLM：Agentic Engineering + GUI Agent

重点项目：

- GLM-5：https://github.com/zai-org/GLM-5
- GLM-V：https://github.com/zai-org/GLM-V

### 为什么值得读

更适合研究：

- long-horizon agentic model；
- coding / tool-use model；
- 多模态 Agent；
- Vision Function Calling；
- GUI Agent。

### 推荐深度

当前主线：技术报告 / README / 关键示例。

进入 Browser / Computer Use / Multimodal Agent 后：再深入 GLM-V 等实现。

---

## 6.2 DeepSeek：重点读模型，不把它误定位成 Agent Harness

重点项目：

- DeepSeek-R1：https://github.com/deepseek-ai/DeepSeek-R1
- DeepSeek-V3：https://github.com/deepseek-ai/DeepSeek-V3
- awesome-deepseek-agent：https://github.com/deepseek-ai/awesome-deepseek-agent

### 为什么值得读

DeepSeek 对 Agent 学习的价值主要在 Agent 背后的模型层：

```text
DeepSeek-V3
→ MoE / MLA / inference

DeepSeek-R1
→ reasoning
→ RL
→ cold start / SFT / RL
→ distillation
```

如果以后转 Agent 算法 / Agentic RL / AI Infra，优先级可以直接提升到 S。

### `awesome-deepseek-agent` 怎么看

它适合观察：

> 不同 Coding Agent / Agent 工具如何适配 DeepSeek 模型。

但必须区分：

```text
DeepSeek 官方收录 / 推荐某个 Agent 集成
≠
该 Agent runtime 本身由 DeepSeek 官方开发
```

因此第三方项目可以用于研究“模型如何接入 Agent”，不能据此推断“DeepSeek 官方 Agent 架构”。

### 推荐深度

当前应用岗准备：R1 / V3 技术报告和关键实现选读。

算法岗准备：系统深入 RL、post-training、inference。

---

## 6.3 AgentScope：Production Agent Framework

官方项目：

- AgentScope：https://github.com/agentscope-ai/agentscope

### 为什么值得读

它适合研究教程 Demo 之外的 Production Agent Framework：

- ReAct / Agent abstraction；
- tools；
- Skills；
- memory；
- planning；
- HITL；
- evaluation；
- MCP / A2A；
- multi-agent；
- observability；
- serving；
- 多 session / deployment。

### 推荐深度

在 Qwen-Agent 之后阅读。

不要一开始通读整个框架；先选一条 Agent -> Tool -> Memory -> Serving 的链路，再根据主项目需要扩展。

---

## 七、B 级：专项需要时再深入

## 7.1 UI-TARS / GUI Agent

- UI-TARS Desktop：https://github.com/bytedance/UI-TARS-desktop
- GLM-V：https://github.com/zai-org/GLM-V

适合以后学习：

```text
Computer Use
GUI Agent
视觉感知
动作生成
屏幕状态反馈
Browser / Desktop Automation
```

当前找通用 Agent / LLM 应用实习时，不应该挤占 Tool / Context / Eval / Production 的时间。

## 7.2 ByteDance Seed

- Seed-1.8：https://github.com/ByteDance-Seed/Seed-1.8

适合在 Agent 算法方向继续研究 perception-reasoning-action、Agentic model 与相关后训练问题。

当前应用工程主线只需知道其技术方向和能力边界。

---

## 八、推荐源码阅读顺序

不要按“厂商名气”排，要按认知复杂度排。

### Stage 0：先有自己的基线

先完成仓库主路线里的：

> 不依赖 Agent 框架，手写一个约 100-300 行的 Minimal Agent。

至少包含：

- model call；
- tool schema；
- tool execution；
- message history；
- max steps；
- error handling；
- trace / log。

没有自己的基线，读框架源码时很难判断“这些额外代码解决了什么问题”。

### Stage 1：Mini-Agent

目标：第一次完整看清一个 Agent pipeline。

```text
自写 Minimal Agent
        ↓
MiniMax Mini-Agent
```

重点比较：

- 多出来了哪些抽象？
- 哪些代码是 production requirement？
- 哪些设计只是项目选择，并非 Agent 必需？

### Stage 2：Qwen-Agent

目标：理解通用 Agent Framework。

```text
Mini-Agent
    ↓
Qwen-Agent
```

重点：Agent abstraction、LLM adapter、Tool、MCP、RAG、Code Interpreter、Browser。

### Stage 3：OpenCode

目标：从通用 Agent 进入完整开源 Coding Agent。

重点追：

```text
Task
→ Agent
→ Context
→ Model
→ Tool
→ Permission
→ File / Shell
→ Observation
→ Compaction
→ Next Step
```

### Stage 4：Kimi Code

目标：比较另一套官方 Coding Agent runtime，并把它和 Kimi K3 的 Agentic Model 能力联系起来。

重点比较 OpenCode：

- session；
- approval；
- tool abstraction；
- context；
- MCP / ACP；
- long-horizon behavior。

### Stage 5：Trae-Agent

目标：从第二套国内 Coding Agent 实现验证哪些抽象是共性的。

不要只看“哪里不一样”，更要总结：

> OpenCode / Kimi Code / Trae-Agent 三者都必须解决哪些问题？

这部分才是可迁移知识。

### Stage 6：DeerFlow

目标：进入 SuperAgent / long-running harness。

重点：Sandbox、Memory、Skills、Subagents、任务拆解与长任务恢复。

### Stage 7：AgentScope

目标：补 Production Framework / Serving / 多 Agent / A2A 等更完整工程抽象。

### Stage 8：模型层

在 Agent runtime 已经比较清楚后，再系统连接模型层：

```text
Kimi K3
DeepSeek-R1 / V3
GLM-5
Seed
```

重点回答：

> 哪些问题必须靠更强的模型能力解决？
>
> 哪些问题应该由 harness / workflow / tools / context engineering 解决？

这是理解 Agent Engineering 非常重要的边界。

---

## 九、不是所有项目都用同一种“读源码”方法

### 9.1 第一次：完整主调用链阅读

适用：Mini-Agent。

目标：建立源码阅读方法。

### 9.2 第二次：框架核心抽象阅读

适用：Qwen-Agent、AgentScope。

目标：看 Agent / Tool / Model / Memory / MCP 等接口怎么拆分。

### 9.3 第三次：运行时行为阅读

适用：OpenCode、Kimi Code、Trae-Agent、Codex / Claude Code 相关 SDK 与机制。

目标：研究 Agent 如何真正“操作一个工作区”。

### 9.4 第四次：大型系统定向阅读

适用：DeerFlow、UI-TARS。

只围绕一个问题追代码，例如：

- sandbox 怎么创建？
- subagent 怎么启动？
- memory 怎么写入？
- context 什么时候 compact？

禁止无目的从第一个文件开始逐行翻。

### 9.5 第五次：模型源码 / 技术报告阅读

适用：Kimi K3、DeepSeek、GLM、Seed。

结合机器学习 / 深度学习主线阅读，不要求当前 Agent 应用阶段把训练与推理全部吃透。

---

## 十、每次源码阅读必须产出的内容

源码阅读不能只有“我看过”。每个重点项目至少形成下面 6 项中的 4 项：

1. **Architecture Map**：模块结构图；
2. **Call Trace**：一次请求从入口到模型 / Tool / 返回的调用链；
3. **Core Abstractions**：Agent、Tool、Context、Session 等核心对象说明；
4. **Design Trade-offs**：至少 3 个设计取舍；
5. **Failure Path**：至少追一个异常 / Tool 失败 / 权限失败路径；
6. **Comparison Note**：和前一个项目比较哪些是共性、哪些是实现选择。

建议最终形成一张横向表：

| 维度 | 自写 Agent | Mini-Agent | Qwen-Agent | OpenCode | Kimi Code | Trae-Agent | DeerFlow |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Agent loop | | | | | | | |
| Tool registry | | | | | | | |
| Context | | | | | | | |
| Session | | | | | | | |
| Memory | | | | | | | |
| Permission | | | | | | | |
| Sandbox | | | | | | | |
| Skills | | | | | | | |
| Subagents | | | | | | | |
| Hooks | | | | | | | |
| MCP | | | | | | | |
| Compaction | | | | | | | |
| Eval / Trace | | | | | | | |
| Recovery | | | | | | | |

这张表最终比“看过多少仓库”更有面试价值。

---

## 十一、什么时候可以停止读一个项目

满足以下条件即可停止，不追求全仓库覆盖：

1. 能画核心架构；
2. 能口头讲清一条完整主调用链；
3. 能指出至少 3 个关键设计取舍；
4. 能找到 Tool / Context / Permission / Error 中至少三个机制的实现位置；
5. 能和前一个项目做有依据的比较；
6. 能把至少一个设计迁移到自己的项目。

如果已经满足这些条件，继续机械翻文件的收益通常很低。

---

## 十二、与实习准备的优先顺序

当前以 Agent / LLM 应用实习为目标时，优先级应是：

```text
Agent 基础 / Tool / MCP
        ↓
自己手写 Minimal Agent
        ↓
LangGraph / Agentic RAG
        ↓
Context / Memory
        ↓
Eval / Observability
        ↓
Production Engineering
        ↓
Mini-Agent + Qwen-Agent 源码
        ↓
OpenCode / Kimi Code / Trae-Agent
        ↓
DeerFlow / AgentScope
        ↓
Kimi K3 / DeepSeek / GLM / Seed 模型层
        ↓
Multi-Agent / GUI Agent 等专项
```

注意：源码阅读不是独立于项目的“额外课程”。最佳方式是每读到一个有价值的设计，就回到自己的主项目里实现一次。

例如：

- 从 Claude Code 学 permission / hooks -> 给自己的高风险 Tool 加 approval；
- 从 OpenCode / Kimi Code 学 compaction -> 给自己的长对话 Agent 加上下文压缩；
- 从 Qwen-Agent 学 Tool abstraction -> 重构自己的工具注册层；
- 从 DeerFlow 学 sandbox -> 给执行代码的 Tool 增加隔离；
- 从 AgentScope 学 serving -> 改进自己的服务化结构。

这样源码阅读才会真正转化成求职项目能力。

---

## 十三、当前阶段最终推荐

### 第一梯队：认真读文档 + 重点源码 / 机制

```text
Mini-Agent
Qwen-Agent
OpenCode
Kimi Code
OpenAI Agents SDK / Codex
Claude Code / Claude Agent SDK
Trae-Agent
DeerFlow
```

### 第二梯队：官方技术报告 + 关键实现

```text
Kimi K3
DeepSeek-R1 / V3
GLM-5
Seed
AgentScope（按项目需要可提前）
```

### 第三梯队：方向明确后再深入

```text
UI-TARS
GLM-V GUI Agent
Multi-Agent / Agent Swarm
更深入的 Agentic RL / AI Infra
```

当前不要把目标设成“把所有厂商都学一遍”。真正应该形成的是：

> **从多个成熟实现中抽象出 Agent Engineering 的共性，再把这些共性落实到自己的主项目。**
