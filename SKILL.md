---
name: tudou-single-h3
description: 为单个视频片段编写或修改可直接使用的 MiniMax H3 提示词。适用于用户提供剧情、动作、角色、场景、首帧或风格参考，需要一个片段的完整提示词；支持对白语气、声音标签、微表情、连续多情绪演绎，以及文戏的听者反应、接话节奏、行动动词、地位关系、调度、重音停顿与语速时间预算编排，支持纯文字需求和参考图，不限定题材。
---

# Tudou Single H3 · 单片段提示词

Turn the user's scene request and any supplied references into one ready-to-paste MiniMax H3 prompt for a single video clip. With reference images, use the reference-based workflow below; with text only, define subjects from the user's description without inventing uploaded references. Do not ask the user to pre-classify references; infer each image's role and preserve the intended identity, wardrobe, location, and visual style.

## Scope and defaults

- Deliver one clip prompt per request, not a full-film production package or multiple alternative clips unless requested. This skill writes prompts; it does not upload assets or spend generation credits.
- A single clip may contain cuts when the user requests them. Otherwise prefer one continuous shot with a clear action progression; do not impose a three-shot structure.
- Follow the requested genre, visual style, performance, and audio. Do not default to Japanese settings, winter, romance, prayer, photorealism, or music-video treatment.
- If timing is unspecified, use a six-second creative planning default and identify it as the assumed duration in `summary`, not as a claimed H3 platform limit. Do not invent supported resolutions, durations, or API parameters. Leave aspect ratio unspecified unless supplied or reasonably inferable.
- Ask only when essential missing input prevents a faithful result. If an explicitly required image is unavailable, request it rather than inventing its contents.

## Output contract

Output only one complete prompt using these six headings in this exact order:

```text
subject_definitions:
summary:
retention_analysis:
detailed_description:
overall_soundscape:
non_diegetic_music:
```

Use `<Subject 1>`, `<Subject 2>`, and onward consistently. Write the prompt in the user's requested language; for a Chinese-language request without a stated preference, write Chinese. Keep dialogue and visible text in the user's original language. Do not add a preface, explanation, prompt fragments, or a second alternative unless requested.

### Format hard rules (apply in every environment and every agent)

