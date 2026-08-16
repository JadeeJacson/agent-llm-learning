# 模块 A 复习笔记：Agent 核心范式与经典工作

> 目标：复习并掌握 Agent 的底层心智模型。重点不是背框架 API，而是能够解释 Agent 为什么工作、什么时候应该使用、怎样形成闭环、Planning / Reflection / Tool Use 分别解决什么问题，以及 ReAct、Tree of Thoughts、Reflexion 在现代 Agent 体系中的位置。
>
> 本笔记对应主学习路线中的「模块 A：Agent 核心范式与经典工作」。完成后应能不依赖 LangChain、LangGraph、Agents SDK 等框架，解释并手写一个 Minimal Agent Loop。

---

## 1. 一句话理解 Agent

一个实用的工程定义：

> **Agent 是一个以 LLM 作为决策核心、以目标和指令约束行为、通过工具与环境交互，并在 Observation → Decision → Action 的循环中自主选择下一步，直到满足终止条件的系统。**

可以抽象为：

```text
Agent
=
Model
+ Tools
+ Instructions
+ State
+ Control Loop
+ Constraints
```

其中：

- **Model**：理解任务、推理、选择下一步动作；
- **Tools**：获取外部信息或改变外部环境；
- **Instructions**：目标、规则、权限和行为边界；
- **State**：保存当前任务状态、历史动作和环境反馈；
- **Control Loop**：不断「观察 → 决策 → 行动 → 再观察」；
- **Constraints**：最大步数、权限、预算、超时、终止条件等。

OpenAI 常用的基础拆分是 `Model + Tools + Instructions`；在真正实现 Agent Runtime 时，State、Loop 和 Constraints 通常会被进一步显式化。

参考：

- Anthropic, Building effective agents: https://www.anthropic.com/engineering/building-effective-agents
- OpenAI, A practical guide to building agents: https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/

---

## 2. 普通 LLM、Augmented LLM、Workflow、Agent 的区别

### 2.1 普通 LLM

最简单的模式：

```text
User
 ↓
LLM
 ↓
Answer
```

例如：

```text
用户：解释快速排序。
模型：快速排序是一种分治排序算法……
```

模型只完成一次或少量固定调用，没有控制一个开放式执行过程，也没有持续与环境形成行动闭环。

---

### 2.2 Augmented LLM

给 LLM 增加外部能力：

```text
             ┌─ Retrieval
User → LLM ──┼─ Tools
             └─ Memory
```

例如：

```python
tools = [
    search_web,
    calculator,
    read_file,
]
```

但：

> **有 Tool 不等于 Agent。**

例如：

```python
search_result = search_web(question)
answer = llm(question, search_result)
```

这里程序已经规定「一定先搜索，再调用 LLM」，因此整体仍然更接近固定 Workflow。

---

### 2.3 Workflow

Workflow 的核心特征不是「有没有 LLM」，而是：

> **主要执行路径由程序预先确定。**

例如：

```text
上传 PDF
  ↓
提取文本
  ↓
切分 Chunk
  ↓
分别总结
  ↓
合并摘要
  ↓
输出
```

即使中间多个节点调用 LLM，整体仍可以是 Workflow。

例如：

```python
text = read_pdf(file)
chunks = split(text)
summaries = [llm(chunk) for chunk in chunks]
result = llm(summaries)
```

执行拓扑仍然是程序员提前决定的。

---

### 2.4 Workflow 内也可以有 LLM 决策

不能简单认为：

```text
Workflow = 所有决定都由 if/else 完成
Agent = 只要 LLM 做决定就是 Agent
```

例如 Routing Workflow：

```text
             ┌→ 数学处理器
用户 → 分类器
             ├→ 编程处理器
             └→ 普通问答
```

分类器完全可以由 LLM 实现：

```python
category = llm_classify(question)

if category == "math":
    math_handler()
elif category == "code":
    code_handler()
else:
    general_handler()
```

虽然 LLM 决定了分支，但整个结构仍然是：

```text
固定入口
 ↓
固定分类节点
 ↓
固定分支集合
 ↓
固定终点
```

因此仍可以视为 Workflow。

---

### 2.5 Agent

当「下一步执行什么」主要由模型根据当前状态动态决定时，系统开始具有明显的 Agent 特征。

例如：

```text
用户：调研 Transformer 为什么比 RNN 更适合大规模语言模型。

Agent 拥有：
- search_web
- read_page
- save_note
- calculator
```

程序并未规定：

```text
必须搜索几次
必须先看哪篇文章
什么时候结束
```

而是由模型动态决定：

```text
Goal
 ↓
LLM
 ↓
是否需要搜索？
 ↓
Search
 ↓
Observation
 ↓
是否还缺信息？
 ↓
Read Page
 ↓
Observation
 ↓
是否完成？
 ↓
Final Answer
```

