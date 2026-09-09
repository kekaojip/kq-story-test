# workflow-v2-override.md：长篇双仓 Writer V2 强制覆盖层

> 本文件只覆盖 `story-long-write` 在 **正文创建 / 续写 / 日更 / 回炉 / 重写 / 审稿 / 去 AI / Tracking 收编** 阶段的旧单仓条款。
>
> 旧 `workflow-chapter.md`、`workflow-daily.md`、`workflow-revision.md` 保留作历史与非冲突方法参考；一旦与本文件冲突，以本文件为准。
>
> 本文件与 `external-writer-bridge.md` 一起构成长篇双仓正文生产权威。事实与权限冲突仍按主工作流现有 Truth / Constraint Lock 处理。

---

## 0. 强制读取与适用范围

以下任一场景进入正文阶段前，Main 必须完整读取：

1. `external-writer-bridge.md`
2. 本文件 `workflow-v2-override.md`
3. 对应旧流程文件中仍不冲突的方法部分

适用场景：

- 写第 N 章
- 日更 / 续写
- 修改第 N 章
- 回炉 / 重写已写章节
- Writer 返回后的审稿与返修
- 最终正文收编与 Tracking 提交

短篇不受本覆盖层影响。

---

## 1. 统一生命周期：Tracking 永远晚于 PASS

所有长篇正文，无论新写、日更、回炉还是重写，统一使用：

```text
Main 准备当前章真相与执行输入
↓
发布 Writer Workspace
↓
Writer 生成候选
↓
story-review 只读审稿
↓
必要时 story-deslop DETECT_ONLY
↓
Main 裁决 PASS / PASS WITH MINOR / REVISE
↓
必要时 Writer LOCAL_REVISION
↓
最终 PASS
↓
正文收编主仓
↓
Tracking transaction
↓
下一章
```

### 强制覆盖旧规则

以下旧行为在双仓 V2 中全部失效：

- “正文一落盘就立即提交 Tracking”
- “先提交 Tracking，批末再统一 review”
- “`/story-review lean` 只是可选提示”
- “批末对已经收编正文再次自动去 AI 并直接改文”
- “质量检查 agent / narrative-writer 直接修改 Writer 成稿”

**只有最终 PASS 的正文版本可以进入 Tracking。**

若 Reviewer / Writer revision 改变事实、人物状态、伏笔、时间线或下一章承诺，Tracking 只根据最终 PASS 版本构造一次正式事务；已提交历史章的回炉则使用现有 `mode=revision` 规则。

---

## 2. 日更 / 批量续写覆盖规则

`workflow-daily.md` 的加载、旧信息查询、细纲补建、跨章核对等方法仍可使用，但每章正文阶段改为严格串行：

```text
第 N 章 Main 编译 Workspace
→ Writer
→ Review / 必要 Deslop Detect
→ PASS / Revision
→ 最终 PASS
→ 收编 + Tracking
→ 才允许第 N+1 章
```

### 禁止

- 同时发布多个 Writer 章节。
- Writer 尚在 `awaiting_external_writer` 或 `awaiting_writer_revision` 时先写下一章。
- 为了批量效率跳过逐章 Review。
- 批末再对已 PASS 章节执行全章 AI wash。

### 批末检查只做跨章诊断

批末可以做：

- 标题去重
- Tracking 完整性 / state check
- 跨章契约与连续性核对
- 本批新增伏笔 / 关系 / 时间线一致性检查

如果批末发现某章需要正文修改：

1. 定位到具体章；
2. 重新进入该章 V2 Revision 流程；
3. Writer 修改；
4. Review；
5. 最终 PASS；
6. 再做对应 `mode=revision` Tracking 事务。

不得在批末直接改正文后只补 Tracking。

---

## 3. `workflow-chapter.md` Phase 5 覆盖规则

旧 Phase 5 中所有“narrative-writer 去 AI 审查后直接删改正文”“主会话按 anti-ai reference 直接改正文”“三脚本逐章清零”条款，在双仓生产中改为：

### 3.1 Writer 本地只读 preflight

Writer 负责：

- `check-outline-copy.js`
- `check-degeneration.js`
- `measure_draft.py`

它们只产生 evidence，不自动触发全章改写。

### 3.2 Main 正式 Review

外部候选返回后必须运行 `story-review`。

Reviewer 只输出 findings，不写回正文。

### 3.3 Deslop 只在命中具体 prose 病灶时调用

必须明确使用：

`仅标注 / 只检测 / 不要改`

不得进入自动逐项清除或全文三遍清洗。

### 3.4 所有非机械正文修改都退回 Writer

Main 只生成 `REVISION.md`，不自己把 Reviewer 建议改写成新正文。

`PASS WITH MINOR` 仅允许错字、标点、姓名误写、明确格式错误等机械修正；一旦涉及句式、节奏、解释、对白、段落或场景表达，仍视为 Writer revision。

