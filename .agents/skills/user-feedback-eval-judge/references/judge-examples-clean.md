# Judge 示例

## 示例 1：默认格式

输入字段：

- `input`
- `output`
- `context`
- 实际 Agent 输出

这种情况下直接按标准 judge 即可。

## 示例 2：用户自定义格式

用户给的字段：

- `prompt`
- `expected_result`
- `notes`

这时先做字段映射：

- `prompt` -> 用户请求
- `expected_result` -> 期望输出
- `notes` -> 上下文

映射完成后再开始 judge。

## 示例 3：同模型 judge，无效

如果：

- `generator_model=gpt-x`
- `judge_model=gpt-x`

就要把这次评估标记为无效，而不是继续打分。

## 示例 4：过程有失败，但不能直接判任务失败

如果：

- `asset_entity_generate` 因容量限制失败
- 实际输出仍然交付了可用的 world、outline、shots
- 没有虚假完成表述

那就可以提过程里有失败，但主结论不该是“任务未完成”。
