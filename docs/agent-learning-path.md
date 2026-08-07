# Agent 学习路线与资料汇总（面试竞争力导向）

> 目标：不止于“会用框架跑通 demo”，而是在实习面试中能讲清设计取舍、失败模式与评测方法。
> 适用对象：已有 Python 与 RAG 基础，目标为国内 LLM 应用 / Agent 开发方向实习。

---

## 一、什么叫“面试能讲得有竞争力”

先把标准定死，否则学到什么程度没有参照。以下四条全部满足，才算达标：

1. 能说清 workflow 与 agent 的边界，并解释自己项目为什么选了其中一种。
2. 能不依赖任何框架手写最小 ReAct 循环，并说出框架替你做了哪些事。
3. 能举出自己项目的真实失败案例与修复过程（上下文超限、工具误调、循环不终止等）。
4. 能给出量化的评测结论，而不是“效果还不错”。

判定方式：每学完一个模块，尝试在不看笔记的情况下开口讲 3-5 分钟，录音回听。讲不连贯就是没掌握。

---

## 二、学习内容清单

### 模块 A：Agent 核心范式与经典论文

**掌握标准**：能画出 Thought-Action-Observation 循环图，说出每篇论文解决的问题、相比前一篇的改进点与自身局限。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| ReAct：推理与行动协同 | https://arxiv.org/abs/2210.03629 | 论文 |
| Tree of Thoughts：树状搜索式推理 | https://arxiv.org/abs/2305.10601 | 论文 |
| Reflexion：自我反思与记忆写入 | https://arxiv.org/abs/2303.11366 | 论文 |
| AI Agent 系列（综述/基础技术/应用/框架/Benchmark） | https://zhuanlan.zhihu.com/p/695667400 | 中文导读 |

**面试常问**：ReAct 和 Chain-of-Thought 的区别？为什么引入 Observation 能缓解幻觉？ToT 的代价是什么？

### 模块 B：工具调用与协议标准

**掌握标准**：能独立写一个 MCP Server 并被客户端正确调用；能解释工具描述写法如何影响调用准确率。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| MCP 官方规范 | https://modelcontextprotocol.io/specification | 标准/规范 |
| MCP 入门文档 | https://modelcontextprotocol.io/docs/getting-started/intro | 官方文档 |
| OpenAI Function Calling 指南 | https://developers.openai.com/api/docs/guides/function-calling | 官方文档 |
| Writing effective tools for LLM agents | https://www.anthropic.com/engineering | 工程实践 |
| 工具调用三种模式详解（单一/依赖/并行） | https://zhuanlan.zhihu.com/p/2000210358820946463 | 中文导读 |

**面试常问**：模型怎么知道该调哪个工具？工具返回值过长怎么处理？MCP 和直接写 Function Calling 的区别与取舍？

### 模块 C：框架与编排

**掌握标准**：能用 LangGraph 实现带条件分支、状态持久化、人工介入的 Agent；能说出为什么选它而不是其他框架。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| LangGraph 官方文档与 Quickstart | https://docs.langchain.com/oss/python/langgraph/overview | 官方文档 |
| LangGraph Academy（免费课程） | https://academy.langchain.com/courses/intro-to-langgraph | 官方课程 |
| langgraph-101（官方动手 Notebook） | https://github.com/langchain-ai/langgraph-101 | 源码/实践 |
| 框架横向对比（LangGraph/CrewAI/AutoGen/OpenAI SDK） | https://langfuse.com/blog/2025-03-19-ai-agent-comparison | 工程参考 |

框架取舍结论：**主学 LangGraph**（国内外 JD 出现频率最高、工程可控性最好），CrewAI 与 AutoGen 只需理解其多智能体协作模式，不必深入。

**面试常问**：为什么需要图式编排而不是一个 while 循环？状态怎么持久化？怎么做人工审批中断？

### 模块 D：上下文工程与记忆

**掌握标准**：能说出至少三种上下文压缩/清理策略及其适用场景与代价。这是长程任务能否稳定的决定因素，也是大多数学习者的盲区。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| Effective context engineering for AI agents | https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents | 工程实践 |
| Context engineering：memory / compaction / tool clearing | https://platform.claude.com/cookbook | 官方 Cookbook |
| Effective harnesses for long-running agents | https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents | 工程实践 |

**面试常问**：对话变长后上下文爆了怎么办？短期记忆和长期记忆如何分层？压缩会丢失什么？

### 模块 E：评测与可观测性

**掌握标准**：能为自己的 Agent 设计一套离线评测集，并用 trace 定位一次真实失败。**这是区分“玩具项目”与“工程项目”的分水岭。**

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| Demystifying evals for AI agents | https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents | 工程实践 |
| Agent observability：追踪与评测 | https://www.langchain.com/resources/agent-observability | 官方文档 |
| LangSmith 实践（追踪 + pytest 离线评测） | https://www.langchain.com/langsmith/observability | 官方文档 |

**面试常问**：你怎么证明你的 Agent 比 baseline 好？非确定性系统怎么写回归测试？LLM-as-judge 的坐坑风险？

### 模块 F：工程化与安全

