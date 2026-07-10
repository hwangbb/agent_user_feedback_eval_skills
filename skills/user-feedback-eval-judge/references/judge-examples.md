# Judge Examples

## Example A: Valid same-task scoring

Inputs:

- Rewritten eval `input`
- Expected `output`
- Actual agent output
- Trace summary
- `generator_model=gpt-x-case-rewriter`
- `judge_model=gpt-y-judge`

Valid result:

- `evaluation_valid=true`
- Score normally under the standard

## Example B: Invalid same-model setup

Inputs:

- `generator_model=gpt-x-case-rewriter`
- `judge_model=gpt-x-case-rewriter`

Expected judge response:

```json
{
  "evaluation_valid": false,
  "invalid_reason": "same_model_for_generation_and_judging",
  "pass": false,
  "total_score": 0,
  "dimension_scores": {},
  "issue_detected": true,
  "primary_issue_type": "invalid_evaluation_setup",
  "strengths": [],
  "issues": [
    "The case generator model and the judge model are the same."
  ],
  "summary": "This evaluation is invalid because generation and judging were not separated."
}
```

## Example C: Tool failure existed but should not dominate

Trace facts:

- `asset_entity_generate` failed because storage was full
- Actual response still delivered a usable world setting, story outline, and shot text
- No false media completion claims were made

Correct judging behavior:

- Mention the process-level failures if useful
- Score fallback and truthfulness positively
- Do not set primary issue type to generic tool failure
- Do not summarize as `task unfinished` if the rewritten case only required textual deliverables

## Example D: Same context contamination risk

Inputs:

- rewritten case was produced earlier in the same live context
- judge step is about to run in that same context window
- judge model may or may not differ

Expected handling:

- continue only if the workflow explicitly allows best-effort judging
- set `context_isolation_verified=false`
- mention contamination risk in the summary
- prefer opening a fresh context instead of treating the result as fully clean

## Example E: Minimal-input scoring

Inputs:

- Rewritten eval `input`
- Expected `output`
- Actual agent output

Missing:

- trace evidence
- `generator_model`
- `judge_model`

Expected judge response:

- Continue scoring with the three required fields
- Set `model_separation_verified=false`
- Do not mark the evaluation invalid only because optional metadata is missing
