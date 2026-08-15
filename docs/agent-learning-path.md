# Agent 学习路线与资料汇总（面试竞争力导向）

> 目标：不止于“会用框架跑通 demo”，而是在实习面试中能讲清设计取舍、失败模式、工程边界与评测方法。
>
> 适用方向：国内 LLM 应用 / Agent 开发、Agent 平台 / Infra、AI Coding；Agent 算法岗需要在本路线基础上额外加强 PyTorch、后训练与强化学习。
>
> 招聘侧依据：见 [`2026 国内 Agent / LLM 实习岗位调研`](./china-agent-intern-market-2026.md)。Coding Agent 专项见 [`Coding Agent 与 Agent Harness 学习指南`](./coding-agent-and-harness.md)。

---

## 一、什么叫“面试能讲得有竞争力”

以下能力全部满足，才算达到“实习可讲”的程度：

1. 能说清 workflow 与 agent 的边界，并解释自己的项目为什么选其中一种。
2. 能不依赖任何框架手写最小 Agent / ReAct loop，并解释框架替自己做了什么。
3. 能把 Function Calling、MCP、A2A、LangGraph 放在正确层级理解，而不是把它们混成“Agent 框架”。
4. 能举出项目的真实失败案例与修复过程，例如上下文失控、工具误调、循环不终止、检索失败、权限错误。
5. 能给出量化评测，而不是“效果还不错”。
6. 能把 Agent 做成服务：有 API、日志、测试、重试 / 超时、权限、成本与部署意识。
7. 能解释 Claude Code、Codex、OpenCode 这类 Coding Agent 比普通 ReAct Demo 多了哪些 runtime / harness 能力。

判定方式：每学完一个模块，不看笔记口头讲 3-5 分钟；涉及工程模块时必须配合代码、trace、测试或实验结果，而不是只做概念笔记。

---

## 二、前置能力：不在本仓库重复完整课程

本仓库聚焦 Agent。下面这些属于前置或并行主线，只定义需要达到的最低边界。

### 0.1 Python 与计算机基础

至少掌握：

- Python 基础与项目结构；
- typing、Pydantic、异常处理、日志、配置；
- `async` / `await`、HTTP / REST 基础；
- Git、Linux；
- 数据结构与算法；
- 操作系统、网络、数据库的基础概念。

偏 Agent Infra / 平台岗位时，需要继续加强并发、缓存、消息队列、容器、服务治理等后端能力。

### 0.2 LLM 基础

至少能够解释：

- Transformer / Attention；
- token / tokenizer；
- embedding；
- context window；
- KV Cache；
- temperature / top-p 等生成参数；
- pretraining、SFT、LoRA、DPO、RLHF / GRPO 分别解决什么问题；
- 模型幻觉、非确定性、长上下文退化等基本能力边界。

应用岗不要求先完整训练大模型；Agent 算法岗则需要同步深入 PyTorch、Transformers、SFT / RL 等。

### 0.3 RAG 基础

至少理解：

`文档处理 -> chunk -> embedding -> retrieval -> rerank -> context assembly -> generation -> evaluation`

不能把“接一个向量数据库”当成 RAG 的全部。

---

## 三、Agent 学习内容清单

### 模块 A：Agent 核心范式与经典工作

**掌握标准**：能画出 Thought-Action-Observation 循环，理解 planning、reflection、tool use 分别解决什么问题，并能说明它们的代价。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| ReAct：推理与行动协同 | https://arxiv.org/abs/2210.03629 | 论文 |
| Tree of Thoughts：树状搜索式推理 | https://arxiv.org/abs/2305.10601 | 论文 |
| Reflexion：自我反思与记忆写入 | https://arxiv.org/abs/2303.11366 | 论文 |
| Building effective agents | https://www.anthropic.com/engineering/building-effective-agents | 工程实践 |

重点问题：

