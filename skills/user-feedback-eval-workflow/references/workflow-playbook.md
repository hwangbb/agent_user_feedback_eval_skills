# Workflow Playbook

## Stage A: Raw Trace Intake

Goal:

- Identify the real issue the user wants to test
- Decide whether the provided material is enough to rewrite a case

If the trace is partial:

- Use only the directly relevant span or output
- Reconstruct minimal missing context only when needed for execution

## Isolation Rule

This workflow has three logically separate phases:

- rewrite
- judge
- analyze / recommend

They should not reuse the same LLM in the same context window.

Best practice:

- rewrite in context A
- judge in fresh context B
- analyze in fresh context C

If only model identity changes but the same context window is reused, treat the separation as insufficient.

## Stage B: Case Rewrite

Produce:

- `input`
- `output`
- `context`

Then summarize:

- preserved original issue
- reconstructed preconditions
- why this version is more executable and judgeable

## Stage C: User Confirmation

Ask one short question:

- “这个改编版可以直接进评测集，还是你想再改 input / output / 两者？”

If the user asks for revision:

- revise only the requested part
- keep the preserved-test-point summary updated

## Stage D: Judge Preparation

Do not ask to judge immediately after the rewrite is accepted.

Instead:

- tell the user to run the rewritten case in the real agent / CMS
- wait for the user to paste back the actual output

After the actual output returns, ask one short question:

- “要不要继续放进 LLM Judge 跑一遍？”

If yes:

- collect actual agent output
- collect trace evidence if available
- collect generator/judge model identifiers if available

## Stage E: Judge Execution

Judge using:

- rewritten `input`
- expected `output`
- actual agent output
- optional trace evidence

Return:

- validity
- score
- pass/fail
- per-dimension breakdown
- primary issue type
- summary

Record whether the judge context is fresh relative to the rewrite context.

## Stage F: Meta Analysis

Always analyze whether the weak point is:

- the actual agent output
- the rewritten case
- the expected output contract
- the judge rubric
- the lower-level pipeline summary

Prefer doing this in a third fresh context. If not possible, explicitly warn that recommendation quality may be influenced by prior context memory.

## Stage G: Running Ledger

Maintain a short list entry for every case processed in the current conversation.

Minimum fields:

- `case_label`
- `issue_type`
- `rewrite_status`
- `judge_status`
- `score`
- `main_takeaway`
- `next_action`
