# 2026 国内 Agent / LLM 实习岗位调研

> 调研日期：2026-08-15
>
> 目的：不是统计“市场上所有岗位的精确占比”，而是用近期公开 JD 反推学习优先级、项目要求和面试能力边界。
>
> 范围：以国内 Agent 应用开发、Agent 算法、Agent Infra / 平台、AI Coding / Coding Agent 等实习岗位为主，优先采用企业官方招聘页；官方页不可得时使用牛客、实习僧等公开 JD。

---

## 一、结论先行

2026 年国内 Agent 实习已经明显从“会调用大模型 API + 会 LangChain”向完整的 **Agent Engineering** 迁移。

当前最稳定、跨岗位复用度最高的能力可以归纳为以下七层：

1. **计算机与编程基础**：Python、数据结构与算法、Linux、Git；偏平台岗位继续要求操作系统、网络、数据库、并发与后端基础。
2. **LLM 基础**：Transformer、推理过程、模型能力边界、常用 API、Prompt / Structured Output；算法岗进一步要求 PyTorch、SFT、DPO / GRPO / RL 等。
3. **Agent 核心机制**：ReAct / Plan-Act / CodeAct、任务拆解、工具调用、循环终止、错误恢复、workflow 与 autonomous agent 的取舍。
4. **RAG + Context / Memory**：检索、rerank、上下文压缩、长短期记忆、跨会话状态、长上下文处理。
5. **协议与编排**：Function Calling、MCP、A2A、LangGraph 等；重点是理解协议边界和运行机制，而不是背框架 API。
6. **评测与生产工程**：Bad Case、自动评测、LLM-as-Judge、Trace / Observability、Guardrails、Human-in-the-loop、权限、重试、超时、成本、延迟、部署。
7. **新趋势：Agent Harness / Skills / AI Coding**：Claude Code、Codex、OpenCode 等 Coding Agent 的 agent loop、workspace、tool permissions、skills、subagents、hooks、sandbox、context compaction 等已经进入招聘要求。

因此，找实习时最有竞争力的形态不是“做过很多聊天机器人 Demo”，而是：

> **一个能真实执行任务、可观测、可评测、可恢复、可部署的 Agent 项目 + 对底层 agent loop / tool use / context / permissions / harness 的理解。**

---

## 二、代表性岗位样本

下面选取 15 个近期公开岗位作为学习路线依据。样本用于观察能力结构，不应理解为严格市场份额统计。