- Workflow 和 Agent 的边界是什么？
- ReAct 相比单纯 Chain-of-Thought 多了什么？
- Observation 为什么重要？
- Reflection 什么时候有用，什么时候只是增加 token？
- Planning 为什么可能让系统更慢、更不稳定？

### 模块 B：Tool Use、Function Calling 与协议

**掌握标准**：能独立实现规范的工具调用，能写 MCP Server / Client，并能清楚区分 Function Calling、MCP 与 A2A。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| OpenAI Function Calling | https://developers.openai.com/api/docs/guides/function-calling | 官方文档 |
| MCP Specification | https://modelcontextprotocol.io/specification | 标准 / 规范 |
| MCP Getting Started | https://modelcontextprotocol.io/docs/getting-started/intro | 官方文档 |
| A2A Protocol | https://a2a-protocol.org/latest/ | 标准 / 规范 |
| Anthropic 工具设计相关工程文章 | https://www.anthropic.com/engineering | 工程实践 |

必须形成下面这张概念图：

```text
Function Calling
模型 <-> 应用内函数 / 工具

MCP
Agent / Client <-> 外部 Tools / Resources / Prompts

A2A
Agent <-> Agent
```

重点问题：

- 模型怎么决定调哪个工具？
- Tool schema 为什么会影响调用准确率？
- 工具返回过长怎么办？
- 并行调用和依赖调用怎么组织？
- MCP 解决了什么标准化问题？
- 为什么有 MCP 之后仍然需要 A2A？

### 模块 C：框架与状态编排——主学 LangGraph

**掌握标准**：能用 LangGraph 实现条件分支、显式 State、checkpoint、人工中断与恢复，并能把它和自己手写的 Agent loop 对照起来。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| LangGraph Overview | https://docs.langchain.com/oss/python/langgraph/overview | 官方文档 |
| LangGraph Academy | https://academy.langchain.com/courses/intro-to-langgraph | 官方课程 |
| langgraph-101 | https://github.com/langchain-ai/langgraph-101 | 源码 / 实践 |

**为什么主学 LangGraph：**它能比较清楚地暴露 State、Node、Edge、Checkpoint、Interrupt、Durable Execution 等 Agent 编排概念，并且在当前 Agent 岗位中具有较高认可度。目标不是押注某个框架长期占优，而是借它掌握可迁移的 orchestration 思维。

CrewAI、AutoGen、AgentScope、OpenAI Agents SDK 等不需要全部深学；能读懂它们的核心抽象、快速上手并解释取舍即可。

重点问题：

- 为什么不是永远用一个 `while` 循环？
- 显式 State 有什么价值？
- checkpoint 与 memory 有什么区别？
- Human-in-the-loop 怎么插入运行图？
- 什么逻辑应该写成确定性 edge，而不应该交给 LLM 判断？

### 模块 D：Agentic RAG

**掌握标准**：让 Agent 能判断何时需要检索、如何改写查询、如何处理低质量检索，并用评测证明 Agentic RAG 是否真的优于固定 RAG workflow。

重点实践：

- retrieval tool；
- query rewrite；
- hybrid retrieval；
- rerank；
- multi-hop retrieval；
- retrieval failure handling；
- citation / evidence；
- retrieval eval。

重点问题：

- “让 Agent 自己决定是否检索”一定更好吗？
- 多轮检索什么时候有收益？
- 如何防止 Agent 在错误证据上继续推理？

### 模块 E：Context Engineering 与 Memory

**掌握标准**：能设计 working context，而不是把所有历史消息永久追加进 prompt；能说清短期状态、长期记忆和外部持久化的边界。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| Effective context engineering for AI agents | https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents | 工程实践 |
| Claude Cookbook / Context Engineering | https://platform.claude.com/cookbook | 官方 Cookbook |
| Effective harnesses for long-running agents | https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents | 工程实践 |

至少掌握：

- context pruning；
- summarization / compaction；
- tool result clearing；
- working memory；
- long-term memory；
- memory retrieval；
- memory update / expiration；
- memory pollution；
- persistent instructions 与普通对话历史的差异。