---

## 3. Workflow 与 Agent 最关键的区别：控制权

可以把问题压缩成一句话：

> **谁决定下一步执行什么？**

Workflow：

```python
next_step = program_logic(state)
```

Agent：

```python
next_step = llm(state, tools, instructions)
```

现实系统通常不是纯 Workflow 或纯 Agent，而是混合架构：

```text
Deterministic Code
       ↓
   Agent Loop
   ↙       ↘
Tool A    Tool B
   ↘       ↙
Deterministic Code
```

成熟工程的目标不是「尽可能 Agent 化」，而是把确定性逻辑留给代码，把真正难以提前写死的决策交给模型。

---

## 4. 什么时候不应该使用 Agent

假设任务是：

```text
读取 CSV
 ↓
删除空行
 ↓
计算平均值
 ↓
保存 CSV
```

直接代码更合适：

```python
df = pd.read_csv(...)
df = df.dropna()
mean = df["score"].mean()
df.to_csv(...)
```

没有必要让 Agent 每一步都思考：

```text
我应该先读 CSV 吗？
我应该删除空行吗？
我应该计算平均值吗？
```

把确定性程序改成 Agent 通常会导致：

```text
更慢
+ 更贵
+ 更不可预测
+ 更难测试
```

却没有增加实际能力。

因此：

> **Agent 不是目标，解决问题才是目标。**

---

## 5. 什么时候 Agent 更有价值

Agent 更适合：

> **任务目标明确，但完成路径无法事先完全确定。**

例如 Coding Agent：

```text
用户：这个仓库登录功能有 Bug，修一下。
```

程序员不可能提前写死：

```text
Step 1 打开 auth.py
Step 2 修改第 52 行
Step 3 修改 database.py
Step 4 跑某个测试
```

因为每个仓库、每个 Bug 都不同。

Agent 需要自己：

```text
查看目录
 ↓
搜索代码
 ↓
阅读文件
 ↓
形成假设
 ↓
修改代码
 ↓
运行测试
 ↓
观察失败
 ↓
继续调查
 ↓
再次修改
 ↓
测试成功
```

这里「下一步是什么」本身就是问题的一部分。

适合 Agent 的典型特征：

- 问题开放；
- 正确路径难以提前预测；
- 需要根据中间反馈动态调整；
- 依赖非结构化信息和复杂判断；
- 用规则穷举会非常复杂。

---

## 6. Agent 的最核心结构：Loop

Agent 最关键的不是 Prompt，而是闭环：

```text
                    ┌───────────────┐
                    │     Goal      │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Instructions  │
                    └───────┬───────┘
                            ↓
                ┌───────────────────────┐
                │      Current State    │
                └───────────┬───────────┘
                            ↓
                       ┌─────────┐
                       │   LLM   │
                       └────┬────┘
                            ↓
                    Decide Next Step
                       ↙          ↘
                 Final Answer     Tool Call
                       ↓              ↓
                     STOP           Action
                                      ↓
                                   Tool /
                                Environment
                                      ↓
                                  Observation
                                      ↓
                                  Update State
                                      │
                                      └────→ LLM
```

最小逻辑：

```text
Ask Model
   ↓
Finished?
 ├─ Yes → Final
 └─ No
      ↓
   Execute Tool
      ↓
   Observation
      ↓
   Update State
      ↓
      Loop
```

---

## 7. Agent Loop 的最小伪代码

```python
MAX_STEPS = 5

messages = [
    {"role": "user", "content": user_input}
]

for step in range(MAX_STEPS):
    response = llm(messages)

    if response["type"] == "final":
        return response["content"]

    if response["type"] == "tool_call":
        result = execute_tool(
            response["tool_name"],
            response["arguments"]
        )

        messages.append(response)
        messages.append({
            "role": "tool",
            "content": str(result)
        })

return "Agent 超过最大执行步数"
```

这里已经具备一个 Agent 的骨架：

```text
State
 ↓
LLM
 ↓
Decision
 ↓
Tool Call
 ↓
Tool Execution
 ↓
Observation
 ↓
State Update
 ↓
LLM
```

---

## 8. Tool Call 与 Tool Execution 必须区分

模型输出：

```json
{
  "type": "tool_call",
  "tool_name": "calculator",
  "arguments": {
    "a": 123,
    "b": 456,
    "operation": "multiply"
  }
}
```

这只是：

> **Action Proposal / Action Decision**

模型并没有真的完成计算。

真正执行的是 Runtime：

```python
result = execute_tool(
    tool_name,
    arguments
)
```

得到：

```text
56088
```