---

## 4. 历史章节回炉 / 重写 V2

旧 `workflow-revision.md` 的“主仓直接修改正文”在双仓 V2 中失效。

历史章修改必须先构造一个 revision Workspace。

### 4.1 Main 读取

- 主仓当前正式 `正文/第{X}章_*.md`
- 第 X 章原细纲
- 第 X-1 章最终正文尾部；第1章则使用开篇必要设定
- 第 X+1 章正文或细纲（若存在），仅用于连续性边界
- 第 X 章写完后当时应成立的历史状态 / 可重建状态
- 当前静态人设与会影响修订合法性的长期规则
- 用户本次修改要求

### 4.2 Revision Workspace

发布到 Writer：

```text
00_TASK.md
01_OUTLINE.md
02_CURRENT_STATE.md
03_PREVIOUS_PROSE.md
04_BOUNDARIES.md
05_STYLE_RESOLUTION.md
06_AUTHOR_PREFERENCES.md
ORIGINAL_DRAFT.md
NEXT_CONTEXT.md              # 有需要才生成
characters/
rules/
benchmark/
  EXECUTION_CARD.md
REVISION.md
HANDOFF_STATE.json
```

`ORIGINAL_DRAFT.md` 是待修改原文，只用于 revision；普通 FIRST_DRAFT 不生成。

`NEXT_CONTEXT.md` 只放后文连续性所需最小片段 / 约束，不把未来全章或未来秘密无界暴露给 Writer。

### 4.3 Revision 类型

- 局部修改：`REVISION.md` 明确 `LOCATION / DEFECT / MUST_PRESERVE / TARGET / SCOPE`。
- 全文重写：仍必须保留事件结果、真值、知识边界、章尾停点和后文已经依赖的事实；Main 必须在 `04_BOUNDARIES.md` 中明确。

### 4.4 收编

Writer 返回 → Review → 必要 Deslop Detect → Main 裁决 → 最终 PASS。

最终 PASS 后：

1. 先保留主仓原正式正文备份；
2. 用 PASS 版本替换正式章；
3. 从第 X 章到当前最后已写章检查受影响事实；
4. 使用 `mode=revision` Tracking transaction 更新当前权威状态；
5. 成功通过 Tracking check 后才算回炉完成。

---

## 5. REVISION 输出命名统一

生产协议统一采用**版本化输出，不覆盖第一稿证据**：

### FIRST_DRAFT

```text
output/current/draft.md
output/current/report.json
```

### 第一次 REVISION

```text
output/current/draft_v2.md
output/current/report_v2.json
```

### 只有 S1/S2 仍未解决且 Main 明确授权第二次返修时

```text
output/current/draft_v3.md
output/current/report_v3.json
```

不得让 Writer 在 revision 时覆盖 `draft.md / report.json`。

最终收编必须显式记录采用哪个版本。

Revision 时 Main 必须在 `HANDOFF_STATE.revision_base` 中显式指定返修基线；普通第一次返修通常为 `output/current/draft.md`，历史章回炉为 `input/current/ORIGINAL_DRAFT.md`。Writer 不得只凭聊天上下文重构上一稿。

---

## 6. 文风 / 作者偏好跨仓传递

Writer 禁止跨仓读取主小说目录，因此旧 `style-resolution.md` 中“Writer 自己打开主仓 `设定/文风.md` / 查询作者记忆”的单仓行为，在 V2 中由 Main 预编译替代。

Main 每次发布 Workspace 时生成：

```text
05_STYLE_RESOLUTION.md
06_AUTHOR_PREFERENCES.md
```

具体编译协议见：

`writer-style-package.md`

### 原则

- `05_STYLE_RESOLUTION.md` 是本次生效的表达裁决，不是真相源。
- `06_AUTHOR_PREFERENCES.md` 只放本章相关 active 偏好，保持紧凑。
- 当前用户请求优先于二者。
- 二者不得改变事件、知识边界、停笔点和 Tracking 真相。
- Writer 不再因为缺主仓路径而自行猜文风。

---

## 7. Workspace 发布卫生

每次发布新任务前，Main 必须把 Writer `input/current/` 当成**热工作区**处理，而不是增量堆文件。

### 发布前必须清理上一任务遗留

归档上一任务后，清理：

- `00_TASK.md` ～ `06_AUTHOR_PREFERENCES.md`
- `ORIGINAL_DRAFT.md`
- `NEXT_CONTEXT.md`
- `REVISION.md`
- `characters/*`
- `rules/*`
- `benchmark/*`

上一任务的 `HANDOFF_STATE.json` 在归档完成后保留为 `status: archived`；发布下一任务时用新任务状态**原子替换**，不先删除。

然后只写当前任务需要的文件。

