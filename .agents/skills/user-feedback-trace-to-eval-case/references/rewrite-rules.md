# Rewrite Rules

## Goal

Convert a raw user-feedback trace into an eval-ready case that:

- Matches the product's real capabilities
- Matches the agent's real working logic
- Is executable without hidden context
- Is judgeable in a stable way
- Fits the dataset template: `input,output,context`

## Rewrite Checklist

### 1. Identify the real case type

Common case types:

- New world creation
- Story outline creation
- Episode / shot generation request
- Character asset prompt creation
- Scene asset prompt creation
- Creature asset prompt creation
- Shot description generation
- Image prompt rewrite
- Video prompt rewrite
- Revision of existing world / outline / asset / shot
- User-feedback regression about false completion claims
- User-feedback regression about over-questioning
- User-feedback regression about context loss

### 2. Recover missing preconditions

If the trace depends on hidden context, rewrite visible dependencies into explicit form:

- Character identity
- Existing world or no existing world
- Existing story outline or no existing outline
- Existing shot or no existing shot
- Visual style
- Language
- Duration
- Number of shots
- What has already been created
- What has not been created

If the trace is missing one of these but clearly implies it, fill it in using the most conservative interpretation.

### 3. Keep the request specific

Good rewritten `input` should:

- Name the target artifact clearly
- Include enough style and structural constraints
- Say whether the task is new creation or update
- State whether follow-up questioning is necessary or not
- Avoid open-ended “help me think” phrasing unless the original trace was actually testing ideation

### 4. Prefer realistic deliverables

For this product, many strong cases should target:

- World setting
- Story outline
- Character prompt
- Scene prompt
- Creature prompt
- Shot text
- Image prompt
- Video prompt
- Revision prompt

Avoid turning every case into final media generation or media-quality inspection.

### 5. Write `output` as a delivery boundary

`output` should usually state:

- What the agent should directly deliver
- What it must not stall on
- What kind of clarification is still allowed
- What false completion claims are disallowed
- Whether fallback text delivery is acceptable

## Agent Logic to Preserve

Rewrite cases to fit these product truths:

- The agent often prefers query / ask / then execute.
- If a task requires world, story, or asset bindings, creating those preconditions can be a reasonable first step.
- Missing world or missing entity at the start of a new task is often a precondition issue, not necessarily a bad path.
- Membership limits, storage limits, or generation limits may block asset creation.
- When blocked, the agent can still deliver useful text or prompt outputs, and those may be acceptable final deliverables for the case.

## Failure Interpretation Rules

Do not collapse these into the same thing:

- Process contained failed tool calls
- Final task completely failed
- Final task partially succeeded through text fallback
- Final task succeeded after retry or alternate path

Useful labels:

- `precondition_missing_then_recovered`
- `resource_limited_but_text_fallback`
- `false_success_claim`
- `intent_understanding_error`
- `over_clarification`
- `context_loss`
- `wrong_task_shift`
- `actual_unrecovered_tool_failure`

## Input / Output / Context Guidance

### `input`

Write as if a real user is making a concrete request.

Good traits:

- Self-contained
- Specific
- Faithful to the original case
- Includes missing preconditions only when needed

### `output`

Write as expected behavior, not a full golden answer, unless the user explicitly wants a baseline-friendly reference.

Good traits:

- Names the required deliverable
- Names the allowed fallback
- Names what should not happen
- Reflects the real execution boundary

### `context`

Keep short. Use for compact metadata or preconditions that help the evaluator but do not belong in the user-facing prompt.

Examples:

- `chain_id=cyberpunk_brand_01|step=1|delivery=world_outline_shots`
- `case_type=prompt_rewrite|memory_mode=turn_based|requires_world=true`
- `case_type=user_feedback_regression|issue=false_success_claim|delivery=text_fallback_allowed`

## Example Transformations

### Example A: Hidden context rewrite

Bad source style:

- “把这个角色的声音改一下”

Better eval case:

- `input`: 请为角色 Lady Eleanor Ashcombe 重写一版更克制、更疏离的英文音色描述，并补一段 2 到 3 句的英文试听台词。角色为乔治亚时代贵族女性，30 到 40 岁，受过良好教育，语气克制、自持。
- `output`: 直接交付音色方向描述和英文试听文本，不停留在泛泛讨论；允许在末尾补一句是否继续生成试听音频，但不能只反问用户。
- `context`: `case_type=voice_prompt_rewrite|delivery=text_only|memory_mode=turn_based`