之后 Runtime 把结果作为 Observation 放回上下文，模型下一轮才能继续决策。

因此：

```text
LLM 决定动作
≠
LLM 自己执行动作
```

---

## 9. Tool 的两个角色：手和眼睛

Tool 不只是「函数调用」，它可以扮演两种角色：

```text
Actuator（执行器）
+
Sensor（传感器）
```

例如：

### 偏 Action

```text
delete_file
write_file
send_email
update_database
```

它们会改变环境。

### 偏 Observation

```text
read_file
search_web
query_database
run_tests
```

它们主要向 Agent 提供新的环境信息。

`run_tests()` 尤其重要：它表面上是在执行测试，真正价值却是产生高质量 Ground Truth：

```text
PASS / FAIL
错误堆栈
失败测试
```

---

## 10. 为什么 Observation 是 Agent 的核心

没有 Observation：

```text
LLM
 ↓
想
 ↓
继续想
 ↓
继续猜
 ↓
Answer
```

有 Observation：

```text
LLM
 ↓
Action
 ↓
真实环境
 ↓
Observation
 ↓
LLM
```

Agent 因而拥有一个闭环反馈系统。

Coding Agent 例子：

```text
Agent 修改 auth.py
 ↓
run_tests()
 ↓
Observation:
3 failed, 27 passed
 ↓
Agent：还没修好
 ↓
继续定位
```

再次修改：

```text
run_tests()
 ↓
Observation:
30 passed
```

这时「修好了」至少有外部环境证据，而不是模型自己的主观判断。

---

## 11. ReAct：Reasoning + Acting

论文：

- ReAct: Synergizing Reasoning and Acting in Language Models
- https://arxiv.org/abs/2210.03629

核心思想：

```text
Thought / Reason
 ↓
Action
 ↓
Observation
 ↓
Thought / Reason
 ↓
Action
 ↓
Observation
 ↓
...
```

ReAct 的关键不是「让模型多想几步」，而是：

> **Reasoning 与 Acting 交替，Action 从环境中取得新的 Observation，再影响后续决策。**

### 与 Chain-of-Thought 的区别

Chain-of-Thought：

```text
Question
 ↓
Reason
 ↓
Reason
 ↓
Reason
 ↓
Answer
```

主要在模型内部信息空间中推进。

ReAct：

```text
Question
 ↓
Reason
 ↓
Action
 ↓
Environment
 ↓
Observation
 ↓
Reason
 ↓
Action
 ↓
Environment
 ↓
Observation
 ↓
Answer
```

最大的变化：

```text
模型内部世界
   ↕
真实环境
```

形成闭环。

---

## 12. ReAct 与现代 Tool-calling Agent 的关系

经典 ReAct 经常写成：

```text
Thought
Action
Observation
Thought
Action
Observation
```

现代 API 更常见：

```text
LLM Response
 ↓
Tool Call?
 ↓ Yes
Execute Tool
 ↓
Tool Result
 ↓
LLM
```

现代 Agent 不一定会把详细 `Thought:` 文本显式暴露出来。

因此：

> 不应该用「有没有输出 Thought 字段」来判断是不是 ReAct-like Agent。

ReAct 真正要学习的是：

```text
Reasoning
+ Action
+ Environment Feedback
```

而不是一种固定文本格式。

---

## 13. Reactive Agent 的局限：局部合理，整体迷路

如果 Agent 永远只是：

```text
观察当前状态
 ↓
决定下一步
 ↓
观察
 ↓
再决定
```

就可能发生：

```text
每一步都看起来合理
但整个任务没有方向
```

例如行业调研 Agent：

```text
搜索市场规模
 ↓
看到某公司
 ↓
搜索公司
 ↓
看到某技术
 ↓
搜索技术
 ↓
看到竞争对手
 ↓
继续搜索……
```

结果可能调用 20 多次工具，却没有形成完整报告。

这就是一种典型的：

> **Myopic Decision Making（短视决策）**

因此需要 Planning。

---

## 14. Planning：解决全局方向问题

Planning 不是「让模型多想一会儿」。

更准确地说：

> **Planning 是对未来动作序列进行组织、预测和选择。**

如果目标需要：

```text
a1 → a2 → a3 → ... → an
```

纯 Reactive Agent 更像：

```text
看 s0 → 选 a1
看 s1 → 选 a2
看 s2 → 选 a3
```

Planning 会引入：

```text
Goal
 ↓
Plan
 ↓
Execute
```

但真实环境并不可完全预测，所以真正实用的形式通常是：

```text
Plan
 ↓
Execute
 ↓
Observation
 ↓
Evaluate
 ↓
Re-plan
 ↓
Continue
```

---

## 15. 三种常见 Planning 形式

### 15.1 Reactive Planning

