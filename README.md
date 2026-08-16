# agent-llm-learning

Agent 与 LLM 应用学习笔记、实验与求职准备仓库。

当前目标：围绕国内 LLM 应用 / Agent 开发实习，建立从 Agent 原理、Tool Use、MCP、LangGraph、RAG、Context Engineering、Eval 到 Production Engineering、Coding Agent / Agent Harness 的完整学习主线。

## 学习入口

1. **主学习路线**：[`docs/agent-learning-path.md`](./docs/agent-learning-path.md)
   - Agent 核心范式
   - Function Calling / MCP / A2A
   - LangGraph
   - Agentic RAG
   - Context / Memory
   - Eval / Observability
   - Production Agent Engineering
   - Coding Agent / Agent Harness
   - Multi-Agent 与算法岗分支

2. **模块 A 复习笔记：Agent 核心范式与经典工作**：[`docs/module-a-agent-core-review.md`](./docs/module-a-agent-core-review.md)
   - 普通 LLM / Augmented LLM / Workflow / Agent 的边界
   - Agent Loop、Tool Use、Observation 与环境反馈
   - ReAct 与 Chain-of-Thought 的区别
   - Planning / Replanning 与 Tree of Thoughts
   - Reflection / Reflexion、Evaluator 与失败反馈
   - Minimal Agent 代码骨架、执行轨迹与自检题

3. **2026 国内岗位调研**：[`docs/china-agent-intern-market-2026.md`](./docs/china-agent-intern-market-2026.md)
   - 代表性 Agent / LLM 实习 JD
   - 共性能力与岗位分叉
   - 学习优先级
   - 项目与面试要求

4. **Coding Agent 专项**：[`docs/coding-agent-and-harness.md`](./docs/coding-agent-and-harness.md)
   - OpenAI Agents SDK / Codex
   - Claude Code / Claude Agent SDK
   - OpenCode
   - Skills / Subagents / Hooks
   - Permissions / Sandbox
   - Context Compaction
   - Agent Harness 与源码分析

5. **厂商官方文档与源码阅读计划**：[`docs/vendor-docs-and-source-reading-plan.md`](./docs/vendor-docs-and-source-reading-plan.md)
   - OpenAI、Anthropic、Kimi、Qwen、ByteDance、MiniMax、GLM、DeepSeek、AgentScope
   - 按 Agent Runtime / Harness、通用 Agent Framework、Agentic Model 三类拆分
   - S / A+ / A / B 学习优先级及理由
   - Mini-Agent → Qwen-Agent → OpenCode → Kimi Code → Trae-Agent → DeerFlow 的推荐源码阅读顺序
   - Kimi K3、DeepSeek-R1 / V3、GLM-5、Seed 的模型层阅读定位
   - 每次源码阅读的固定产出、停止条件与横向比较方法

## 学习原则

- 先理解 Agent loop，再使用框架。
- 先完成单 Agent 的稳定性、评测与工程化，再学习 Multi-Agent。
- 不以“会多少框架”为目标，以能够解释设计取舍、失败模式和量化结果为目标。
- Coding Agent 不只作为开发工具使用，而作为成熟 Agent Runtime / Harness 的学习样本。
- 读源码不追求覆盖整个仓库；围绕 Agent loop、Tool、Context、Permission、Session、Recovery 等问题追完整调用链，并把可迁移设计落实到自己的项目。
- 产品、SDK 与招聘要求变化快，涉及版本和功能时以最新官方文档为准。
