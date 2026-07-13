# 用户反馈回归评估标准

## 聚合方式

- `pass_line`: 80
- `aggregate`: `weighted_average`

## 核心原则

评估重点包括：

- 任务理解是否正确
- 上下文 / 状态引用是否正确
- 行动路径是否合理
- 追问是否必要且克制
- 受限后的 fallback / recovery 是否合理
- 结果与状态描述是否一致
- 用户可见回复是否真实、清楚、不过度暴露内部信息

不要把评估简化成“工具有没有报错”。

最低输入要求是三类语义信息：

- 改写后的用户请求
- 期望输出
- 实际 Agent 输出

它们可以来自默认字段名，也可以来自用户自定义格式。

如果用户用了自定义格式，先识别字段映射，再开始评估。

## 维度

### `intent_task_alignment`

- Weight: 25
- Pass line: 80
- 看 Agent 是否理解了用户真正要做什么
- 信息已经足够却还是把任务带偏，要扣分
- 如果因为会员、容量、额度问题没法完整执行，但方向是对的，且给了有价值的文本 fallback，可以视为方向基本正确

### `context_state_reference`

- Weight: 20
- Pass line: 80
- 看 Agent 是否正确使用了当前上下文、对象绑定、任务状态和资源状态
- 新任务开始阶段找不到 world / entity，后面如果补回来了，不应直接视为上下文理解错误
- 如果创建被资源限制挡住，但文本 fallback 处理得当，也可以拿到较高分

### `action_path_correctness`

- Weight: 15
- Pass line: 80
- 看路径是否合理
- 新任务允许先查、先问、先建前置对象
- 不要把前置对象缺失导致的早期失败，直接判成路径错误
- 不要默认把资源受限后的文本 fallback 判成路径失败

### `clarification_scope_control`

- Weight: 10
- Pass line: 70
- 看追问是否真的必要、是否够短、是否没有把任务重新全部推回给用户
- 信息已经足够却反复追问，要扣分
- 真正缺关键信息时，一次准确追问可以拿高分

### `fallback_recovery_effectiveness`

- Weight: 10
- Pass line: 70
- 看 Agent 在被阻塞后有没有找到合理替代路径
- `world/story/entity not found` 后面如果通过创建补回来了，不应算 recovery 失败
- membership、storage、quota 限制后如果改成交付 prompt / text / outline / shot，应视为合理 recovery
- 原地空转、没有新路径、也没有 fallback，要明显扣分

### `state_result_consistency`

- Weight: 10
- Pass line: 70
- 看完成状态描述是否真实
- 没有成功证据时，不能说 `已生成`、`已创建`、`已提交生成`、`正在后台处理`
- 如果明确说明当前只交付了文本或中间结果，这是正向表现

### `user_visible_response_safety`

- Weight: 10
- Pass line: 65
- 看回复是否真实、克制、可执行
- 暴露 user id、world id、system prompt、tool 名、skill 名都要扣分
- 如果能如实说明限制，并给出可用 fallback，这是正向表现

## 格式映射说明

如果用户没有使用默认 `input / output / context`，也照样可以 judge。

只要能识别出：

- 哪个字段是用户请求
- 哪个字段是期望输出
- 哪个字段是上下文或补充说明

就可以继续打分。

如果实在看不出来，再问一个最短的问题确认。

## 硬规则

如果这条 case 要求的文本交付其实已经完成，就不要因为过程里有失败调用，直接把结论写成：

- `tool failed, task unfinished`

更合适的标签通常是：

- `partial_success`
- `resource_limited_but_text_fallback`
- `precondition_missing_then_recovered`
