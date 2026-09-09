# Writer Execution Card｜主侧正文执行编译规范

## 目的

把主工作流已经完成的剧情、人物、悬念、情绪、战斗、反转与章尾决策，压缩成外部 Writer 能直接执行的**当前章正文执行卡**。

它不是新的剧情设计层，也不是第二份细纲。

> Main 负责决定“写什么”；Execution Card 只把这些批准决定编译成 Writer 需要的“怎么执行到位”。

输出位置：

`kq-story-writer/input/current/benchmark/EXECUTION_CARD.md`

如果当前章不需要专项模块，可以只生成 `PROSE CORE`，不存在的模块直接省略。

---

## 权威边界

Execution Card 只能从以下已批准材料编译：

- 当前章细纲 / `01_OUTLINE.md`；
- 当前 Tracking / `02_CURRENT_STATE.md`；
- 当前人物、规则与知识边界；
- 当前章已批准 benchmark；
- 主模型已经完成的悬念、战斗、反转、情绪、钩子设计判断；
- 作者当前明确要求。

Execution Card **不得**：

- 新增细纲里没有的主线事件；
- 新增真相、规则、伏笔、敌人、奖励或升级；
- 改变事件结果；
- 改变人物知识边界；
- 改变批准停笔点；
- 用“为了好看”替代上游事实；
- 把完整设计方法论原样塞给 Writer。

发生冲突时：

`04_BOUNDARIES > 01_OUTLINE > 02_CURRENT_STATE > 00_TASK > characters/rules > previous prose > benchmark/EXECUTION_CARD`。

Execution Card 永远不能覆盖更高层输入。

---

## 输出模板

```md
# EXECUTION CARD

## PROSE CORE
- chapter_function: {本章在当前剧情单元中的功能}
- reader_experience: {本章最主要的读感/情绪体验}
- visible_change: {本章最重要的可见变化}
- prose_pressure: {当前场景主要叙事压力}
- explanation_limit: {本章解释应停到哪里；无特殊要求写“服从 Human Writing L2”}
- must_land: {正文必须真正落出来的批准内容}
- do_not_invent: {本章特别容易被 Writer 自行补造的东西}

## SUSPENSE
- core_question: {当前核心问题}
- pov_knows: {视角人物当前知道什么}
- reader_knows: {读者当前已知什么}
- may_reveal_this_chapter: {本章允许新增知道什么}
- forbidden_reveal: {本章禁止揭示什么}
- evidence_available: {已经批准、可以写进正文的线索/证据}
- release_order: {必要时写批准的信息释放顺序；无则写“自然执行”}
- end_knowledge_state: {章末人物/读者应知道到哪里}

## COMBAT
- approved_result: {胜负/逃脱/中断/伤势等结果}
- primary_payoff: {本场主要兑现的爽感/压力/人物能力}
- must_show: {必须展示的已批准能力、判断或结果}
- may_compress: {已经展示过、可以一笔带过的内容}
- forbidden_additions: {不能自行新增的技能/底牌/敌人/奖励}
- battle_endpoint: {这场战斗最终落在哪里}

## HOOK
- opening_function: {如有明确章首功能}
- ending_function: {章尾需要留下什么阅读动力}
- approved_final_beat: {最后一个批准动作/对白/信息/画面}
- stop_after: {写到这里立刻停止，不再补下一拍}

## REVERSAL
- surface_belief: {人物/读者当前表层认知}
- truth: {仅在 Writer 本章需要知道时提供；否则放入 forbidden_reveal 而不明示}
- approved_clues: {本章已批准可出现的线索}
- reveal_level: {不揭示 / 发现矛盾 / 部分确认 / 完整揭示}
- post_reveal_knowledge: {揭示后各方知道到哪里}
- forbidden_reveal: {不得提前释放内容}

## EMOTION
- target_emotion: {本章目标情绪}
- carrier: {主要由人物/关系/目标/规则/场景中的什么承载}
- trigger: {本轮批准触发}
- visible_payoff: {必须让读者看见的变化/兑现}
- do_not_overexplain: {哪些东西让动作/对白/结果承担，不要二次解释}

## DIALOGUE
- relationship_action: {关键对话当前真正要完成的人际动作}
- information_control: {谁愿意说什么、谁不愿意说什么}
- must_not_become_exposition: {不能被 Writer 写成科普说明的内容}
```

---

## 模块路由

只生成当前章需要的模块：

| 当前章情况 | 生成模块 |
|---|---|
| 所有章节 | `PROSE CORE` |
| 悬疑、异常、信息差、调查 | `SUSPENSE` |
| 打斗、斗法、智斗、装逼兑现 | `COMBAT` |
| 章首或章尾需要特殊落点 | `HOOK` |
| 身份/信息/动机/认知/时间线反转 | `REVERSAL` |
| 明确情绪交付、关系拉扯 | `EMOTION` |
| 关键谈判、试探、关系对白 | `DIALOGUE` |

不要为了“完整”把七个模块每章填满。

---

## 编译原则

### 1. 传决定，不传教材

主模型可以读取完整：

- `long-suspense.md`
- `style-combat-face.md`
- `long-chapter-hooks.md`
- `long-reversal.md`
- `plot-emotion-system.md`
- `emotional-methods.md`
- `reader-contract-and-progression.md`

但 Writer 只收到这些方法论已经产出的**当前章决定**。

### 2. 传执行边界，不给 Writer 新设计权

错误：

> “这一章要加强悬念，可以多加几个假线索。”

正确：

> `approved_clues: 门锁没有破坏痕迹；桌上少了一只杯子`
> `forbidden_reveal: 暂不确认来人身份`

错误：

> “战斗要有反转。”

正确：

> `approved_result: 主角先被压制，使用已批准的第二式后反胜`
> `forbidden_additions: 不新增临时底牌`

### 3. 允许“不完整”

Execution Card 本身也服从 Human Writing L2 思想：

- 不要求每个问题本章解决；
- 不要求每个情绪本章解释；
- 不要求人物获得统一认知；
- 不要求每个线索加工出结论。

卡片要写清“本章允许到哪里”，而不是逼正文把事情做完。

### 4. 最小充分

卡片只保留会改变 Writer 正文选择的信息。

如果某条设计已经完整体现在 `01_OUTLINE.md` 且没有特殊执行风险，可以不重复。

---

## Writer 侧对应

Writer Runtime 根据存在的模块按需读取：

- `PROSE CORE` → Human Writing L2 + `execution/scene-craft.md`
- `SUSPENSE` → `execution/suspense-execution.md`
- `COMBAT` → `execution/combat-execution.md`
- `HOOK` → `execution/hook-execution.md`
- `REVERSAL` → `execution/reversal-execution.md`
- `EMOTION` / `DIALOGUE` → Human Writing L2 +现有 dialogue/craft references

Writer 不得根据“模块缺失”自行补一个模块。

---

## 验收

发布 Writer Workspace 前检查：

- Execution Card 是否只包含批准内容？
- 是否给了 Writer 不该拥有的剧情设计权？
- 是否和 00-04 冲突？
- 是否把方法论抄进卡片而不是编译成决定？
- 是否存在应该明确的 `forbidden_reveal / stop_after / forbidden_additions`？
- 是否只生成当前章需要的模块？

任何冲突先修主侧输入，再发布 Writer Workspace。