**掌握标准**：能说清工具权限边界、人工确认点、失败降级与成本控制的设计。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| Building effective agents（workflow vs agent、五种可组合模式） | https://www.anthropic.com/engineering/building-effective-agents | 工程实践 |
| A practical guide to building agents（约 34 页白皮书） | https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/ | 工程实践 |
| Production best practices | https://developers.openai.com/api/docs/guides/production-best-practices | 官方文档 |

**面试常问**：Agent 要删数据库怎么办？怎么防止无限循环？一次任务成本怎么估算和降低？

### 模块 G：多智能体与前沿（了解层）

**掌握标准**：能说出多智能体的适用场景与引入代价，不需要动手实现。面试时能说“我评估过但没用，因为……”比硬套更加分。

| 内容 | 来源 | 类型 |
| --- | --- | --- |
| Agentic RAG / 多智能体架构趋势 | https://www.anthropic.com/engineering/building-effective-agents | 工程实践 |
| A2A 等智能体间协议（概念了解） | https://modelcontextprotocol.io | 标准/规范 |

---

## 三、学习顺序

不排时间线，只按依赖关系排序。每一步完成后再进入下一步。

1. 读 ReAct 论文，弄懂 Thought-Action-Observation 循环。
2. 不依赖任何框架，手写一个最小 ReAct 循环（一个 while 循环 + 手动解析模型输出）。
3. 读 Anthropic《Building effective agents》与 OpenAI 实践指南，建立“workflow vs agent”的选型判断力。
4. 读 Reflexion 与 Tree of Thoughts，补齐反思与规划两条技术线。
5. 读 OpenAI Function Calling 文档，把第 2 步的手写循环改成规范的工具调用形式。
6. 读 MCP 规范，写一个最简 MCP Server（如封装一个查询工具）+ 客户端调用。
7. 读 Anthropic 工具编写文档，回头优化自己工具的名称与描述，对比调用准确率变化。
8. 过 LangGraph 官方 Quickstart 与 Academy 课程，理解 State / Node / Edge / Memory。
9. 用 LangGraph 重写第 2 步的 ReAct 循环，逐项对照框架替你做了什么。
10. 接入已有 RAG 能力，做成“检索增强型 Agent”：能判断是否需要检索、调用检索工具、多轮修正。
11. 读上下文工程文档，给项目加上上下文压缩、工具结果清理与记忆层。
12. 接入 LangSmith（或其他追踪工具），看完整执行链，定位至少一次真实失败并修复。
13. 读评测文档，为项目建立评测集与评测脚本，产出量化对比结果。
14. 读生产实践文档，补上护栏、重试降级、成本控制与部署。
15. 了解多智能体与 Agentic RAG 架构，能说清适用场景与代价即可。
16. 把全部内容整理成项目文档 + 面试讲解稿，开口录音自测。

关键顺序原则：**先手写再用框架**（第 2 步先于第 8-9 步），**先跑通再评测**（第 10 步先于第 13 步）。跳过手写直接上框架，会变成只会拼装组件、出问题不知道从哪查的人。

---

## 四、项目产出要求

面试讲得流畅的前提是有东西可讲。最终项目应同时满足：

- 完整链路：用户输入 -> 规划 -> 工具调用（含检索）-> 结果整合 -> 输出。
- 有错误恢复：工具失败、输出格式错误、循环不终止都有处理策略。
- 有评测：至少一个小规模评测集 + 量化指标 + 与 baseline 的对比。
- 有追踪：能展示一条完整 trace 并解释每一步。
- 有文档：架构说明 + 设计取舍 + 踩坑记录。

---

## 五、面试自检清单

能不看笔记答出以下全部问题，才算准备好：

1. 什么情况下不该用 Agent，而应该用固定 workflow？
2. 你的 Agent 怎么决定调哪个工具？调错了怎么办？
3. 上下文超限你怎么处理？丢信息了怎么办？
4. 怎么证明你的优化真的有效？
5. 一次完整任务多少 token、多少钱、多长时间？
6. 如果让你重做一次，哪里会改？

第 6 题最关键。答不出来说明你没真正反思过自己的设计。

---

## 六、常见坑

- **只学框架 API，不懂底层循环**：面试官一追问实现细节就卡住。
- **项目只有 happy path**：没有失败处理和评测，会被归类为教程复刻品。
- **堆砌技术名词**：简历写一堆框架名但讲不出取舍，反而扣分。写能力，不写框架名。
- **无条件相信厂商文档**：OpenAI 与 Anthropic 的文档各自有产品立场，需区分通用工程原则与产品推荐。
- **引用过时信息**：模型版本、价格、API 形态变化快，引用前重新核验。

---

## 七、资料优先级说明

本文资料分为四层：**论文 > 标准/规范 > 官方文档 > 工程实践博客**。

需要注意的特殊情况：Agent 方向的**学术论文严重滞后于工程实践**。ReAct 是 2022 年的工作，而上下文工程、评测方法、工具设计这些问题目前主要沉淀在厂商工程博客中，没有对应教材。因此在这个方向上，工程博客的实际参考价值高于通常情况，但仍需保持对其产品立场的警惕。

本文所有链接均为整理时有效，具体页面路径可能调整，访问失败时以官方站点当前导航为准。