禁止上一章角色、规则、benchmark、Execution Card、Revision 残留到下一章。

`output/current/` 同样在发布新任务前只保留 `OUTPUT_CONTRACT.md`。

历史证据必须先归档，再清热区。

---

## 8. Handoff 状态持久化

聊天上下文不是状态源。每次 Main 发布任务时必须写：

`input/current/HANDOFF_STATE.json`

协议见：

`writer-handoff-state.md`

至少区分：

- `published`
- `awaiting_external_writer`
- `reviewing`
- `awaiting_writer_revision`
- `accepted`
- `archived`
- `blocked`

Main 在每次状态切换时更新该文件；Writer 不修改 input。

新会话恢复时，先读该状态，再读当前 input/output，禁止只凭聊天记忆判断工作进行到哪。

---

## 9. Archive Manifest

最终 PASS 后，Main 在清空热区前把本章证据归档到 Writer：

`archive/chapter-{NNN}/`

并生成：

`MANIFEST.md`

Writer 仓的模板：

`archive/MANIFEST_TEMPLATE.md`

Manifest 至少登记：

- chapter / title
- workspace source commit / main source state revision
- draft versions
- review verdict
- deslop detect 是否运行
- final accepted version
- 主仓正式正文路径
- Tracking commit / state revision（可得时）
- proposed additions 裁决摘要

归档不是小说真相源，只是审计证据。

---

## 10. 字数口径覆盖

Main 在 `00_TASK.md` 中必须明确：

- 用户原始字数目标或范围
- `visible_chars_v1`

如果 `01_OUTLINE.md` 也有标准 `字数目标 / 字数口径`，两者必须一致；不一致先阻塞发布。

Writer 的 `measure_draft.py` 必须优先识别当前任务显式 target / range；无法解析目标时要报告 `target_unresolved`，不得静默退化成“只测实际字数但假装完成 target check”。

---

## 11. 不变量

1. Main 决定写什么。
2. Writer 决定批准语义怎么落成正文。
3. Reviewer / Deslop 只诊断。
4. Human Writing L2 控制第一稿自然成文，不做后置全章 AI wash。
5. Tracking 只接收最终 PASS 正文。
6. 新章、日更、历史回炉全部走同一双仓闭环。
7. 旧 workflow 中任何与本文件冲突的“主仓直写 / 先 Tracking / 批末自动改文”条款均视为 legacy，不执行。

---

## 12. Final Parity：第一稿默认 CHECKPOINTED

为恢复原版 `story-long-write` 的“前组 → midpoint 字数反馈 → 后组”能力，长篇 FIRST_DRAFT 增加两种交付模式：

- `CHECKPOINTED`：默认。
- `ONE_SHOT`：仅当用户明确要求“一次成文 / 一次性写完整章”或 Main 有明确任务级理由时使用。

Main 在 `00_TASK.md` 与 `HANDOFF_STATE.json` 中显式写：

```text
delivery_mode: CHECKPOINTED | ONE_SHOT
delivery_phase: FRONT | COMPLETE
heading_literal: <完整标题首行，或 NONE>
```

### 12.1 CHECKPOINTED / FRONT

Main 仍发布**整章** 00-06、Outline、Boundaries 与 EXECUTION_CARD，让 Writer 能统筹全章；但 `expected_output` 只等待：

`output/current/segment.md`

Writer 在批准情节点之间选择自然的场景 / 因果停顿处结束前段，不得截断一句对白、一个动作或一个必须连写的微场景，也不得为了“正好一半”机械切段。

`segment.md`：

- 不是正式候选稿；
- 不进入 story-review；
- 不进入 Tracking；
- Main 不在 checkpoint 阶段改写它。

### 12.2 Main midpoint checkpoint

收到 `segment.md` 后，Main 使用现有原版字数能力按 `visible_chars_v1` 测量，优先沿用：

`storyctl.py wordcount checkpoint`

Main 计算并写回：

```text
checkpoint_actual
remaining_user_range
front_completed_scope
remaining_scope
```

同时把：

`delivery_phase: COMPLETE`

并把 HANDOFF `expected_output` 改为：

```text
output/current/draft.md
output/current/report.json
```

Main 不以 checkpoint 为由新增、删除或重排批准剧情。

如果前段已经使剩余用户范围很紧或已经超预算，仍不得要求 Writer 把未完成批准内容硬塞成提纲；Writer先自然完成全部批准内容，完整稿若最终 `over`，再按本文件 `COMPRESS_ONCE` 处理。

### 12.3 CHECKPOINTED / COMPLETE

Writer 必须把既有 `segment.md` 当成**冻结正文前缀**：

- 原文保持不变；
- 不为了追字数回改前段；
- 只继续 `remaining_scope` 中尚未完成的批准内容；
- 最终 `draft.md = segment.md 原文 + 后续正文`。

