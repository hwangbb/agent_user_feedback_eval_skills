---
name: user-feedback-eval-workflow
description: Guide a user through the full user-feedback eval workflow for the AI video/story agent: start from a raw trace, rewrite it into an eval-ready case, support either the default `input / output / context` schema or a user-provided dataset format, collect the actual agent output, run judging, then summarize findings and next actions.
---

# 用户反馈评测工作流

## 概述

这套 skill 用来串起完整流程：

- 从原始 trace 开始
- 改写成评测 case
- 让用户去真实 Agent / CMS 跑一遍
- 把实际输出带回来 judge
- 最后给出归因和修改建议

它会协调两套子 skill：

- `$user-feedback-trace-to-eval-case`
- `$user-feedback-eval-judge`

## 硬规则：改写、Judge、分析最好分开

这三个角色最好分开：

- `case_rewriter_llm`
- `judge_llm`
- `analysis_llm`

尽量不要让同一个模型、同一个上下文窗口同时负责改写、judge 和后续分析。

比较理想的做法是：

- 一个上下文负责改写 case
- 一个新上下文负责 judge
- 另一个新上下文负责分析和建议

如果没法做到完全隔离，要明确告诉用户存在污染风险。

## 工作流程

### Step 1：先收集最少材料，并确认评测集格式

先收这些内容：

- 原始 trace 或 bad case 材料
- 如果已经有了，也可以直接给改写后的 case
- 如果已经跑过 Agent，也可以直接给实际输出

同时确认用户想要的评测集格式：

- 用户上传模板
- 用户口头描述格式
- 用户使用默认格式 `input / output / context`

如果用户没有明确说，而格式会影响后续步骤，就只问一个很短的问题确认格式。

### Step 2：按选定格式改写 case

用 `$user-feedback-trace-to-eval-case` 的逻辑，产出：

- 默认格式下的 `input / output / context`
- 或用户自己定义的字段格式

然后明确告诉用户：

- 原始 trace 里保留了哪些关键问题
- 哪些上下文是补出来的
- 当前使用的评测集格式是什么
- 如果用了自定义格式，核心字段是怎么映射的

### Step 3：让用户确认内容和格式

确认项包括：

- 当前 case 内容是否满意
- 当前格式是否满意
- 是改内容，改格式，还是两者都改

如果用户中途换格式，就把当前 case 重新整理成新格式，后面所有步骤一起跟着改。

### Step 4：让用户去真实 Agent / CMS 里跑

case 确认后，不要直接开始 judge。

先让用户把这条 case 放到真实 Agent / CMS 里运行。

等用户回来后，收集：

- 实际 Agent 输出
- 可选的 trace evidence
- 可选的 generator / judge model 信息

### Step 5：按当前格式进入 Judge

用 `$user-feedback-eval-judge` 的逻辑评分。

如果用户一直用的是默认格式，就按默认字段收。

如果用户用的是自定义格式，就沿用用户的字段名和字段顺序；需要 judge 的时候，只做语义映射，不强行改回默认格式。

### Step 6：做归因和建议

Judge 结束后，至少回答这些问题：

- 实际输出有没有真的打到原问题
- 改写后的 case 是否够准
- 期望输出是否够明确
- judge 结果和评估标准是否一致
- 是 Agent 行为本身有问题，还是 case / judge 链路有问题

### Step 7：维护当前 case 列表

每处理完一条 case，都保留一个简短列表项，记录：

- case 名称
- issue type
- case format
- rewrite status
- judge status
- score
- main conclusion
- next action

## 交互规则

- 一次只问下一个必要问题
- 用户已经给够材料时，直接往下做
- 如果用户已经给了模板或格式，不要重复追问默认格式
- 如果用户改了格式，后面所有步骤都跟着切到新格式
- 如果 judge 结果和标准看起来不一致，要明确指出这是评估链路问题，不要硬把责任推给 Agent

## 默认回应结构

当你在改写 case 时：

1. `改编后的案例`
2. `保留的关键检验点`
3. `当前使用的评测集格式`
4. `请确认是否满意`

当 case 确认后：

1. `请去 agent / CMS 中运行这条改编后的 case`
2. `返回后粘贴实际输出结果`
3. `再确认是否继续跑 LLM Judge`

当你在跑 Judge 时：

1. `Judge 结果`
2. `判断依据`
3. `问题归纳`
4. `修改建议`
5. `当前改编列表`

必要时补一行：

- `rewrite_judge_isolated=true/false`
- `judge_analysis_isolated=true/false`
- `separation_risk=...`

## References

- Read [references/workflow-playbook-clean.md](references/workflow-playbook-clean.md) for the stage-by-stage playbook.
- Read [references/ledger-template-clean.md](references/ledger-template-clean.md) for the running list format.
