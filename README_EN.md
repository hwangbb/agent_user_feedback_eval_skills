# User Feedback Eval Skills for Codex

A set of Codex skills for regression testing AI video and story agents from real user feedback.

This repo is built for bad cases that already showed up in production, in the agent itself, or in an internal management system such as a CMS. The idea is simple: turn a raw trace into a stable eval case, run the actual agent output back through the workflow, and use that result for judging, issue attribution, and follow-up fixes.

## Why

Real user feedback is useful, but it is usually messy in ways that make it hard to turn into a reliable eval set:

- the original context is missing or too large to carry over cleanly
- there are too many references like "this character" or "that shot" to make sense of outside the original trace
- the task is too open-ended to judge consistently
- there may be failed calls in the middle even though the final result is still usable
- useful bad cases are collected over time, but rarely end up in one clean regression set

These skills are meant to make that process easier and more repeatable.

## Features

- Rewrite raw traces, bad cases, and execution logs into `input / output / context`
- Fill in the minimum missing context so a case can actually run
- Preserve the way most agents really work: query first, ask when needed, then execute
- Support workflows that depend on creating world, story, or entity assets before downstream steps
- Focus on intermediate outputs that matter in real production flows, such as worldbuilding, outlines, prompts, shot text, and revisions
- Judge final behavior based on what was actually delivered, not just whether some call failed along the way
- Keep rewrite, rerun, judging, and follow-up suggestions in one workflow

## Included Skills

### `user-feedback-trace-to-eval-case`

Rewrites a raw trace, bad case, or pasted execution log into the common eval format: `input`, `output`, and `context`.

### `user-feedback-eval-judge`

Scores a rewritten case against the actual agent output and returns structured grading plus issue attribution.

### `user-feedback-eval-workflow`

Connects the full loop: rewrite the case, confirm it, run it in the real agent, judge the result, and summarize what to fix.

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

Copy the `.agents` directory from this package into the root of your Codex project.

Your project should then look roughly like this:

```text
your-project/
├─ .agents/
│  └─ skills/
│     ├─ user-feedback-trace-to-eval-case/
│     ├─ user-feedback-eval-judge/
│     └─ user-feedback-eval-workflow/
└─ ...
```

After that, start a new conversation in Codex and call the skill by name.

## Usage

### Rewrite a case

```text
$user-feedback-trace-to-eval-case

Please rewrite the following user-feedback trace into an eval-ready case.
Use this output format:
input:
output:
context:

Source material:
[paste the trace / bad case / pasted text here]
```

### Run judge

```text
$user-feedback-eval-judge

Please score the following case using the eval standard.

Rewritten input:
[paste here]

Expected output:
[paste here]

Actual agent output:
[paste here]
```

### Run the full workflow

```text
$user-feedback-eval-workflow

I have a real user-feedback bad case. First, rewrite it into an eval-ready case. Do not run judge yet.

Source material:
[paste the trace / pasted text here]
```

## Recommended Workflow

To reduce context bleed, it works best as three separate conversations:

1. Use `$user-feedback-trace-to-eval-case` to rewrite the case
2. Run that case in the real agent or CMS and collect the actual output
3. Use `$user-feedback-eval-judge` to score the result and explain what happened

If you can separate roles further, keep case rewriting, judging, and follow-up analysis out of the same model context.

## Minimum Inputs

### For case rewrite

Provide at least one of the following:

- the raw trace
- a pasted bad-case excerpt
- the directly relevant agent output

### For judge

Provide at least:

- rewritten `input`
- expected `output`
- actual agent output

If you also have trace evidence, tool results, or model metadata, the judgment will be more complete. If not, you can still run a first pass.

## Best Fit Scenarios

This repo is a good fit for cases like:

- creating a new world and story outline from scratch
- generating character asset prompts from an existing story
- generating scene asset prompts
- generating creature asset prompts
- generating shot text
- revising world, outline, character, scene, or shot prompts
- checking whether multi-turn context carries forward correctly
- checking whether a resource-limited run falls back honestly and clearly
- checking whether the right tool or skill was used
- checking whether the agent followed a reasonably reliable path
- checking instruction-following consistency

## Notes

- These skills assume a fairly common agent pattern: look things up first, ask when needed, then execute
- They are especially useful for intermediate outputs, so not every case needs to end at final video generation
- They try to reconstruct just enough missing context to make a case runnable again
- They distinguish between failed calls, true task failure, reasonable fallback behavior, and false completion claims

## Contributing

If you want to add more cases, this order usually works well:

1. Start with a real user feedback case or a real bad case.
2. Keep the original failure mode intact, whether that is intent drift, context loss, over-questioning, unreliable tool choice, or poor fallback behavior.
3. Rewrite it into executable `input / output / context`, and add only the minimum missing setup.
4. If you have the actual agent output, run judge on it and check whether the rewritten case still captures the original issue.

If all you have for now is the raw trace, that is still useful. You can add the source material first and rewrite it later.

## Roadmap Ideas

Useful additions over time:

- more real few-shot bad cases
- rewrite templates for different case types
- more detailed judge examples
- team-specific example cases

## Summary

This repo is meant for teams working on AI video or story agents who want a cleaner way to turn real user feedback into reusable regression cases.