| 公司 / 岗位 | 类型 | JD 中值得关注的要求 |
| --- | --- | --- |
| 美图 AI Agent 研发实习生 | Agent 平台 / 应用工程 | LangGraph / ADK / AgentScope、Claude Code、RAG、Tool Calling、Harness Engineering、Skills、Memory、自动评测、Observability、开源 PR |
| 百度 Agent 策略算法实习生 | Agent 算法 / 后训练 | Transformer、PyTorch、Prompt、SFT、DPO / GRPO、Agent 框架、Long-context RAG、Multi-Agent |
| 百度 智能体应用开发实习生 | Agent 应用 / AI Coding | OS / 网络 / 数据结构 / 算法、Python / Java / Go、RAG、LangChain / LlamaIndex、CodeAgent、代码分析、自动测试 / 部署 |
| 上海人工智能实验室 多模态大模型算法（Agent 知识库） | Agent 全栈 / 平台 | ReAct / PlanAct / CodeAct、MCP / A2A / FunctionCall、LangChain / LangGraph、PyTorch、FastAPI / Django、REST、数据库、评测系统 |
| 上海人工智能实验室 智能体研发 | Agent 算法 | Agentic LLM、工具强化、沙盒、强化学习、Python / PyTorch、LangGraph / OpenAI Agents SDK / AutoGen |
| 上海人工智能实验室 大模型强化学习算法实习生 | LLM / Agent 算法 | PyTorch、Python、LLM 基础、强化学习训练系统、模型训练与推理能力研究 |
| 携程 AI Agent 算法实习生（风控） | Agent 应用 / 决策系统 | LangChain / LlamaIndex、Prompt、RAG、Function Calling、Memory、Human-in-the-loop、Bad Case、真实业务 SOP |
| 携程 Agent 开发工程师实习 | Agent 平台 | Agent 应用上线、基础开发框架、Memory、评测、观测、Agent 注册 / 管理 / 发现 / 部署 |
| 传音 AI Agent 研发实习生 | Agent 应用工程 | Python asyncio、后端框架、LLM 原理、LangChain / LlamaIndex / CrewAI、ReAct / Reflexion、RAG、评测 |
| 阿里云 Agent Infra 工程师实习 | Agent Infra / AgentOps | OS / 网络 / DB、Go / Python / Java / Rust、微服务、MQ、缓存、K8s / Serverless / Docker、RAG / Tools / Skills、AgentOps、Observability、AI Coding、CI |
| 阿里 AI Agent 优化工程师实习 | Agent 工程 / AI Coding | Claude Code / Cursor 重度使用、LLM 边界、RAG、工具调用、MCP / Skill Agent、自动评测、观测、异步 / 降级、安全、vLLM / Ollama |
| 阿里 AI Agent 研发工程师实习 | Agent 算法 / 应用 | RAG、LangGraph、Agent Eval、Skill、模型微调 / 蒸馏 / 后训练、Tool RL / RLHF |
| 新石器 Agent 开发工程师实习 | Agent 应用工程 | 计算机基础、Python / Git / Linux、Memory、规划、Reflection、Tool 编排、RAG、权限、LangChain / LlamaIndex / Dify |
| 洞墟 AI Agent 开发工程师实习 | Multi-Agent / 工程 | LangGraph / LangChain、Skill、MCP、Memory、Reflector、REST / gRPC、LLM-as-Judge、Context Engineering、高并发 / 高可用 |
| 广发证券 Agent 工程师实习 | Agent 产品 / 前端工程 | OpenAI Agents SDK、LangGraph、MCP、Browser Agent、Context / Memory、Tool Calling、Human-in-the-loop、安全、SSE / WebSocket、AI Coding |

### 主要来源

- 美图官方招聘：<https://hr.meitu.com/jobCampus/5996c55a-6237-4732-bee0-89b8ce53bbad>
- 百度 Agent 策略算法：<https://talent.baidu.com/jobs/detail/INTERN/6f85641f-2a8e-4806-bbb5-4bbbf4705741>
- 百度智能体应用开发：<https://talent.baidu.com/jobs/detail/INTERN/e3cec5b8-b7a3-4946-99fc-b292b749cd53>
- 上海 AI Lab Agent 知识库：<https://www.shlab.org.cn/joinus/detail/7630823487225088319?mode=campus>
- 上海 AI Lab 智能体研发：<https://www.shlab.org.cn/joinus/detail/7630829116464662830?mode=campus>
- 上海 AI Lab 强化学习：<https://www.shlab.org.cn/joinus/detail/7474835615747705126?mode=campus>
- 携程风控 Agent：<https://www.shixiseng.com/intern/inn_oveszt5vsfer>
- 携程 Agent 开发：<https://www.nowcoder.com/jobs/detail/434803>
- 传音 AI Agent：<https://www.nowcoder.com/jobs/detail/444729>
- 阿里云 Agent Infra：<https://www.nowcoder.com/jobs/detail/440662>
- 阿里 Agent 优化：<https://www.nowcoder.com/jobs/detail/440836>
- 阿里 Agent 研发：<https://www.nowcoder.com/jobs/detail/439114>
- 新石器 Agent 开发：<https://www.nowcoder.com/jobs/detail/451952>
- 洞墟 AI Agent 开发：<https://www.nowcoder.com/jobs/detail/456373>
- 广发证券 Agent 工程师：<https://www.nowcoder.com/jobs/detail/453705>

---

## 三、从样本反推的能力优先级

下面的“出现频率”是对上述样本的人工归类，只用于排学习优先级，不是招聘市场统计结论。

### S 级：必须形成可独立展示的能力

#### 1. Python + 软件工程基本功

几乎所有岗位都要求至少一门主流语言，Python 是 Agent / LLM 岗最稳定的主语言。

应达到：