- Return the prompt inside **one** fenced code block (```text). Nothing else inside the block; outside the block write nothing, unless the user asked for explanation.
- The block starts with `subject_definitions:` and contains exactly the six headings above, each on its own line, in that order, spelled exactly, lower-case, followed by a colon. Never translate, rename, number, bold, or wrap them in Markdown (`#`, `**`, tables, bullet lists of headings).
- Under `detailed_description:`, write the global visual lock first, then shot blocks labeled `[Shot 1] 00:00.000–00:06.000` with sub-beats `00:00.000–00:01.200：…`. Times are continuous, start at `00:00.000`, end exactly at the total duration, and use three decimals.
- Each dialogue line appears once, on its own line, as `<Subject N> (SN): <d>[Language] …</d>`.
- If a section has nothing to add, write a short explicit statement (for example `non_diegetic_music: 无配乐。`), never leave it empty or delete it.

Skeleton (structure only; replace every part with the actual content):

```text
subject_definitions:
<Subject 1> (S1)：……
<Subject 2>：……
summary:
……
retention_analysis:
<Subject 1>：fully_preserved —— ……
detailed_description:
全局视觉锁定：……
[Shot 1] 00:00.000–00:06.000
景别与机位：……
00:00.000–00:01.200：……
<Subject 1> (S1): <d>[Chinese] ……</d>
00:01.200–00:06.000：……
overall_soundscape:
……
non_diegetic_music:
……
```

Use a compact production-prompt structure: subject definitions first, then a one-paragraph summary stating duration, format, story beat, and reference-role separation; then a per-subject retention analysis; then a global visual lock followed by timed `[Shot 1]`, `[Shot 2]`, and onward blocks.

## Reference-role inference

Classify every supplied image before writing:

- A character portrait, turnaround sheet, or outfit sheet becomes a fully preserved character subject.
- An environment, building, or location image becomes a fully preserved scene subject.
- A cinematic screenshot becomes an attribute-transfer-only style subject when separate images already establish the target characters or setting, or when the screenshot primarily communicates lighting, grading, lens quality, or atmosphere rather than a reusable subject.
- A composition, staging, pose, or action screenshot becomes an **extraction-only analysis source** when separate images already establish the target characters and setting. Inspect it, extract its camera angle, framing, spatial proportions, left/right placement, depth layers, and action geometry, then **exclude it from the final H3 reference-image upload set**.
- A supplied first-frame image becomes the visual starting state. Preserve its immediately visible people, positions, framing, wardrobe, and environment unless the requested action requires a later change.

Honor any roles named by the user over automatic inference. If a role is ambiguous, make the least disruptive assumption and state it in `retention_analysis`; do not pause for clarification. When character cards and scene assets coexist with a cinematic frame, favor treating that frame as a pure visual-style reference so it cannot overwrite the supplied identities, wardrobe, setting, or blocking.

## Composition-reference extraction rule

When the user provides a separate image only to communicate composition, person placement, pose, interaction, camera height, or action rhythm, do not let that image compete with the target identity references. A visually strong composition screenshot can otherwise cause H3 to copy its faces, wardrobe, and setting even when the prompt says otherwise.

1. Classify it internally as `extraction_only`; do not create a `<Subject N>` for it and do not mention it as a H3 reference image in the final prompt.
2. Translate its useful visual information into direct text constraints: camera angle and height; shot size and crop; frame percentages; screen-left/screen-right placement; foreground, middle-ground, and background layers; scale relationships; gaze lines; pose; hand placement; and action timing.
3. Explicitly bind identity, wardrobe, and environment only to the uploaded character and scene references. For example: `must use <Subject 2>'s male identity and <Subject 3>'s female identity`; never rely on a composition image to supply either person.
4. Exclude every recognizable element from the composition source: faces, hair, wardrobe, accessories, scene, architecture, lighting, grading, captions, and visible text. Preserve only the abstract spatial/action rule in words.
5. Exception: if the same image is the user's only source for a character, location, or required first frame, it is not extraction-only; classify it using the ordinary character, environment, or first-frame rules instead.

For a reference set containing scene + target character cards + a composition screenshot, the final H3 upload set normally contains only the scene and character cards. The composition screenshot stays outside H3; its extracted constraints appear only in `detailed_description`.

Number uploaded references in their final upload order after excluding extraction-only sources, and use that mapping consistently. For text-only prompts, `<Subject N>` identifies a described subject, not an uploaded image; state that no reference images were supplied in `retention_analysis`.

For each subject, write an exact, visible description. Include identity-defining facial and hair traits, complete wardrobe, accessories, and left/right placement for characters. Include key architecture, terrain, vegetation, weather, and foreground elements for scenes.

## Style-reference control

When an image is a style-only reference, define it exactly as an attribute transfer, never as source content:

```text
<Subject N> is reference image N used solely as a visual-style reference. Transfer only its artistic direction, cinematography, lens character, color palette, saturation, color-temperature relationship, exposure, contrast, lighting logic, material rendering, depth of field, bokeh texture, grain, highlight roll-off, shadow density, atmospheric perspective, and overall mood. Do not transfer any recognizable people, faces, hairstyle, wardrobe, objects, architecture, text, composition, camera angle, pose, or action from <Subject N>.
```

In `retention_analysis`, label that subject `attribute_transfer`; label characters and locations `fully_preserved`.

In `detailed_description`, begin with a direct separation between content and style. State that the style reference controls **how the video is photographed and graded**, while character, wardrobe, scene, blocking, and action come only from the fully preserved subjects. Then use this adaptable continuity lock. Keep weather or particles only when the requested scene needs them:

```text
All shots must continuously retain <Subject N>'s overall color system, color-temperature relationship, luminance distribution, contrast, key-light direction, light softness, environmental bounce logic, material detail, focal-length feel, depth-of-field relationship, bokeh behavior, grain scale, highlight roll-off, shadow density, atmospheric perspective, and artistic mood. Cuts must not cause a jump in grading, lighting ratio, skin tone, sharpness, grain, bokeh, or lens texture. Preserve only scene-appropriate weather and particles; when they are absent, maintain depth through natural foreground layers, atmosphere, spatial perspective, and out-of-focus highlights.
```

A visual-style reference is optional. If none is supplied, follow the user's written style instructions and derive restrained continuity constraints from available content references. If neither provides a style, use neutral visual language without inventing a genre-specific direction. Apply style-reference separation only when such a reference exists.

## Scene and performance workflow

1. Convert the user's story into a timed, shot-by-shot sequence. Respect all stated durations, hard cuts, camera distance, screen positions, and audio constraints. Use `00:00.000` timing and `[Shot N]` labels. State the total duration and aspect ratio when either is specified or inferable.
2. Use unambiguous framing language. Describe both shot size and crop, for example: `two-person medium-close shot, framed from mid-thigh upward`. For a two-person close shot, specify which face is sharp and which figure is recognizably soft.
3. Preserve continuity across cuts: character identity, outfit, accessories, time of day, location, props, weather direction, and screen-left/screen-right placement remain stable unless the user requests a change.
4. Describe micro-expression acting as a sequence of isolated, small changes, each with its own time window. Specify eye state, gaze destination, eyelid tension, breathing pause when useful, head movement, mouth-corner movement, and the emotional result. Let pupils move before the head, and let softened eyelids precede the smile when the action calls for restrained affection.
5. Restrict a delicate emotional beat to one clear change at a time. Choose the actual movement from the user's scene and intended performance, not a default smile or eye-opening routine. Limit exaggerated movement only when it conflicts with the requested acting; preserve explicitly requested broad smiles, abrupt turns, or heightened performance.
6. For a resolved emotional beat, end with a stable, readable pose for the final 0.8–1.0 seconds when timing permits. Respect requested ongoing motion or other endings.

## Professional emotion-performance planning

When a request involves emotional acting, treat emotion words as outcomes to be earned, not as a list of expressions to display. Before drafting the timeline, internally define this compact performance brief:

1. **Given circumstance and trigger:** What has just happened, who is the character responding to, and what specific visible, audible, or already-established fact begins each change? Do not invent a trigger when the user has supplied none; use the least disruptive continuity-based trigger or keep the performance internally motivated and restrained.
2. **Objective, obstacle, and subtext:** What does the character want from the other person in this moment, what prevents it, and what are they unable or unwilling to say outright? Let dialogue, silence, and gaze serve that intention rather than label emotions.
3. **Emotional quality and intensity:** Distinguish the quality of an emotion (for example, indignant hurt versus betrayed anger; frightened compliance versus startled fear) and its intensity level. Do not treat every "sad," "angry," or "afraid" request as the same facial shape.
4. **Emotional arc and carryover:** Plan a causal path with readable stages—such as contained anger → hurt exposure → loss of control—rather than abrupt swaps. A later state retains appropriate residue from the prior one: anger may remain in breath and diction after tears begin; fear may remain in gaze and vocal hesitation after a plea.
5. **External score:** Select only the visible and audible evidence that the supplied framing can show: face (eyes, brows, mouth, jaw), voice (pace, volume, texture, pauses), and body only if the user permits it and the framing supports it. Coordinate all selected channels with the same event and avoid decorative gestures.
6. **Character and situation calibration:** Match amplitude to the stated personality, relationship, privacy of the setting, and stakes. A request for heightened performance permits larger, still-causal expression; a request for restraint favors pressure, interruption, and incomplete release rather than stock gestures.
7. **Playable endpoint:** State what remains at the end—recovered resolve, suspended fear, unresolved sobbing, relieved warmth, etc. Do not reset to neutral or a smile by default.

For actor-test prompts, make the acting challenge observable without turning the output into a grading rubric: provide a clear trigger, a legible inner shift, coordinated facial/voice evidence, and an end state. When the user asks for evaluation criteria, map the result to psychological causality, emotional quality, transition continuity, technical coordination, control of intensity, and felt realism.

## Actor performance: voice, expression, and timing

When a clip involves dialogue, vocal performance, or facial acting, read [演员演绎速查表](references/actor-performance.md). Select only the entries that serve the requested performance; do not paste the whole glossary into the prompt.

### 根据用户输入选择，不随机搭配

速查表是可选表达词库，不是随机抽取池，也不是每次必须覆盖的清单。先理解用户内容，再选择能实现这段表演的词条；不为了丰富画面、增加变化或展示标签而添加动作。

1. **提取依据**：识别用户明确指定的演法、禁用项、原始台词、人物关系与意图、正在发生的事件、情绪起点与变化、景别和时长。未提供的信息不编造成确定事实。
2. **按优先级判断**：用户明确演法与限制优先，其次是剧情事件和人物意图，再其次是台词语义与上下文。不能仅凭一个词或问号推断哭腔、愤怒、恐惧或嘲讽；如用户要求“笑着说狠话”，应保留笑与狠话的反差。
3. **先定表演节拍**：明确谁在什么刺激下产生什么变化，以及何时说话、停顿或保持沉默。没有情绪变化的平静陈述可以维持自然表情，不必制造转折。
4. **分别选择声音和可见动作**：对每个候选标签或微表情，内部确认“输入依据 → 表演目的 → 具体表达 → 触发时机”。声音、面部动作可表达不同层次的情绪，但这种反差必须有用户要求或情境依据。没有依据的候选项删除；不需要声音标签时可以只保留普通对白。
5. **检查可执行性**：只保留当前景别可读、时长可容纳且互不冲突的动作；根据克制或外放的演法调整幅度。不能为了加入某个词条而改写用户台词、人物意图或镜头计划。
6. **依据不足时收敛**：选择中性、自然、最少附加动作的表达，不随机补哭、笑、喘、结巴、耳语或吞咽。只有缺失信息会改变核心剧情或表演含义且无法保守处理时才询问。

以上判断在内部完成，默认仍只交付六段完整提示词；用户询问选择理由时，再说明输入依据与选用项的对应关系。同一输入的表达可以变化，但表演意图、强度和触发依据应保持一致。

### 将表演意图组织成连续过程

- 先确定人物此刻想做什么、为什么迟疑或改变，再安排可见动作。同一句台词可表达害羞、崩溃、麻木或克制，不能由台词字面直接套用演法。具体方法见[演员演绎速查表中的编排规则](references/actor-performance.md#从表演意图到连续过程)。
- 按需要形成“起始状态 → 刺激或准备 → 变化或中断 → 说话／行动 → 余波”的因果链；这不是固定五阶段模板，平静陈述不必制造爆发，短片段也不必填满所有阶段。
- 分别控制情绪强度、动作幅度、语速、音量、音色和对白清晰度。低声不自动等于含混，平淡不等于嘴唇不动；按实际发音呈现自然口型，不为清晰度强制夸张张嘴。
- 附属运动跟随真实原因：转头带动头发，低头带动饰物，呼吸带动肩胸。只使用用户描述或参考图确有的元素，不随机补风、飘带、流苏或配饰；强烈动作也要符合用户要求的画面边界。
- 结尾承接该次表演留下的状态或运动，不默认微笑、恢复中性或冻结画面。若未明确要求冻结帧，“停留在某种神态”可保留自然呼吸、眨眼或尚未结束的惯性运动，但不能违背声音与动作限制。
- 输出前统一台词版本并检查发声时间与声音范围。明确的台词修正或“只说某句”优先于示例中混入的台词；无法判断的核心冲突才询问。对白时长不够时先减少可选动作、调整未锁定的时间分配，不擅自删改台词、加速慢速演法或延长指定总时长。

### 多情绪转换与对白递进

当用户要求多个情绪变化时，先为整个片段写一句可检验的情绪弧线：人物因同一事件或关系变化，从什么状态经过哪些转折，最终留下什么状态。情绪数量服从时长和剧情，不为了“多”而排列无关表情。详见[演员演绎速查表中的多情绪编排规则](references/actor-performance.md#多情绪转换与对白递进)。

- 每次转换都必须有可见或可听的触发：对方的动作、沉默、已知信息、人物意识到自己的失态，或用户指定的事件。不得用未说明的画外声音、突然出现的物件或无原因的情绪跳变推动转折。
- 将台词当作情绪弧线的一部分：每句台词应推进人物目标、关系或压力，并与说话前后的表情相衔接。不要把多句台词写成同一语气、同一表情下的平铺陈述。
- 先安排目光、表情或身体准备，再让台词落下；说完后为对方的沉默、人物的意识或动作后果留出反应时间。若一句话本身是情绪顶点，允许其先说出口，再在余波中显露反应。
- 按台词含义和用户要求分别设定每句的语速、音量、重音和停顿；相邻句须保留声音连续性。强度可以上升、降低或反转，但不能无依据地从耳语跳到嘶吼，或从崩溃瞬间变成愉快。
- 无入画的对话对象可以作为角色的稳定注视方向，但只能以用户明确或已建立的剧情意图描述其状态。不得为方便推进情绪，擅自给对方补台词、脚步、关门声或其他可听行动。

以下仅演示选择逻辑，不是情绪到标签的固定映射：

| 用户输入 | 可选择的演绎 | 不应自行附加 |
| --- | --- | --- |
| “压低声音，怕门外的人听到” | 根据压低程度选择 `<softer>` 或 `<whisper>`；有观察门口的剧情时才加入侧瞥 | 无依据的哭腔、喘气、惊恐表情 |
| “想哭但忍住，平静地说完” | 用眼睛湿润但不落泪等少量可见变化体现克制，声音保持用户要求的平静 | 自动使用 `[Chinese, crying]`、抽泣或下唇持续颤抖 |
| “跑到门口，喘着气说话” | 按台词间隙放置 `<pant>` 或 `<catches breath>`，给恢复呼吸留时间 | 将喘气直接解释为害怕，或在同一瞬间安排持续说话与吸气 |
| 只有“你真的要走吗？” | 保留询问语义，使用自然语气；等待上下文决定更具体演法 | 仅因“走”或问号就指定悲伤、哭腔或挽留动作 |

- Bind each speaking character once in `subject_definitions`, using `<Subject N> (SN)` with the same subject number throughout. Do not assign speaker identities to environment or style references. Maintain the character's voice across cuts.
- In timed `detailed_description` blocks, place language and delivery cues plus dialogue and vocal tags inside `<d>...</d>`. Keep the speaker label, facial movements, body acting, and camera instructions outside it. Preserve the user's actual dialogue and language; translate descriptive prose only as requested. Tags retain their exact spelling.
- Convert each emotional beat into a visible sequence: trigger → localized facial change → spoken or silent reaction → release or hold. Specify amplitude, duration, gaze target, and the relation to speech when useful. Word-triggered reactions must follow an actual word in the supplied dialogue; do not change dialogue merely to fit an example.
- Coordinate speech, breathing, and visible movement on one timeline. Allow enough time for the line, pauses, and reactions. Avoid simultaneous contradictory actions such as closed lips during speech, an inhale during a sustained utterance, or a smile held while its disappearance is requested. Sequential beat windows may contain naturally synchronized audio and visual actions.
- Use a small number of readable changes per beat. Match detail to framing: subtle eyelid or throat changes belong in a readable close view, while distant shots need larger visible gestures. Do not insert a close-up or cut against the user's shot plan merely to show a micro-expression.
- Silence does not require a dialogue block. Do not add speech, humming, audible breathing, or other vocalizations when the user requests a silent performance; visible breathing can be described outside `<d>`.
- These tags are user-provided writing conventions, not a verified universal H3 parser or API contract. Use them as the skill's default notation; if the target environment is known not to support them, express the same performance in plain language. Do not add compatibility disclaimers to every generated prompt.
- Include `Facial Realism` only when the user or actual workflow confirms that `MiniMax-H3-Facial-Realism-CloseUp` LoRA is loaded. Never assume it is loaded, load it automatically, or treat the phrase as a universal quality booster.

## Scene acting: listening, cues, actions, status, blocking, timing

For any clip with dialogue between characters, a confrontation, a negotiation, a comic beat, or any request about acting quality (“文戏”, “演技”, “台词”, “表演”), also read [文戏表演编排](references/scene-acting.md). It complements the actor-performance glossary: the glossary controls *how one speaker sounds and moves*; scene-acting controls *who listens, how lines are picked up, what each line does to the other person, who holds status, how distance and props carry subtext, where stress and pauses go, and whether the lines fit the time*.

Apply it with the same selection discipline (input basis → purpose → visible/audible evidence → timing). In particular:

- **Listener track.** For every line in a multi-character clip, also write the listener: the exact word that lands, the first reacting channel, its delay (normally 0.2–0.4 s; a freeze of 0.3–0.6 s before reacting when stung), and any suppressed attempt to interrupt. Do not invent lines or sounds for an off-screen listener.
- **Cue pickup.** Choose normal pickup, overlap, interruption (written as a broken line ending in “——”), or a delayed reply with a visible silence, according to relationship and stakes. Never let characters alternate at an even, recital-like pace without reason.
- **Action verbs.** Give each line an internal action toward the other person (逼问, 搪塞, 试探, 讨好, 刺痛, 稳住 …). A turn in the scene is a change of tactic after the previous one fails, preceded by a visible moment of realization. Keep these verbs internal unless the user wants them shown; express them through gaze, timing, stress, and delivery.
- **Status.** Decide who holds the higher position and whether it flips; place the flip on a specific line or action. Do not equate loudness with status.
- **Blocking and props.** When the user permits body acting and the framing can show it, use distance, orientation, eye level, and existing props as subtext. Never add props or movement the input does not support.
- **Stress and pauses.** One main stress per line (`<i>…</i>`), chosen by what the speaker most needs the other person to hear. Distinguish logical pauses (0.2–0.4 s) from psychological pauses (0.5–1.5 s) and write what the face does during a psychological pause.
- **Time budget.** Check each window with the speech-rate table in scene-acting (≈4 字/秒 normal; 5–6 heated; 2.5–3 restrained). If the lines do not fit, cut optional business first; never ship a timeline that cannot be spoken.
- **Genre register.** Calibrate to the genre grammar (everyday realism, epic/period, comedy timing with setup → beat → punchline and delayed reactions, dialect only on request).
- **Failure-mode bans.** Add the 2–4 most likely AI mis-performances for this beat to the description's restrictions (e.g., constant smiling, recital-like alternation, dead eyes while speaking, lip-sync drift).

## Single-clip action and audio planning

Build one readable beat: starting state → main action or interaction → visible result. Allocate non-overlapping time windows that cover the full duration without gaps. For a continuous take, use one `[Shot 1]` block with timed sub-beats. Add further shot blocks only when cuts are requested or necessary to the user's stated shot plan.

Use micro-expression details for delicate emotional acting; use clear anticipation, contact, weight transfer, and follow-through for physical actions. Do not force restrained smiles or small movements onto energetic action. Keep the final hold only when it fits the requested ending; ongoing motion can remain ongoing when requested.

In `overall_soundscape`, describe ambience, action sounds, voice quality, and mixing priorities, referring to the speaker and dialogue timing already written in `detailed_description`. Write each full dialogue line once in its timed shot block to avoid duplicate delivery or conflicting directions. In `non_diegetic_music`, follow the requested music treatment; default to no added score when unspecified. Do not invent dialogue, lyrics, captions, or extra people. These six headings are this skill's writing format, not a claim about H3 API fields.

## Quality pass before output

Check that:

- all attached references have a defined role;
- every extraction-only composition/action source is omitted from the final H3 reference-image upload set, omitted from `subject_definitions`, and represented only by text constraints in `detailed_description`;
- uploaded scene and character references, not extraction-only sources, are the sole basis for generated identity, wardrobe, and environment;
- each subject keeps the same label in every section;
- the timeline covers exactly the user's duration or the clearly stated planning default;
- every cut has a clear pre-cut and post-cut framing;
- when a style-only reference exists, it controls visual treatment, never source content;
- every delicate micro-expression is broken into chronologically ordered, isolated beats;
- for emotional acting, each change has a causal trigger or an explicitly established inner realization; the emotional quality, intensity, visible/aural evidence, and carryover from the previous beat agree;
- the performance uses only the expression channels the user permits, fits the framing and character situation, and ends in the requested residual state rather than a default reset;
- style-only references contribute attributes only, never content;
- visual continuity is locked across all shots;
- the prompt forbids the likely generation failures for the requested beat;
- sound and music do not contradict the user's requirements;
- speaker bindings remain consistent, and each dialogue line appears once with enough time for its delivery and pauses;
- `<d>` blocks contain only dialogue, language/delivery cues, and vocal tags; facial actions and camera directions stay outside;
- expression triggers, lip movements, breathing, and spoken words agree on the timeline, including silent-performance requirements;
- `Facial Realism` appears only when the corresponding LoRA is confirmed loaded.
- every selected vocal tag and facial action has an identifiable basis in the user's instructions or scene context; unsupported embellishments and automatic emotion-to-tag mappings are removed.
- emotional changes, accessory motion, and ending states follow the scene's causal sequence rather than a stock performance template;
- dialogue wording, delivery speed, available speaking time, and permitted sound sources agree; unresolved hard constraints are not concealed by an impossible timeline.
- for multi-emotion performance, every change has a stated trigger and causal link; dialogue, reaction windows, and the final state form one continuous emotional arc.
- in multi-character clips, every line has a listener reaction and a chosen cue-pickup type; interruptions appear as broken lines in the dialogue itself;
- each line has one main stress at most; psychological pauses state what the face does; every window passes the speech-rate check;
- status, blocking, and props (if used) are supported by the input and readable in the framing; the genre register matches the request;
- the 2–4 most likely AI mis-performances for this beat are explicitly excluded;
- the output is one `text` code block containing exactly the six headings in order, with no Markdown inside and nothing outside unless requested.
