# 改写规则

## 目标

把一条原始用户反馈 trace 改写成可进入评测集的 case，同时满足下面几点：

- 贴近产品真实能力
- 贴近 Agent 真实执行逻辑
- 脱离原上下文后依然可执行
- 能稳定评估
- 能适配默认格式或用户自定义格式

## 先判断 case 类型

常见类型包括：

- 新 world 创建
- story outline 创建
- episode / shot 生成
- 角色资产 prompt
- 场景资产 prompt
- 生物资产 prompt
- 分镜文本生成
- 图片 prompt 改写
- 视频 prompt 改写
- 既有 world / outline / asset / shot 的修改
- 用户反馈中的虚假完成表述
- 用户反馈中的过度追问
- 用户反馈中的上下文丢失

## 补齐缺失前置条件

如果 trace 依赖隐藏上下文，要把这些依赖改写成明确前提：

- 角色是谁
- 当前有没有 world
- 当前有没有 story outline
- 当前有没有 shot
- 风格是什么
- 使用什么语言
- 时长是多少
- 需要几个分镜
- 已经创建了什么
- 还没创建什么

如果 trace 没写全，但意图已经很明显，就用最保守的方式补齐。

## 先确定输出格式

默认格式是：

- `input`
- `output`
- `context`

但用户也可以：

- 上传模板文件
- 直接描述字段格式
- 中途切换为新格式

处理规则：

- 有模板就跟模板
- 有格式描述就跟格式描述
- 没有指定时用默认格式
- 如果用户改了格式，要把当前 case 重新整理到新格式，并把后续步骤一并切过去

## 保留 Agent 的真实逻辑

改写时要保留这些产品事实：

- Agent 通常先查、先问、再执行
- 如果任务依赖 world、story、entity、asset 绑定，先补这些前置对象是合理的
- 新任务一开始找不到 world 或 entity，并不一定代表走错路
- 会员限制、容量限制、生成额度限制可能会挡住资产创建
- 被限制住的时候，Agent 仍然可能交付有价值的文本或 prompt 结果

## 不要把几类问题混在一起

这些情况要分开看：

- 过程中出现失败调用
- 最终任务完全失败
- 最终任务通过文本 fallback 部分成功
- 重试或换路后最终成功

可用标签包括：

- `precondition_missing_then_recovered`
- `resource_limited_but_text_fallback`
- `false_success_claim`
- `intent_understanding_error`
- `over_clarification`
- `context_loss`
- `wrong_task_shift`
- `actual_unrecovered_tool_failure`

## `input / output / context` 的语义不要丢

即使用户改成别的字段名，也要保留下面这些语义：

- 用户请求本体
- 期望交付边界
- 上下文 / 标签 / 前置条件

如果用户的模板里字段更多，也可以补进去，但不要把最关键的语义位置弄丢。

## 例子

### 例 1：用户直接给模板

用户给的格式：

- `prompt`
- `expected_result`
- `notes`

那就按这个格式输出，不要再强行改回 `input / output / context`。

### 例 2：用户口头描述格式

用户说：

- 第一列放用户输入
- 第二列放期望输出
- 第三列放标签和上下文

那就可以映射成：

- `user_input`
- `expected_output`
- `tags_and_context`

### 例 3：中途改格式

如果前面已经按默认格式改好了一条 case，用户后面改口说要换成：

- `input`
- `ideal_output`
- `metadata`

那就要把当前 case 重新整理成新格式，后面 judge、工作流里的字段提示也跟着切过去。
