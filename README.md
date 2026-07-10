# User Feedback Eval Skills for Codex

一套面向**生成视频类AI Agent**的 Codex skills，用来把真实用户反馈整理成可复用的回归评测流程。

它适合处理已经在agent/企业管理系统（完美适配cms）出现过的bad case：先把trace改写成稳定、可执行的评测case，再把实际agent输出放回到该skill，来做进一步 judge，最后反馈给用户错误归因和修改建议。

## Why

很多用户反馈都很有价值，但直接拿来做评测，通常会遇到几个问题：

- 上下文缺失，无法完全复现bad case至评测集，或携带上下文体量过大不好操作
- 指代很多，离开原 trace 很难定位
- 任务目标过于开放，不方便判断对错
- 过程里有失败调用，但最终结果不一定真的失败
- 收集到的用户反馈难以沉淀成完整的回归集

这套 skills 的主要目标是把这类素材整理成更稳定的评测资产，服务于agent优化迭代。

## Features

- 从原始trace、bad case、执行日志里改写出 `input/output/context`
- 补齐最小必要上下文，让case更容易执行
- 保留大多数agent的真实工作逻辑：先查、先问、再执行，必要时先建world/story/entity等固定资产
- 更适合评测agent真实执行中的中间产物，比如世界观、故事大纲、角色资产prompt、场景资产prompt、分镜、prompt修改
- Judge时会区分“过程里的失败调用”和“最终真实交付状态”，减少机械误判
- 把改写、复跑、judge、归因建议串成一条完整工作流

## Included Skills

### `user-feedback-trace-to-eval-case`

把原始trace/bad case/粘贴日志改写成评测集常用格式`input, output, context`。

### `user-feedback-eval-judge`

基于改写后的case和实际agent输出做LLM Judge，返回结构化评分和问题归因。

### `user-feedback-eval-workflow`

把“改写 case -> 用户确认 -> 去真实 agent 跑 -> 回来 judge -> 输出建议”串成完整流程。

## Project Structure

```text
user-feedback-eval-skills/
├─ .agents/
│  └─ skills/
│     ├─ user-feedback-trace-to-eval-case/
│     ├─ user-feedback-eval-judge/
│     └─ user-feedback-eval-workflow/
└─ README.md
```

## Quick Start

把这个包里的 `.agents` 目录放到你的 Codex 项目根目录下。

放好之后，目录结构大致如下：

```text
your-project/
├─ .agents/
│  └─ skills/
│     ├─ user-feedback-trace-to-eval-case/
│     ├─ user-feedback-eval-judge/
│     └─ user-feedback-eval-workflow/
└─ ...
```

然后在 Codex 新开对话，直接用 skill 名调用。

## Usage

### Rewrite a case

```text
$user-feedback-trace-to-eval-case

请把下面这个用户反馈 trace 改写成适合放进评测集的 case。
输出格式严格为：
input:
output:
context:

原始材料：
[把 trace / bad case / pasted text 贴进来]
```

### Run judge

```text
$user-feedback-eval-judge

请基于评估标准，对下面 case 打分。

改写后的 input：
[贴这里]

期望 output：
[贴这里]

实际 agent 输出：
[贴这里]
```

### Run the full workflow

```text
$user-feedback-eval-workflow

我有一个用户反馈 bad case，先帮我改写成评测集 case，不要直接 judge。

原始材料：
[把 trace / pasted text 贴进来]
```

## Recommended Workflow

为了减少上下文污染，比较推荐拆成 3 个独立对话：

1. 用 `$user-feedback-trace-to-eval-case` 改写 case
2. 去真实 agent / CMS 跑这条 case，拿回实际输出
3. 用 `$user-feedback-eval-judge` 做评分和归因

如果条件允许，改写case、judge打分、分析建议这三个环节，尽量不要共用同一个模型，也不要放在同一个上下文窗口里。

## Minimum Inputs

### For case rewrite

至少提供下面任意一种：

- 原始trace
- 粘贴出来的bad case片段
- 直接相关的agent输出

### For judge

至少提供：

- 改写后的 `input`
- 期望的 `output`（尽量具体可执行）
- 实际agent输出

如果还能提供 trace evidence、tool results、模型信息，judge 的判断会更完整；暂时没有这些材料，也可以先跑。

## Best Fit Scenarios

这套包比较适合下面这些case：

- 从零创建世界和故事大纲
- 基于既有故事生成角色资产prompt
- 生成场景资产prompt
- 生成生物资产prompt
- 生成分镜文本
- 修改世界观/大纲/角色/场景/分镜prompt
- 检查多轮上下文是否延续正确
- 检查资源受限时是否真实降级，是否如实说明完成状态
- 是否正确调用相关tool/skill
- 是否执行相对可靠的工作路径
- 是否遵循指令一致性

## Notes

- 这套skills默认尊重大多数agent的真实做事方式：先询问/查询，再规划执行
- 它很适合评测中间产物，不需要每条case都落到最终视频，尤其适合复杂长视频的制作过程
- 它会按照给定信息，尽量补齐最小所需的缺失上下文，让case更容易复现
- 它会区分过程里的失败调用、真实失败、合理降级和虚假完成表述

## Contributing

如果你想继续往里补case，建议按下面这个顺序来：

1. 查找真实出现过的用户反馈或bad case。
2. 优先保留原问题的核心矛盾，比如任务理解偏差、上下文丢失、过度追问、资源受限后的降级处理。
3. 改写成可执行的 `input / output / context`，尽量补齐最小必要前置条件。
4. 如果有实际agent输出，可以用这个skill进行二次judge，校对表现情况。

如果只是想先补素材，也可以先把原始trace整理进来，后面再慢慢改写。

## Roadmap Ideas

如果你们后续继续往里补案例，可以优先增加：

- 更多真实few-shot bad cases
- 不同case类型的改写模板
- 更细的judge 示例
- 团队内部常用case示例文档

## Summary

这套仓库适合拿来做 **生成视频类AI Agent 的用户反馈回归评测**，方便团队把真实问题整理下来，反复复现，也方便后续持续补case。
