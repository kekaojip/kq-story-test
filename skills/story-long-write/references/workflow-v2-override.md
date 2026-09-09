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