只规划下一步：

```text
State
 ↓
Next Action
 ↓
Observation
 ↓
State
 ↓
Next Action
```

优点：

- 灵活；
- 能适应环境变化；
- 实现简单。

缺点：

- 容易短视；
- 可能绕圈；
- 可能浪费 Tool Call。

ReAct 非常接近这种局部规划方式。

---

### 15.2 Plan-and-Execute

先生成一个粗计划，再逐步执行。

例如：

```text
Goal：调查公司收入下降原因

Plan：
1. 获取最近 12 个月收入
2. 按产品拆分
3. 按地区拆分
4. 找出主要下降来源
5. 搜索相关外部事件
6. 形成原因假设
7. 验证假设
8. 写报告
```

优点：全局结构清晰。

缺点：最初计划可能建立在错误假设上。

---

### 15.3 Planning + Replanning

更实际：

```text
Goal
 ↓
Initial Plan
 ↓
Execute one/few steps
 ↓
Observation
 ↓
Evaluate progress
 ↓
Update Plan
 ↓
Continue
```

因此：

```text
ReAct
≈ 解决局部行动闭环

Planning
≈ 解决全局方向
```

二者并不冲突，可以同时存在。

---

## 16. Coding Agent 中的 Planning 示例

用户：

```text
登录失败，帮我修。
```

粗计划：

```text
1. 复现错误
2. 找到认证调用链
3. 判断 Root Cause
4. 做最小修改
5. 执行相关测试
6. 执行完整测试
7. 检查 Diff
```

第一步：

```text
run_tests(authentication)
```

Observation：

```text
失败发生在 refresh_token()
```

随后更新计划：

```text
1. ✓ 复现错误
2. 优先追踪 refresh_token 调用链
3. 检查 token expiry calculation
...
```

Planning 的目的不是预测所有未来细节，而是：

> **维持任务整体结构，避免 Agent 漂移。**

---

## 17. Planning 的本质：搜索 Action Space

假设每一步有 5 个可选动作，任务大约有 10 步。

粗略可能路径：

```text
5^10
```

Agent 不可能穷举全部路径。

因此 Planning 本质上在解决：

> **如何有效搜索动作空间。**

这自然引出 Tree of Thoughts。

---

## 18. Tree of Thoughts：从单路径推理到多路径搜索

论文：

- Tree of Thoughts: Deliberate Problem Solving with Large Language Models
- https://arxiv.org/abs/2305.10601

传统 Chain-of-Thought 更像：

```text
A → B → C → D
```

如果 B 错了，后续可能全部建立在错误前提上。

Tree of Thoughts 允许：

```text
                    Start
                  /   |   \
                 A    B    C
                / \  / \  / \
               D  E F  G H  I
```

基本思想：

```text
生成多个候选
 ↓
评价候选
 ↓
选择更有希望的路径
 ↓
继续展开
 ↓
必要时回溯
```

可以粗略理解为：

```text
Chain of Thought
≈ 单路径搜索

Tree of Thoughts
≈ 显式多路径搜索
```

---

## 19. 为什么不是所有 Agent 都应该使用 ToT

ToT 会带来：

```text
更多模型调用
+ 更多 token
+ 更高 latency
+ 更多搜索控制逻辑
```

它更适合：

- 搜索空间明显；
- 中间路径可以评价；
- 错误前缀代价高；
- 需要探索、lookahead 或 backtracking。

例如：

- 复杂数学；
- 组合问题；
- 谜题；
- 某些约束规划任务。

查天气这样的任务没有必要生成五种策略再进行搜索。

所以 Tree of Thoughts 更适合作为：

> **Planning / Search 思想的经典来源**

而不是现代 Agent 项目必须套用的模板。

---

## 20. Observation 与 Reflection 必须区分

例如测试返回：

```text
Observation:

test_refresh_token FAILED
expected 200
actual 401
```

这是外部事实。

Reflection：

```text
我之前假设问题来自 token parser，
但 refresh token 路径仍然失败。
下一步应该检查 refresh_token() 中的 expiry 时间单位，
而不是继续修改 parser。
```

这是 Agent 对反馈的解释和策略修正。

因此：

```text
Observation
=
环境实际发生了什么
```

```text
Reflection
=
这意味着什么，以及下一步应该改变什么
```

---

## 21. Reflection 为什么有用

没有 Reflection 时：

```text
失败
 ↓
重新尝试
 ↓
采用类似策略
 ↓
再次失败
```

加入 Reflection：

```text
Experience
 ↓
Feedback
 ↓
Lesson
 ↓
Change Strategy
 ↓
Next Attempt
```

它的目标是：

> **把一次具体失败压缩成可迁移的经验。**

---

## 22. Reflexion：语言形式的经验学习