重点问题：

- 长上下文为什么不是越长越好？
- 压缩会损失什么？
- 什么信息应该进长期记忆？
- Memory 和数据库到底是什么关系？

### 模块 F：Evaluation、Observability 与 Bad Case

**掌握标准**：能为自己的 Agent 建立小型离线评测集，用 trace 定位真实失败，并做前后量化对比。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| Demystifying evals for AI agents | https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents | 工程实践 |
| LangSmith Observability | https://www.langchain.com/langsmith/observability | 官方文档 |
| OpenAI Agents SDK Tracing | https://openai.github.io/openai-agents-python/tracing/ | 官方文档 |

至少覆盖：

- task success rate；
- tool selection accuracy；
- retrieval quality；
- format validity；
- latency；
- token / cost；
- failure taxonomy；
- regression test；
- LLM-as-Judge。

重点问题：

- 非确定性系统如何做回归测试？
- LLM-as-Judge 有哪些偏差？
- Agent 成功率提升了，但成本翻倍，算不算优化？

### 模块 G：Production Agent Engineering 与安全

**掌握标准**：能把 Agent 从 Notebook / CLI Demo 做成可运行服务，并理解权限、故障与生产约束。

建议掌握：

```text
FastAPI / REST
Pydantic
asyncio
HTTP client
SSE / streaming
PostgreSQL / Redis
Docker
pytest / mock
logging / tracing
timeout / retry / fallback
rate limit / cache
secret management
permission / approval
Human-in-the-loop
prompt injection 防护
cost / latency
部署基础
```

偏平台 / Infra 岗再补：gRPC、消息队列、Kubernetes / Serverless、Sandbox、服务治理、多租户与 Agent lifecycle。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| OpenAI Practical Guide to Building Agents | https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/ | 工程实践 |
| OpenAI Production Best Practices | https://developers.openai.com/api/docs/guides/production-best-practices | 官方文档 |
| Building effective agents | https://www.anthropic.com/engineering/building-effective-agents | 工程实践 |

重点问题：

- Agent 要删除数据库数据时怎么办？
- timeout、retry、fallback 各解决什么故障？
- 如何限制循环次数和预算？
- 如何设计高风险 Tool 的 approval？
- prompt injection 为什么在能执行工具的 Agent 中更加危险？

### 模块 H：Coding Agent 与 Agent Harness

**掌握标准**：能从成熟 Coding Agent 反推 agent runtime / harness 的设计，理解为什么生产级 Agent 不只是“LLM + tools”。

完整资料：[`Coding Agent 与 Agent Harness 学习指南`](./coding-agent-and-harness.md)

重点系统：

| 系统 | 学习重点 | 官方入口 |
| --- | --- | --- |
| OpenAI Agents SDK | Agent loop、tools、handoffs、guardrails、sessions、tracing、MCP | https://openai.github.io/openai-agents-python/ |
| OpenAI Codex | Coding Agent、repo exploration、sandbox / approval、Skills、MCP、长任务 | https://developers.openai.com/codex/ |
| Claude Code / Agent SDK | CLAUDE.md、Skills、Subagents、Hooks、Permissions、MCP、Agent runtime | https://code.claude.com/docs/en/overview |
| OpenCode | 开源 Coding Agent、primary / subagents、permissions、AGENTS.md、compaction、源码调用链 | https://opencode.ai/docs |

重点理解：

- Agent loop；
- workspace / filesystem / shell；
- sandbox；
- permissions / approval；
- persistent instructions；
- Skills；
- subagents；
- hooks；
- context compaction；
- MCP；
- long-running task；
- test-before-finish；
- failure recovery。

目标不是“精通三个 CLI”，而是理解通用 Agent Harness。

### 模块 I：Multi-Agent 与 A2A（后置）

