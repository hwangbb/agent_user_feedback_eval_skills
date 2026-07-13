# 工作流 Playbook

## Stage A：收材料

目标：

- 确认原始问题到底想测什么
- 判断目前材料够不够改写成 case
- 确认用户要用什么评测集格式

格式来源可以是：

- 用户上传模板
- 用户描述字段
- 默认 `input / output / context`

## Stage B：改写 case

产出：

- 用户当前格式下的 case

同时说明：

- 原问题保留了什么
- 上下文补了什么
- 当前格式是什么
- 如果是自定义格式，字段映射是什么

## Stage C：用户确认

只问一个短问题，确认：

- 当前内容是否可以
- 当前格式是否可以

如果用户要改格式，就先重排 case，再继续。

## Stage D：准备 Judge

不要在用户确认 case 后立刻 judge。

先让用户去真实 Agent / CMS 跑一遍，再把实际输出带回来。

如果用户用了自定义格式，收实际输出时也沿用这个格式的字段口径。

## Stage E：执行 Judge

Judge 时看语义，不死盯字段名。

默认格式直接 judge；
自定义格式先映射字段，再 judge。

## Stage F：归因

最后分析问题究竟更像是：

- Agent 行为问题
- case 改写问题
- 期望输出边界问题
- judge 标准或链路问题

## Stage G：列表维护

每条 case 至少记录：

- `case_label`
- `issue_type`
- `case_format`
- `rewrite_status`
- `judge_status`
- `score`
- `main_takeaway`
- `next_action`