论文：

- Reflexion: Language Agents with Verbal Reinforcement Learning
- https://arxiv.org/abs/2303.11366

非常容易误解的一点：

> Reflexion 并不是通过梯度更新模型参数。

传统 RL 可以粗略理解成：

```text
Agent
 ↓
Action
 ↓
Reward
 ↓
更新 Policy 参数
```

而 Reflexion：

```text
Agent
 ↓
Attempt
 ↓
Feedback
 ↓
Reflection
 ↓
写入 Episodic Memory
 ↓
下一次上下文带上经验
```

模型权重没有改变。

改变的是：

```text
Context / Memory
```

可以粗略记忆：

```text
传统 RL：
经验主要写进参数

Reflexion：
经验以语言形式写进上下文 / 记忆
```

---

## 23. Reflection 的三个尺度

### 23.1 Step-level Reflection

每一步后判断：

```text
刚才结果合理吗？
```

例如：

```text
search_web
 ↓
结果明显无关
 ↓
修改 query
```

---

### 23.2 Episode-level Reflection

一次完整尝试失败后总结：

```text
Attempt 1
 ↓
FAILED
 ↓
总结失败原因
 ↓
Attempt 2
```

经典 Reflexion 更接近这种模式。

---

### 23.3 Long-term Reflection

从多次任务中提炼一般经验：

```text
修改数据库 Schema 前先检查 Migration。
修改认证逻辑后必须运行 refresh-token tests。
Tool 返回空结果时不要把 empty 当成 negative evidence。
```

这会自然进入后续 Memory / Context Engineering 的学习范围。

---

## 24. Reflection 最大的风险：反思也可能是错的

不要认为：

```text
让 LLM 反思
=
自动变聪明
```

例如真实失败原因是：

```text
数据库连接超时
```

模型却总结：

```text
应该修改 SQL 查询逻辑
```

然后：

```text
错误 Reflection
 ↓
写入 Memory
 ↓
后续 Agent 更坚定地做错事
```

这就是典型的 Memory Pollution / Incorrect Self-feedback。

Reflection 最可靠的时候通常是有强外部反馈：

- 单元测试；
- 编译器错误；
- 数据库约束；
- Validator；
- Reward；
- Human Feedback。

因此更可靠的结构是：

```text
Action
 ↓
External Feedback
 ↓
Compare with Goal
 ↓
Identify Failure
 ↓
Reflection
 ↓
Change Strategy
```

而不是：

```text
模型凭感觉：
“我再想想自己哪里错了。”
```

---

## 25. Retry、Reflection、Replan、Fallback 的区别

这些概念以后在 Production Agent Engineering 中还会再次出现。

### Retry

```text
失败
 ↓
重新执行同样或近似操作
```

适合临时网络错误、偶发 API 错误等。

### Reflection

```text
失败
 ↓
分析失败原因
 ↓
提炼经验
```

### Replan

```text
现有计划不合适
 ↓
改变后续任务路径
```

### Fallback

```text
主方案不可用
 ↓
切换备用模型 / 工具 / 路径
```

因此：

```text
Retry ≠ Reflection ≠ Replan ≠ Fallback
```

---

## 26. ReAct、Tree of Thoughts、Reflexion 的关系

可以这样复习：

| 方法 | 主要解决问题 | 核心思想 |
| --- | --- | --- |
| ReAct | 当前下一步做什么 | Reason → Act → Observe |
| Tree of Thoughts | 有多条可能路径怎么办 | Explore → Evaluate → Search / Backtrack |
| Reflexion | 失败后如何减少重犯 | Feedback → Reflect → Memory → Retry |

换一个角度：

```text
ReAct
=
局部行动闭环

Tree of Thoughts
=
规划 / 搜索能力

Reflexion
=
经验反馈 / 学习能力
```

现代 Agent 一般不会真的写成：

```python
agent = ReAct() + TreeOfThought() + Reflexion()
```

这些更应该被理解为设计思想，最后往往融入：

```text
Runtime
Prompt
State
Memory
Tool Loop
Evaluation
```

中。

---

## 27. 三个时间尺度理解 Agent 智能

这是一个非常重要的统一视角。

### 微观：下一步做什么？

对应：

```text
ReAct
Action Selection
```

### 中观：整个任务怎么推进？

对应：

```text
Planning
Replanning
```

### 宏观：从失败中学到了什么？

对应：

```text
Reflection
Memory
```

可以记成：

```text
              Agent Intelligence

微观：下一动作选择
      ReAct

中观：任务结构组织
      Planning / Replanning

宏观：从经验调整行为
      Reflection / Memory
```

---

## 28. Evaluator：为什么评测其实可以进入 Agent Loop