checkpoint 只是字数反馈，不是审稿，不改变 Human Writing L2、Truth Guard 或章节蓝图。

### 12.4 ONE_SHOT

ONE_SHOT 直接等待：

`draft.md + report.json`

不生成 `segment.md`，后续 Review / Revision / Tracking 与普通 V2 相同。

---

## 13. Final Parity：Writer 交付附件映射到 report.json

完整候选稿的 `report*.json` 除原字段外，必须提供三组可审计信息。

### 13.1 `outline_coverage`

用于恢复原版“时空表 / 情节点覆盖证明”的功能，但不要求复制旧大表格。

最小结构：

```json
"outline_coverage": [
  {
    "beat": "批准情节点的语义简写",
    "status": "landed|partial|unwritten",
    "location": "正文段落或场景位置"
  }
]
```

Main 必须拿**实际正文 + 01_OUTLINE**复核，不能因为 Writer 自报 `landed` 就自动 PASS。

任何必须情节点为 `partial / unwritten` 时，不得静默进入 Tracking。

### 13.2 `unwritten`

恢复原版“本章没写成的”供给反馈：

```json
"unwritten": [
  {
    "beat": "未成功落地内容",
    "reason": "input_insufficient|boundary_blocked|conflict_or_ambiguity|space_or_pacing|other",
    "note": "最短必要说明"
  }
]
```

Main 收到后先分类，不一律把问题甩回 Writer：

- Writer 执行问题 → `REVISION.md`。
- `input_insufficient` → 回 Main 补细纲 / Execution Card / 必要材料后重新发布，不让 Writer猜。
- `conflict_or_ambiguity` → 查 Tracking / 设定 / Truth 后再决定。
- `boundary_blocked` → 若确属禁止新增，则接受“不写”；若边界本身配置错误，Main 修输入后再发。
- `space_or_pacing` → 先判断是不是细纲供给过密；不得单纯要求 Writer 压成摘要。

### 13.3 `references_read`

Writer 报告实际读取的 Skill / reference 路径，例如：

```json
"references_read": [
  "skills/human-writing-l2/SKILL.md",
  "references/execution/combat-execution.md"
]
```

它只用于可观测性，不作为“读得越多越好”的评分项，也不因为列表短就自动返修。

### 13.4 `proposed_additions` 对象化

V2 Writer 新报告优先写：

```json
"proposed_additions": [
  {
    "item": "新增物",
    "type": "character|fact|relation|setting|resource|other",
    "why_needed": "为什么当前场景需要",
    "future_obligation": "none|possible"
  }
]
```

这只提高申报分辨率，**不扩大 Writer 权限**。旧报告若仍是字符串数组，Main 继续兼容读取。

新主线事件、新反转、新金手指规则、提前后续剧情、改变既定结果仍是禁止项，不能借 `proposed_additions` 合法化。

---

## 14. Final Parity：标题行形态继承

Main 在编译 `00_TASK.md` 时增加：

`heading_literal`

它必须是**Writer 应原样写入正文第一行的完整标题字面量**，例如：

`# 第13章 山门之前`

Main 生成方式：

1. 优先读取上一已接受章节的标题行形态；
2. 保留其 Markdown / 空格 / “第N章”格式；
3. 替换为当前批准章节号与章名；
4. 第1章或没有历史正文时，使用本项目当前平台约定；
5. 若项目正文明确不含标题，写 `heading_literal: NONE`。

Writer 有 `heading_literal` 时不得自行改成另一种标题格式。Revision 除非任务明确改标题，否则保持原 literal。

---

## 15. Final Parity：`OVER → COMPRESS_ONCE`

如果完整候选通过剧情覆盖 / 真值 / 边界检查，但 `visible_chars_v1` 明确为 `over`，Main 可以发一次专项 Revision：

```text
revision_type: COMPRESS_ONCE
target_range: <用户范围或当前权威范围>
MUST_PRESERVE:
- 全部批准且已落地情节点
- 事件结果
- 人物知识边界
- 伏笔与信息释放
- heading_literal
- 章尾停笔点
SCOPE:
- 全章仅做净删 / 压缩，不新增语义
```

Writer 只允许：

- 删除重复解释；
- 删除重复反应；
- 合并同义表达；
- 压缩无新信息的过渡；
- 在不损失场景因果的前提下做局部净删。

禁止：

- 新增剧情或事实；
- 改变事件顺序 / 结果；
- 删除必须情节点；
- 把场景压成提纲摘要；
- 为了过字数检测进行全章同义词洗稿。

`COMPRESS_ONCE` 最多执行一次。完成后重新测字数并重新进入 Review。

若仍 `over`，不自动第二次压缩，由 Main / 用户选择：接受当前长度、修改目标/细纲，或另行明确授权其他处理。

`under` 仍不得靠新增剧情补字数。