- 能写结构化 Python 项目，而不只是 Notebook；
- 熟悉 typing、Pydantic、异常处理、日志、配置、测试；
- 理解 asyncio / 并发、HTTP、REST；
- 会 Git、Linux、环境管理；
- 能读第三方库源码并定位问题。

#### 2. RAG + Tool Calling + Agent Loop

这些已经是应用型 Agent 岗的基础能力，而不是加分项。

应达到：

- 自己实现最小 agent loop；
- 理解工具 schema、工具选择、串行 / 并行调用；
- Tool 失败、格式错误、权限错误、超时后可恢复；
- RAG 不停留在“向量库问答”，理解 chunk、embedding、hybrid retrieval、rerank、query rewrite、引用与评测。

#### 3. Context Engineering / Memory

Memory、long context、context compaction 在多个 2026 JD 中已经成为显式要求。

应达到：

- 区分 conversation history、working memory、long-term memory、external state；
- 能解释为什么“把所有历史都塞进 prompt”不可行；
- 会 context pruning / compaction / summarization / tool-result clearing；
- 理解记忆写入、检索、更新、过期与污染问题。

#### 4. Evaluation + Observability

这是现在最能区分“教程 Demo”和“工程项目”的能力之一。

应达到：

- 有固定 eval set；
- 能定义 task success / tool accuracy / retrieval quality / cost / latency 等指标；
- 会分析 trace 和 Bad Case；
- 会做回归测试；
- 理解 LLM-as-Judge 的偏差和使用边界。

### A 级：实习竞争力的明显分水岭

#### 5. MCP / A2A / LangGraph

建议学习目标不是“背框架”，而是：

- Function Calling：模型与函数 / 工具之间的调用接口；
- MCP：Agent / 应用与 Tools / Resources 的标准化连接；
- A2A：独立 Agent 之间的发现、任务和通信；
- LangGraph：状态、节点、边、checkpoint、interrupt、durable execution 等编排概念。

A2A 官方文档：<https://a2a-protocol.org/latest/>

#### 6. Production Agent Engineering

越来越多岗位明确要求生产化能力：

- FastAPI / Django 等后端；
- REST / gRPC / SSE / WebSocket；
- Redis / PostgreSQL / Vector DB；
- Docker；
- timeout / retry / fallback / rate limit；
- 权限与 secrets；
- 高并发、状态管理、可观测性；
- CI / 测试 / 部署。

#### 7. Agent Harness / Skills / AI Coding

这是 2026 年相比早期 Agent JD 最明显的增量之一。

学习时应把 Claude Code、OpenAI Codex、OpenCode 看成“可运行的 Agent 系统样本”，而不只是 AI 编码工具。

重点观察：

- agent loop 如何组织；
- Agent 如何探索代码库；
- 如何管理 workspace / shell / filesystem；
- permissions / approval / sandbox；
- AGENTS.md / CLAUDE.md 等 persistent instructions；
- skills；
- subagents / agent teams；
- hooks；
- context compaction；
- MCP；
- 长任务的 checkpoint / recovery；
- Agent 如何从“回答问题”变成“完成软件工程任务”。

具体学习资料见 [`coding-agent-and-harness.md`](./coding-agent-and-harness.md)。

### B 级：根据投递方向补强

#### 8. PyTorch / Transformer / SFT / RL

如果主要投 **LLM 应用 / Agent 工程**，要求是“理解并能解释”，不用把训练放在 Agent 主线前面。

如果投 **Agent 算法 / Agentic LLM / 策略算法**，则需要明显提升：

- PyTorch / Transformers；
- Transformer / Attention / KV Cache；
- SFT / LoRA；
- DPO / PPO / GRPO；
- Tool RL / Agentic RL；
- 训练数据与 eval；
- 推理优化 / vLLM 等。

这部分应和机器学习、深度学习主线并行学习，而不是在 Agent 仓库重复完整课程。

#### 9. Multi-Agent

多 Agent 在岗位中存在，但不是所有岗位的必要条件。

优先级应低于：单 Agent 稳定性、Tool use、Context、Eval、工程化。

面试最重要的问题不是“你有没有用 Multi-Agent”，而是：

> 为什么这个问题需要多个 Agent？为什么单 Agent + deterministic workflow 不够？额外的协调成本值不值得？

---

## 四、四类岗位的能力分叉