**掌握标准**：能说清什么时候需要多个 Agent、如何通信以及引入的成本；没有必要为了简历强行把主项目改成 Multi-Agent。

学习内容：

- supervisor / worker；
- agents-as-tools；
- handoff；
- shared state vs isolated context；
- parallelism；
- conflict / aggregation；
- A2A；
- multi-agent evaluation。

优先级低于：单 Agent 稳定性、Tool use、Context、Eval、Production Engineering。

### 模块 J：Agentic LLM / 后训练（算法岗分支）

如果目标是 Agent 应用开发，这一模块只需要理解概念；如果投 Agent 算法 / Agentic LLM 岗，则需要系统学习：

- PyTorch / Transformers；
- SFT / LoRA；
- preference optimization；
- PPO / GRPO 等 RL；
- Tool-use / Tool RL；
- Agentic RL；
- long-context training；
- inference / vLLM；
- benchmark 与论文复现。

这部分应和机器学习 / 深度学习主线并行，而不是在本仓库重复完整课程。

---

## 四、推荐学习顺序

不强行规定周数，只按依赖关系推进。

1. 读 ReAct，理解 Thought-Action-Observation。
2. 不依赖框架，手写最小 Agent loop。
3. 读 Anthropic《Building effective agents》和 OpenAI Agent 工程指南，建立 workflow vs agent 判断。
4. 读 Reflexion / Tree of Thoughts，理解 reflection / planning 的收益和代价。
5. 学 Structured Output / Function Calling，把手写 loop 改为规范 Tool Calling。
6. 学 MCP，自己写一个最小 MCP Server + Client。
7. 优化 Tool schema / description，并用小评测集比较调用准确率。
8. 学 LangGraph 的 State / Node / Edge / Checkpoint / Interrupt。
9. 用 LangGraph 重写自己的最小 Agent，对照框架隐藏了哪些工作。
10. 做 Agentic RAG：检索 Tool、query rewrite、rerank、失败恢复。
11. 学 Context Engineering / Memory，加入 pruning、compaction、memory layer。
12. 接入 tracing，记录至少三类真实 Bad Case。
13. 建 eval set 和评测脚本，形成 baseline 与优化后的量化对比。
14. 把项目服务化：FastAPI、async、日志、测试、数据库 / Redis（按需）。
15. 加 timeout、retry、fallback、权限、Human-in-the-loop 与成本控制。
16. Docker 化并跑通部署链路。
17. 用 Codex、Claude Code、OpenCode 完成同一代码任务，对比 agent loop / tool / permission / context 行为。
18. 系统学习 Skills、Subagents、Hooks、Sandbox / Approval、Persistent Instructions。
19. 阅读 OpenAI Agents SDK / Claude Agent SDK；再从 OpenCode 源码追一次完整 Agent 调用链。
20. 了解 A2A 与 Multi-Agent；只有项目确实需要时再实现。
21. 如果投 Agent 算法岗，再深入 PyTorch、SFT、RL / GRPO、Tool RL。
22. 整理主项目 README、架构图、eval、trace、Bad Case 和面试讲解稿。

三个顺序原则：

- **先手写，再用框架。**否则容易只会拼组件。
- **先跑通，再评测，再优化。**没有 baseline 就无法证明优化有效。
- **先把单 Agent 做稳定，再考虑 Multi-Agent。**复杂度不是竞争力本身。

---

## 五、项目产出要求

不建议做一堆“天气 Agent / 新闻 Agent / PDF Agent”。更有效的是：**两个小实验 + 一个主项目**。

### 小实验 1：Minimal Agent

不依赖 Agent 框架，包含：

- Agent loop；
- 2-3 个 tools；
- structured output；
- max steps；
- tool error；
- trace / log。

### 小实验 2：MCP

包含：

- 一个 MCP Server；
- 至少 2 个 tools / resources；
- 客户端调用；
- 权限 / 错误实验；
- 和直接 Function Calling 的对比说明。

