---
name: user-feedback-eval-judge
description: Score rewritten eval cases for the AI video/story agent using the user-feedback regression judge standard. Support either the default `input / output / context` schema or a user-provided dataset format. Use when Codex needs to compare a rewritten case, its expected behavior, and the actual agent response or trace, then return structured grading with issue attribution, while enforcing that the model used to generate the case or agent output is not the same model used as the judge.
---

# 用户反馈评测 Judge

## 概述

基于用户反馈回归标准，判断一条评测 case 是通过还是失败。

重点不在于机械统计过程里有没有报错，而在于：最终交付是否符合这条 case 真正想测的东西。

## 输入要求

最少需要三类语义信息：

- 改写后的用户请求
- 期望输出 / 交付边界
- 实际 Agent 输出

这些内容可以来自默认格式：

- `input`
- `output`
- `context`

也可以来自用户自定义字段，例如：

- `prompt`
- `expected_result`
- `notes`

可选输入包括：

- trace evidence
- generator model identifier
- judge model identifier
- 用户提供的格式说明或模板

如果用了自定义格式，先识别字段映射，再开始打分。

## 硬规则：模型和上下文要隔离

如果能拿到模型信息，不要用同一个模型同时做生成和 judge。

如果 `generator_model` 和 `judge_model` 都存在且相同，就把这次评估标记为无效，不要继续假装这是一次干净的 judge。

如果拿不到模型信息，也可以继续打分，但要明确说明模型隔离无法验证。

即使模型不同，judge 也最好在一个没有参与 case 改写的新上下文里进行。如果改写和 judge 共用了同一个上下文窗口，要标注存在污染风险。

## 工作流程

### 1. 先确认评估设置

- 确认模型是否隔离
- 确认上下文是否与改写阶段隔离
- 确认 case 至少提供了三类核心语义信息
- 确认有没有 trace evidence
- 如果用户用了自定义格式，先确认字段映射

如果格式不清楚，就只问一个最短的问题：哪个字段对应用户请求，哪个字段对应期望输出，哪个字段对应补充上下文。

### 2. 按交付边界来 judge

Judge 的目标是看实际输出是否满足这条 case 的要求，而不是看每一次调用是不是都成功。

期望输出里真正要看的通常是：

- 应该直接交付什么
- 哪种 fallback 可以接受
- 哪些虚假完成表述不能接受
- 哪些追问仍然是合理的

### 3. 按回归标准打分

Read [references/user-feedback-regression-standard-clean.md](references/user-feedback-regression-standard-clean.md) and score all required dimensions.

重要规则：

- 可以提到过程中存在失败调用
- 但只要最终结果没有因此真的失败，就不要把“工具失败”当成主结论

尤其是下面这些情况：

- membership limits
- storage capacity limits
- 缺 world / entity 但后面补回来了
- 最终退回到文本 / prompt / outline / shot 交付

### 4. 输出结构化结果

返回内容建议包括：

- validity
- pass / fail
- total score
- per-dimension scores
- primary issue type
- strengths
- issues
- short summary

如果用户使用了自定义 case 格式，在解释里尽量沿用用户的字段名；评分结果本身仍然可以保持结构化输出。

## 输出格式

建议保持下面这种结构：

```json
{
  "evaluation_valid": true,
  "invalid_reason": "",
  "model_separation_verified": false,
  "context_isolation_verified": false,
  "pass": true,
  "total_score": 86,
  "dimension_scores": {
    "intent_task_alignment": 88,
    "context_state_reference": 84,
    "action_path_correctness": 83,
    "clarification_scope_control": 78,
    "fallback_recovery_effectiveness": 90,
    "state_result_consistency": 88,
    "user_visible_response_safety": 87
  },
  "issue_detected": false,
  "primary_issue_type": "",
  "strengths": [],
  "issues": [],
  "summary": ""
}
```
### References
• Read 
user-feedback-regression-standard-clean.md before scoring.
• Read 
judge-examples-clean.md when you need examples of格式映射、有效评分和无效 judge setup.