### 方向 1：Agent 应用开发 / LLM 应用工程

优先级：

`Python → LLM API → RAG → Tool Calling → ReAct → LangGraph → MCP → Context / Memory → Eval → FastAPI / Docker → Production`

这是最适合本科阶段用项目证明能力的路线。

### 方向 2：Agent 平台 / Infra / AgentOps

在方向 1 基础上进一步强化：

`OS / 网络 / DB → 并发 → 后端 → Redis / MQ → Docker / K8s → Sandbox → Trace → 多租户 / 权限 → Agent Lifecycle / AgentOps`

这类岗位对传统软件工程基本功要求明显更高。

### 方向 3：Agent 算法 / Agentic LLM

在 Agent 基础上强化：

`Transformer → PyTorch → SFT → Preference Optimization → RL → Tool-use Training → Agentic RL → Benchmark / Research Reproduction`

这类岗位通常更偏硕博，但本科生如果训练 / 论文 / 开源能力强也可以尝试。

### 方向 4：AI Coding / Coding Agent

重点强化：

`Agent Loop → Repo Exploration → Shell / Filesystem Tools → Code Search → Planning → Patch → Test → Review → Sandbox / Permission → Skills → Subagents → Hooks → Long-running Harness`

Claude Code、Codex、OpenCode 是很好的对照样本。

---

## 五、对本仓库学习路线的调整结论

当前仓库已有的 ReAct、MCP、LangGraph、Context Engineering、Evaluation、Observability、Safety 主线应保留。

需要补充的内容：

1. 增加“前置能力”说明：Python 工程、LLM / Transformer、RAG、计算机基础；
2. 增加 Production Agent Engineering：FastAPI、async、数据库、Docker、测试、日志、重试、限流、部署；
3. 增加 Coding Agent / Agent Harness 模块；
4. 系统学习 OpenAI Agents SDK + Codex、Claude Code、OpenCode；
5. 修正 A2A 资料入口，明确 MCP 与 A2A 的边界；
6. LangGraph 继续作为主学编排框架，但理由改为“能清晰暴露状态与 durable orchestration 概念且岗位认可度较高”，不再使用无法严格验证的“JD 出现频率最高”表述；
7. Multi-Agent 保持了解 / 后置，不抢占单 Agent、评测和工程化的学习时间。

---

## 六、最终项目应该达到什么程度

建议最终只保留：

- 一个手写最小 Agent 实验；
- 一个 MCP 实验；
- 一个真正可作为简历主项目的 Production-grade Agent。

主项目至少包含：

- 明确业务任务；
- Planner / workflow / Agent 选型依据；
- 多 Tool；
- RAG；
- Context / Memory；
- Tool permissions / Human-in-the-loop；
- 失败恢复；
- Eval set + 指标；
- Trace / Observability；
- FastAPI 服务；
- pytest；
- Docker；
- README 架构图；
- Bad Case 与优化记录；
- token / latency / cost 数据。

如果项目还能体现 coding agent / sandbox / skills / MCP 中的一个方向，会更贴近 2026 年 Agent 岗位趋势。

---

## 七、面试准备的实际判断标准

最终不要用“我学过 LangGraph / MCP / Claude Code”作为掌握标准，而使用下面的问题：

1. 不用框架，你能不能写出 Agent loop？
2. 什么时候固定 workflow 比 Agent 更好？
3. 工具为什么会调错？如何量化 tool selection accuracy？
4. 长上下文为什么会恶化？Memory 怎么分层？
5. MCP 和 Function Calling 的边界是什么？MCP 和 A2A 又有什么区别？
6. Agent 如何防止无限循环和破坏性操作？
7. 为什么需要 sandbox / approval / permissions？
8. Claude Code / Codex / OpenCode 这类 Coding Agent 比普通 ReAct Demo 多了哪些 runtime / harness 能力？
9. 你的项目最典型的三个 Bad Case 是什么？如何修复？
10. 你的优化究竟提升了多少成功率、延迟或成本？
11. 如果让你重构当前项目，你会删掉哪些 Agent 化设计，改成 deterministic code？

能对这些问题结合自己的代码、trace 和数据回答，才算达到“实习可讲”的程度。
