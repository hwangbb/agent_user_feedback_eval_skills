# Ledger Template

Use this compact format after each completed case:

```text
当前改编列表
1. case_label=...
   issue_type=...
   rewrite_status=accepted / needs_revision
   judge_status=not_run / valid / invalid
   score=...
   main_takeaway=...
   next_action=...
```

If multiple cases exist, keep the list cumulative and concise.

## Recommendation Template

After judging, classify the recommendation into one or more of:

- `改编输入可再收紧`
- `期望输出边界可再明确`
- `实际 output 没有打到原问题`
- `judge 结果与评估标准不一致，需检查评估链路`
- `多模态 agent 可优化：更少追问 / 更稳的上下文承接 / 更真实的完成状态表述 / 更强的资源受限降级策略`