### 主项目：Production-grade Agent

至少包含：

- 明确的真实任务；
- workflow / agent 选型依据；
- 多 Tool；
- RAG；
- Context / Memory；
- Tool permissions；
- Human-in-the-loop；
- loop / timeout / retry / fallback；
- Eval set + 指标；
- baseline 对比；
- Trace / Observability；
- FastAPI 服务；
- pytest；
- Docker；
- README 架构图；
- Bad Case 与修复记录；
- token / latency / cost 数据。

可选加分项：

- MCP；
- Skills；
- Sandbox；
- Coding Agent；
- Open-source PR；
- 一个真正有理由存在的 Multi-Agent 模块。

---

## 六、面试自检清单

能不看笔记结合自己项目答出这些问题，才算准备好：

1. 什么情况下不该用 Agent，而应该用固定 workflow？
2. Agent loop 的最小实现是什么？
3. 模型怎么决定调哪个 Tool？Tool 调错了怎么办？
4. Function Calling、MCP、A2A 分别解决什么问题？
5. 为什么需要显式 State / Checkpoint？
6. 上下文越来越长怎么办？压缩会丢什么？
7. Working Memory、Long-term Memory、数据库有什么区别？
8. 怎么证明 Agent 比 baseline 好？
9. 非确定性系统怎么做 regression test？
10. LLM-as-Judge 有哪些风险？
11. Agent 执行 Shell / 删除数据时怎么设计权限？
12. permission、approval、sandbox 有什么区别？
13. 为什么 Skills 不等于 Tools？
14. Hooks 为什么不能全部被 Prompt 替代？
15. Claude Code / Codex / OpenCode 比普通 ReAct Demo 多了哪些 Harness 能力？
16. Multi-Agent 什么时候值得用？
17. 一次完整任务多少 token、多少钱、多长时间？
18. 你的三个最典型 Bad Case 是什么？
19. 哪个优化真正提高了成功率？提高多少？代价是什么？
20. 如果重新做一次，你会把哪些 Agent 化设计删掉，改成 deterministic code？

最后一题尤其重要：能主动删掉不必要的 Agent 复杂度，通常比堆更多框架更能体现工程判断力。

---

## 七、常见坑

- **只学框架 API，不懂 Agent loop**：出问题时不知道错误发生在哪一层。
- **RAG 只会向量库 Demo**：没有 rerank、eval、Bad Case 和数据质量意识。
- **项目只有 happy path**：没有失败处理、权限和测试。
- **没有量化评测**：无法证明“优化”是否真的有效。
- **把 Memory 当成无限聊天记录**：上下文迅速恶化。
- **强行 Multi-Agent**：增加 token、延迟和协调失败，却没有实际收益。
- **把 AI Coding 工具使用经验当成 Agent 原理**：会用 Claude Code 不等于理解 Agent Harness。
- **堆框架名**：LangGraph、CrewAI、AutoGen 全写在简历上但讲不清设计取舍，价值很低。
- **忽视传统软件工程**：真实 Agent 仍然要面对 API、数据库、并发、测试、部署、权限和故障。
- **引用过时产品信息**：模型版本、API、Coding Agent 功能、价格变化很快，面试前重新核验。

---

## 八、资料与信息源原则

优先顺序：

1. 论文 / 标准规范；
2. 官方 SDK / 产品文档；
3. 官方工程文章与 Cookbook；
4. 高质量开源源码；
5. 第三方工程文章；
6. 中文导读只用于辅助理解，不作为唯一依据。

Agent 方向的工程实践变化快。ReAct 等经典论文仍然用于理解思想来源，但 Tool design、Context Engineering、Eval、Harness、Skills、Coding Agent 等能力目前大量沉淀在 SDK 文档、厂商工程文章和开源实现中，因此学习时必须兼顾论文与工程资料。

具体链接可能调整；访问失败时优先从对应官方站点当前文档导航重新定位，不依赖旧博客转载。