成熟 Agent 可以继续抽象为：

```text
Actor
+
Environment
+
Evaluator
```

Actor：

```text
LLM Agent
```

Environment：

```text
Tools / Web / Shell / DB
```

Evaluator：

```text
Tests
Rules
Validator
Judge
Human
Reward
```

整体：

```text
Actor
 ↓
Action
 ↓
Environment
 ↓
Observation
 ↓
Evaluator
 ↓
Feedback
 ↓
Actor
```

所以 Eval 不一定只是项目最后写的一份报告，它也可以直接成为运行时的一部分。

---

## 29. Generator / Critic 与 Evaluator-Optimizer

一个常见模式：

```text
Generator
 ↓
Solution
 ↓
Critic / Evaluator
 ↓
Feedback
 ↓
Generator
 ↓
Improved Solution
```

如果程序固定规定：

```text
Generate
 ↓
Evaluate
 ↓
Rewrite
 ↓
Evaluate
```

它仍然可能属于 Workflow。

这再次证明：

> Planning、Reflection、多模型并不会自动让系统变成 Agent。

真正要看的是：

```text
谁控制整体执行流程？
```

---

## 30. 一个完整 Coding Agent 例子

用户：

```text
修复项目中的登录 Bug。
```

### 30.1 Planning

```text
1. 复现 Bug
2. 定位认证路径
3. 确认 Root Cause
4. 做最小修改
5. 跑相关测试
6. 跑完整测试
7. 检查 Diff
```

### 30.2 Tool Use

```text
run_tests("auth")
```

### 30.3 Observation

```text
FAILED

test_refresh_token
expected 200
actual 401
```

### 30.4 ReAct 式局部决策

```text
下一步应该搜索 refresh_token
```

执行：

```text
search_code("refresh_token")
```

Observation：

```text
auth/token.py:82
```

### 30.5 继续获取信息

```text
read_file("auth/token.py")
```

Observation：

```text
expiry_timestamp = expires_in * 1000
```

### 30.6 形成假设并修改

```text
可能存在秒 / 毫秒单位错误
```

### 30.7 外部验证

```text
run_tests("auth")
```

Observation：

```text
PASS
```

### 30.8 Evaluation

仍不能马上结束，因为计划要求完整测试：

```text
run_tests()
```

Observation：

```text
2 unrelated tests failed
```

### 30.9 Reflection

```text
修改修复了 auth，
但可能影响共享 timestamp representation。
需要缩小修改范围。
```

### 30.10 Replanning

```text
1. 检查 timestamp 公共接口
2. 缩小修改范围
3. 再跑 auth tests
4. 再跑 full tests
```

这个例子同时包含：

```text
Planning
ReAct
Tool Use
Observation
Evaluation
Reflection
Replanning
```

它们不是互相竞争的 Agent 类型，而是完整 Agent Loop 中不同时间尺度的能力。

---

## 31. Minimal Agent：最小代码骨架

### 31.1 Tool

```python
def calculator(a: float, b: float, operation: str):
    if operation == "add":
        return a + b
    if operation == "subtract":
        return a - b
    if operation == "multiply":
        return a * b
    if operation == "divide":
        return a / b
    raise ValueError("未知操作")


TOOLS = {
    "calculator": calculator
}
```

### 31.2 Tool Runtime

```python
def execute_tool(tool_name, arguments):
    tool = TOOLS[tool_name]
    return tool(**arguments)
```

### 31.3 Agent Loop

```python
def run_agent(user_input):
    messages = [
        {
            "role": "user",
            "content": user_input
        }
    ]

    MAX_STEPS = 5

    for step in range(MAX_STEPS):
        response = llm(messages)

        if response["type"] == "final":
            return response["content"]

        if response["type"] == "tool_call":
            tool_name = response["tool_name"]
            arguments = response["arguments"]

            result = execute_tool(
                tool_name,
                arguments
            )

            messages.append(response)

            messages.append({
                "role": "tool",
                "name": tool_name,
                "content": str(result)
            })

    return "Agent 超过最大执行步数"
```

这已经是一个真正的 Minimal Agent 心智模型。

---

## 32. Minimal Agent 的完整执行轨迹

用户：

```text
计算 123 × 456
```

第一次模型调用：

```text
Current State
 ↓
LLM
 ↓
Decision:
调用 calculator
```

模型输出：

```json
{
  "type": "tool_call",
  "tool_name": "calculator",
  "arguments": {
    "a": 123,
    "b": 456,
    "operation": "multiply"
  }
}
```

Runtime 执行：

```text
calculator(123, 456, "multiply")
 ↓
56088
```

将结果作为 Observation 放回：

```text
User:
计算 123 × 456

Assistant:
调用 calculator

Tool:
56088
```