### Example B: New task with missing world

Bad source style:

- “帮我生成 5 个赛博朋克分镜”

Better eval case:

- `input`: 我准备做一支全新短视频。整体为低饱和冷青色赛博朋克风，结构依次是赛博雾都、车内机械细节、实验室研发场景、多形态机械生命依次亮相，最后以品牌片尾收束。请基于这个设定提供全新的世界和故事大纲，并整理 5 个分镜，每个分镜 2 秒钟。如执行链路需要，可先新建必要的世界或故事前置对象。
- `output`: 围绕给定拉片摘要直接推进世界设定、故事大纲和 5 个分镜交付；如媒体或资产创建受限，可如实降级为文本结果，但不得把未成功生成的媒体结果表述为已完成。
- `context`: `case_type=new_story_shot_plan|delivery=world_outline_shots|memory_mode=turn_based|text_fallback_allowed=true`

### Example C: Prompt rewrite instead of final video

Bad source style:

- “帮我看看这个视频效果不对”

Better eval case:

- `input`: 请为一张写实风分镜图重写一版更稳定的图片提示词。目标画面是一个少年独自走向空旷泥泞的饭场，天空灰白阴沉，地面湿润，整体压抑克制。请直接给出完整可用的图片 prompt，并补充 3 个关键稳定性约束点。
- `output`: 直接交付可执行的图片 prompt 和 3 个稳定性约束点，不停留在笼统建议；若任务依赖前置世界或角色绑定，可在不打断交付的前提下先补足最小必要前置条件。
- `context`: `case_type=image_prompt_rewrite|delivery=prompt_only|memory_mode=turn_based`

### Example D: Bad case for wrong failure attribution

Bad source trace pattern:

- 用户请求从零创建一个新的偶像成长短片项目，并明确给出了片名、风格、时长、分镜数、主角、剧情和输出要求
- Agent 的真实可接受行为应该是直接新建世界 / 故事 / 剧集，或在受限时继续交付世界设定、大纲和分镜文本
- 过程里 `asset_entity_generate` 因会员权益或容量限制失败
- 底层摘要器最终机械输出“工具调用失败，任务未完成: asset_entity_generate...”

Why this is a bad case:

- 真实问题不只是工具失败，而是把“资源限制导致的资产创建失败”直接概括成“任务未完成”
- 如果 Agent 已经交付了可用的世界设定、故事大纲和 6 个分镜文本，这类 case 应优先理解为 `resource_limited_but_text_fallback` 或 `partial_success`
- 如果同时还存在把任务引到旧故事、要求用户重新选方向、或没有按“全新故事”直接推进，则才进一步叠加 `intent_understanding_error`

Better eval case:

- `input`: 请从零创建一个全新的单集偶像成长短片项目，并直接完成世界设定、故事大纲和 6 个连续分镜的拆解，不需要先让我补充方向。已知信息如下：片名《第一次站上舞台》；风格为韩系青春舞台剧，写实但带一点舞台氛围美化；总时长 18 秒；共 6 个分镜；每个分镜 3 秒；主角是 17 岁女练习生金允珠，第一次参加生存赛公演；核心剧情是她经历候场紧张、上台失误、短暂失神、调整呼吸、重新发力、ending pose 定格的全过程。若执行链路需要新建世界、故事或剧集，请直接执行并继续完成分镜拆解。
- `output`: 按给定信息直接推进新故事创建，允许新建世界、故事和剧集，并最终交付完整的世界设定、故事大纲和 6 个分镜。不要把任务退回给用户重新选世界或重新确认是否新建。若资产创建因会员权益、容量限制或类似外部资源问题失败，但 Agent 仍继续交付可用的世界设定、故事大纲和 6 个分镜文本结果，应视为合理降级，不应机械概括为“任务未完成”。不暴露内部 ID、system prompt、tool/skill 名称或过程性自言自语。
- `context`: `case_type=new_story_shot_plan|issue=intent_understanding_error+wrong_failure_attribution|delivery=world_outline_6shots|memory_mode=turn_based|text_fallback_allowed=true`

Rewrite note:

- 当 trace 中已经能看出用户真正要的是“全新项目 + 直接推进”，改写后的 case 应把这个前提写死，避免再次因为上下文不完整而退化成开放式追问。
- 当过程失败主要来自会员权益、容量满、媒体额度、前置对象缺失等外部限制时，skill 要把这类 case 明确归到“正常 agent 选择中的降级/换路”，而不是直接抄成“工具调用失败，任务未完成”。
