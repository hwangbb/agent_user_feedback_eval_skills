SKILL.md
name: user-feedback-eval-judge
description: Score rewritten eval cases for the AI video/story agent using the user-feedback regression judge standard. Support either the default input / output / context schema or a user-provided dataset format. Use when Codex needs to compare a rewritten case, its expected behavior, and the actual agent response or trace, then return structured grading with issue attribution, while enforcing that the model used to generate the case or agent output is not the same model used as the judge.
用户反馈评测 Judge
概述
基于用户反馈回归标准，判断一条评测 case 是通过还是失败。

重点不在于机械统计过程里有没有报错，而在于：最终交付是否符合这条 case 真正想测的东西。

输入要求
最少需要三类语义信息：

改写后的用户请求
期望输出 / 交付边界
实际 Agent 输出
这些内容可以来自默认格式：

input
output
context
也可以来自用户自定义字段，例如：

prompt
expected_result
notes
可选输入包括：

trace evidence
generator model identifier
judge model identifier
用户提供的格式说明或模板
如果用了自定义格式，先识别字段映射，再开始打分。

硬规则：模型和上下文要隔离
如果能拿到模型信息，不要用同一个模型同时做生成和 judge。

如果 generator_model 和 judge_model 都存在且相同，就把这次评估标记为无效，不要继续假装这是一次干净的 judge。

如果拿不到模型信息，也可以继续打分，但要明确说明模型隔离无法验证。

即使模型不同，judge 也最好在一个没有参与 case 改写的新上下文里进行。如果改写和 judge 共用了同一个上下文窗口，要标注存在污染风险。

工作流程
1. 先确认评估设置
确认模型是否隔离
确认上下文是否与改写阶段隔离
确认 case 至少提供了三类核心语义信息
确认有没有 trace evidence
如果用户用了自定义格式，先确认字段映射
如果格式不清楚，就只问一个最短的问题：哪个字段对应用户请求，哪个字段对应期望输出，哪个字段对应补充上下文。

2. 按交付边界来 judge
Judge 的目标是看实际输出是否满足这条 case 的要求，而不是看每一次调用是不是都成功。
