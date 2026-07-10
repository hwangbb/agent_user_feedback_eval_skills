---
name: user-feedback-trace-to-eval-case
description: Rewrite raw user-feedback traces, pasted execution logs, bad cases, or agent regression examples into eval-ready cases for the AI video/story agent. Support either the default `input / output / context` schema or a user-provided dataset format. Use when Codex needs to turn a real trace into a dataset row while preserving the agent's actual operating logic: query-first, clarify when needed, create world/story/assets when required, prefer intermediate deliverables such as worldbuilding, outline, prompts, and shot text over final media, and reconstruct enough missing context to make the case executable.
---

# 用户反馈 Trace 改写评测 Case

## 概述

把零散、混乱的用户反馈 trace 改写成可放进评测集的 case。

改写后的 case 需要尽量保留原 trace 真正想测的问题，同时去掉因为上下文缺失、指代不清、工具噪音带来的额外干扰。

## 工作方式

### 1. 先把 trace 当成执行材料来读，不要直接把它当成评测题

先抽出这些信息：

- 用户真正要做什么
- 当前能看到的前置条件和上下文
- Agent 实际走了什么路径
- 哪些步骤失败了，哪些成功了，哪些只产出了文本
- 真正值得进评测集的问题是什么

不要直接把原 trace 原封不动抄成数据集一行。

### 2. 补齐缺失的执行上下文

如果 trace 里有“这个角色”“继续上面的故事”“改这个分镜”这类指代，要改写成明确、简洁的上下文。

如果原 case 依赖上游信息，就把真正需要的前置条件补进 `input` 或目标格式里的上下文字段，例如：

- world / world style
- story premise or outline
- episode scope
- role identity
- scene identity
- asset type
- target medium
- style constraints
- duration, shot count, lens framing, language, and other execution-critical parameters

如果 trace 本身缺了关键前置条件，但原意已经很明确，就补足最小必要信息，让 case 能重新执行。不要借这个机会把任务改成别的东西。

### 3. 按真实 Agent 逻辑改写 case

改写后的 case 要符合这类 Agent 的常见工作方式：

- Agent 往往会先查、先问，再执行
- 有些任务需要先创建 world、story、episode、entity、asset 这类前置对象
- 很多任务先产出的是中间结果，比如 worldbuilding、outline、shot text、image prompt、video prompt、asset prompt、rewrite prompt
- 很多好 case 更适合测这些中间结果，而不是直接测最终视频效果
- 如果原 trace 明确说明媒体生成链路本来就不稳定，评测目标应尽量落在真实可交付的结果上

避免写成“帮我直接生成一个视频看看效果”这类很难稳定判断的开放式请求。

### 4. 先确定评测集格式

支持三种方式：

- 用户直接上传一个评测集模板
- 用户口头描述字段格式
- 用户没有指定时，使用默认格式 `input / output / context`

处理规则：

- 如果用户上传了模板，就按模板字段组织结果
- 如果用户描述了格式，就按描述组织结果
- 如果用户明确说用默认格式，就用 `input / output / context`
- 如果用户没有说清楚，而格式会明显影响后续步骤，只问一个简短问题确认格式
- 如果用户中途改了格式，要把当前 case 重新整理到新格式里，后续 judge 和 workflow 也按新格式继续

### 5. 把 case 整理成用户指定的格式

默认格式是：

- `input`
- `output`
- `context`

但如果用户指定了别的字段名或模板，也要照着来。

无论最终导出的字段名是什么，都要保留下面这几个语义位置：

- 用户请求本体
- 期望交付边界
- 补充上下文 / 标签 / 前置条件

如果用户换了格式，只改字段组织方式，不要把 case 本身的判断标准改掉。

### 6. 确保这个 case 真的可评估

在交付前检查：

- 改写后的请求是否已经足够具体
- 如果 Agent 可以先查或先问，是否在期望输出里保留了这件事
- 如果原 trace 只适合交付文本或 prompt，是否避免把目标写成必须生成最终媒体
- 是否区分了“过程里有失败调用”和“最终应判为失败”
- 是否去掉了内部 id、tool 名、system prompt 和无关的 trace 噪音

## 硬规则

- 保留原来的评测目标，不要把 prompt 改写类 case 改成最终视频类 case
- 只要原 trace 依赖上文，就把隐含信息展开成明确上下文
- 如果 trace 缺上下文，补最少、最必要的前置条件
- 上下文要短，要够用，不要把整段 trace 都塞进 `input`
- 优先保留产品里本来就更稳定的中间交付物：world、outline、角色 prompt、场景 prompt、资产 prompt、分镜、修改 prompt
- 除非原 case 就是在测媒体生成提交链路，否则不要强行要求最终媒体生成
- 如果 trace 里有 world 不存在、权限限制、容量限制这类报错，不要一看到报错就把 case 改写成“工具失败”
- 如果用户指定了自定义评测集格式，后续所有导出和衍生步骤都要沿用这个格式

## 输出要求

如果用户没有指定格式，默认输出：

```text
input: <改写后的用户请求>
output: <期望行为 / 交付边界>
context: <紧凑的上下文字符串>
```

如果用户指定了模板或字段格式，就严格按用户给的格式输出。

如果用户还要求 CSV-ready 内容，就按用户当前使用的字段顺序输出一行。

## References

- Read [references/rewrite-rules-clean.md](references/rewrite-rules-clean.md) for the rewrite checklist, issue taxonomy, and format-handling examples before producing the final case.