第二次模型调用：

```text
LLM 判断信息已经足够
 ↓
Final Answer
```

最终：

```text
123 × 456 = 56088
```

---

## 33. 为什么这个 Minimal Agent 已经算 Agent

因为程序没有写死：

```python
result = calculator(123, 456)
```

而是模型自己决定：

```text
是否需要调用 Tool
 ↓
调用哪个 Tool
 ↓
传哪些参数
 ↓
什么时候结束
```

如果用户问：

```text
你好
```

模型可以直接：

```text
Final Answer
```

而不调用任何 Tool。

因此执行路径已经从：

```text
固定程序
```

变成：

```text
模型根据 State 动态决定路径
```

---

## 34. 为什么必须有 MAX_STEPS

如果写：

```python
while True:
    ...
```

模型可能：

```text
Tool A
 ↓
Tool B
 ↓
Tool A
 ↓
Tool B
 ↓
...
```

无限循环。

因此：

```python
MAX_STEPS = 5
```

本质上已经是一种最原始的：

```text
Safety Boundary
+
Budget Control
```

以后会扩展为：

```text
max_steps
max_tool_calls
max_tokens
max_cost
timeout
```

---

## 35. Function Calling 在整个体系中的位置

Minimal Agent 中我们假设 `llm(messages)` 能稳定返回：

```json
{
  "type": "tool_call",
  "tool_name": "calculator",
  "arguments": {
    "a": 123,
    "b": 456
  }
}
```

如果只是靠 Prompt 要求模型输出 JSON，现实中可能出现：

```text
好的，我来调用工具。

TOOL_CALL:
calculator(123, 456)
```

或者返回非法 JSON，导致程序无法稳定解析。

因此 Structured Output / Function Calling / Tool Calling 主要解决：

> **模型如何可靠地把「我要采取什么 Action」表达成程序可执行的结构化数据。**

必须牢记：

> **Function Calling 不是 Agent。**

它只是：

```text
Model
↕
Agent Runtime
```

之间的一种结构化通信机制。

真正的 Agent 仍然来自：

```text
LLM
 ↓
Decision
 ↓
Tool Call
 ↓
Execution
 ↓
Observation
 ↓
LLM
 ↓
Loop
```

---

## 36. 后续模块如何映射回 Agent Loop

模块 A 建立的 Agent Loop 是整个后续路线的坐标系。

### Memory

解决：

```text
Current State 应该保存什么？
```

### Context Engineering

解决：

```text
哪些 State 应该送进 LLM？
```

### Function Calling

解决：

```text
Action 如何结构化表达？
```

### MCP

解决：

```text
Tools / Resources 如何标准化暴露给 Agent？
```

### LangGraph

解决：

```text
State 和控制流如何工程化编排？
```

### Tracing

记录：

```text
LLM → Action → Observation → LLM
```

### Evaluation

判断：

```text
这条 Agent Trajectory 到底好不好？
```

### Permission / Sandbox

限制：

```text
Agent 可以对 Environment 做什么？
```

因此 Agent Loop 不是「入门知识学完就丢」，而是后面所有模块的统一坐标系。

---

## 37. Agent 的主要失败模式

因为下一动作由概率模型参与决策，Agent 可能出现：

### Tool Selection Error

```text
选错 Tool
```

### Argument Error

```text
Tool 选对，但参数错
```

### Premature Termination

```text
任务还没完成就 Final
```

### Over-execution

```text
已经有答案仍继续搜索 / 调工具
```

### Looping

```text
Tool A
 ↓
Tool B
 ↓
Tool A
 ↓
Tool B
```

### Error Propagation

```text
错误 Observation
 ↓
错误推理
 ↓
错误 Action
 ↓
错误进一步扩大
```

### Planning Drift

```text
局部每一步合理
 ↓
整体偏离目标
```

### Reflection Error

```text
错误解释失败原因
 ↓
错误经验写入 Memory
```

这些失败模式解释了为什么后续必须学习：

```text
max_steps
retry
fallback
permission
human-in-the-loop
tracing
eval
timeout
cost control
```

---

## 38. 模块 A 最重要的三句话

### 第一

> **有 LLM 不等于 Agent，有 Tool 也不等于 Agent。**

关键看：

```text
LLM 是否在动态控制任务执行过程。
```

### 第二

> **Agent 最核心的结构不是 Prompt，而是 Loop。**

```text
Observe
 ↓
Decide
 ↓
Act
 ↓
Observe
 ↓
...
```

### 第三

> **Agent 最大价值在于处理「事先不知道完整执行路径」的任务。**

路径高度确定：

```text
Code / Workflow
```

路径需要动态探索：

```text
Agent
```

---

## 39. 复习时的统一知识图

```text
                         USER GOAL
                            │
                            ↓
                    ┌───────────────┐
                    │ Instructions  │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Current State │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │   Planning    │
                    │  全局怎么做？ │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │   Decision    │
                    │ 下一步是什么？│
                    └───────┬───────┘
                            ↓
                       Tool Call
                            ↓
                    ┌───────────────┐
                    │ Environment   │
                    └───────┬───────┘
                            ↓
                      Observation
                            ↓
                    ┌───────────────┐
                    │  Evaluation   │
                    │ 是否有进展？  │
                    └───────┬───────┘
                            ↓
                 ┌──────────┴──────────┐
                 ↓                     ↓
              Success                Failure
                 ↓                     ↓
          Goal Complete            Reflection
                 ↓                     ↓
               STOP               Update State
                                       ↓
                                   Re-planning
                                       │
                                       └──→ Loop
```

对应经典工作：

```text
ReAct
→ Reasoning + Acting + Observation

Tree of Thoughts
→ Planning / Search / Backtracking

Reflexion
→ Feedback + Reflection + Episodic Memory
```

---

## 40. 模块 A 自检题

不看笔记，尝试口头回答以下问题。

1. 普通 LLM、Augmented LLM、Workflow、Agent 有什么区别？
2. 为什么「有 Tool」不等于 Agent？
3. Workflow 中能不能让 LLM 做判断？为什么仍可能是 Workflow？
4. Workflow 与 Agent 最关键的边界是什么？
5. 什么类型的问题不应该 Agent 化？
6. 什么类型的问题更适合 Agent？
7. Agent Loop 最小结构是什么？
8. Tool Call 和 Tool Execution 有什么区别？
9. Tool 为什么既可以是「手」也可以是「眼睛」？
10. Observation 为什么是 Agent 闭环的关键？
11. ReAct 相比 Chain-of-Thought 增加了什么？
12. 为什么现代 Tool-calling Agent 不一定显式输出 Thought？
13. Reactive Agent 为什么容易局部合理但整体迷路？
14. Planning 真正解决什么问题？
15. 为什么 Planning 通常需要 Replanning？
16. Tree of Thoughts 相比线性 CoT 改变了什么？
17. 为什么不能所有 Agent 都使用 ToT？
18. Observation 和 Reflection 有什么区别？
19. Reflexion 有没有更新 LLM 权重？
20. 为什么 Reflection 最好建立在可靠外部反馈之上？
21. Retry、Reflection、Replan、Fallback 有什么区别？
22. ReAct、Planning、Reflection 是三种互斥 Agent，还是同一 Agent 可同时具有的能力？
23. Evaluator 为什么可以进入 Agent Loop？
24. 为什么 Generator + Critic 不一定就是 Agent？
25. Minimal Agent 为什么必须设置 MAX_STEPS？
26. Function Calling 在 Agent 系统中解决的是哪一层问题？
27. 为什么说 Agent Loop 是后续 Memory、MCP、LangGraph、Eval、Sandbox 的共同坐标系？

如果这些问题能够不看笔记连续讲清楚，模块 A 的理论部分基本已经掌握。

---

## 41. 推荐原始资料

### 必读

1. ReAct: Synergizing Reasoning and Acting in Language Models  
   https://arxiv.org/abs/2210.03629

2. Anthropic: Building effective agents  
   https://www.anthropic.com/engineering/building-effective-agents

### 理解 Planning / Search

3. Tree of Thoughts: Deliberate Problem Solving with Large Language Models  
   https://arxiv.org/abs/2305.10601

### 理解 Reflection / Verbal Learning

4. Reflexion: Language Agents with Verbal Reinforcement Learning  
   https://arxiv.org/abs/2303.11366

### 工程补充

5. OpenAI: A practical guide to building agents  
   https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/

6. Anthropic Engineering：Tool / Context / Agent 相关工程文章  
   https://www.anthropic.com/engineering

---

## 42. 下一步

模块 A 的理论学习完成后，不要继续堆更多 Agent 论文。下一步应该进入代码：

```text
模块 A 理论
 ↓
手写 Minimal Agent
 ↓
理解 Tool Schema
 ↓
Structured Output
 ↓
Function Calling / Tool Calling
 ↓
模块 B
```

推荐第一个实验只保留：

```text
LLM
+ 1~3 个 Tools
+ Agent Loop
+ MAX_STEPS
+ Tool Error
+ Trace / Log
```

暂时不要加入：

```text
LangGraph
MCP
RAG
Memory
Multi-Agent
```

先确保自己能够清楚解释：

> **模型做了什么、Runtime 做了什么、Tool 做了什么、Observation 怎样回到模型、为什么会继续下一轮，以及什么时候退出循环。**
