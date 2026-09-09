# external-writer-bridge.md：外部正文 Writer 双仓库桥接协议（V1）

> 本协议是 `story-long-write` 的正文执行层权威覆盖协议。
> 
> **上游主工作流继续负责：** 选题、拆文、设定、人物、全书/卷/章规划、对标召回、状态筛选、边界裁决、审稿、Tracking、最终入库。
> 
> **外部 Writer 只负责：** 把已经确定的章节语义写成正文，并按要求返修。

---

## 0. 默认拓扑

- 主工作区 / 唯一真相源：当前小说项目所在仓库（本测试环境为 `kekaojip/kq-story-test`）
- 外部 Writer 仓库：`kekaojip/kq-story-writer`
- Writer Skill：`skills/story-writer-runtime/`
- Writer 当前输入：`input/current/`
- Writer 当前输出：`output/current/`

正文不得再由主工作流中的 `narrative-writer` 作为默认执行者生成。除非用户明确要求 fallback / 本地直写，否则进入正文阶段时必须走本协议。

---

## 1. 职责边界

### 主工作流拥有决定权

- 本章必须发生什么
- 谁出场、谁知道什么
- 哪个伏笔推进/回收
- 哪些信息允许释放、哪些禁止提前释放
- 章尾停笔点
- 人物与规则真相
- 对标技法选择
- 是否 PASS / REVISE
- 最终正文是否进入主仓库
- Tracking 如何更新

### Writer 只拥有表达权

- 句子如何落地
- 对话怎么说
- 动作与场景如何展开
- 段落与叙述节奏
- 局部生活化细节
- 在不改变剧情契约前提下的自然场景化

Writer **不得**直接修改主仓库大纲、人物设定、世界规则、Tracking、未来剧情或真相文件。

---

## 2. 写正文前的主侧准备

仍先完整执行 `workflow-chapter.md` 的：

1. 细纲检查
2. 卷纲 write-stage 取段
3. Tracking / 当前状态筛选
4. 相关人物、规则、势力读取
5. 对标召回与文风裁决
6. Constraint Lock

当这些内容都已确定后，**不要直接生成正文**，而是编译为 Writer Workspace。

---

## 3. Writer Workspace 固定结构

发布到 `kq-story-writer/input/current/`：

```text
00_TASK.md
01_OUTLINE.md
02_CURRENT_STATE.md
03_PREVIOUS_PROSE.md
04_BOUNDARIES.md
characters/
rules/
benchmark/
```

### `00_TASK.md`
只描述任务元信息：项目、章节号、章名、目标字数、平台/题材、POV、输出路径、当前明确指定的题材正文卡。

### `01_OUTLINE.md`
当前章细纲全文或保持语义等价的章节蓝图。细纲是“要发生什么”，不是正文句型模板。

### `02_CURRENT_STATE.md`
只装“不知道就会把本章写错”的热状态：
- 当前地点/时间
- 当前活跃人物状态
- 当前知识边界
- 本章必须履行的上一章承诺
- 当前开放问题

不得把完整 Tracking state 全量塞给 Writer。

### `03_PREVIOUS_PROSE.md`
- 第1章：明确写“无上一章正文”
- 第 N>1 章：放上一章正式 PASS 版本末尾约 1000–1500 中文字，用于语感、空间和动作连续性

### `04_BOUNDARIES.md`
明确：
- 必发生
- 禁止提前释放
- 不可改变的结果
- 章尾精确停笔点
- 不可新增的高等级事实
- 允许 Writer 自由发挥的低等级现场材料

### `characters/`
只放本章出场或本章不读就会写错的角色切片。**不得默认复制完整人物档案中的未来秘密。**

### `rules/`
只放当前章必须使用的世界/能力/设施规则切片。

### `benchmark/`
只放主工作流已经选中的本章技法，不允许 Writer 自己扫描全部拆文库寻找“更好的写法”。

---

## 4. 发布卫生规则

发布新章节前必须：

1. 更新 `input/current/00–04`
2. 更新相关 `characters/`、`rules/`、`benchmark/`
3. 删除上一章遗留的 `input/current/REVISION.md`
4. 清空上一章 `output/current/` 的正文与报告，只保留 `OUTPUT_CONTRACT.md`
5. 确认 Writer 当前任务章节号与主仓库目标章节一致

Writer 默认禁止读取 `archive/`。只有主工作流在当前任务中明确授权某个历史材料时才可读取。

---

## 5. Writer 执行契约

Writer 从根目录 `START_HERE.md` 进入，读取当前任务，输出：

```text
output/current/draft.md
output/current/report.json
```

`report.json` 至少包含：

```json
{
  "chapter": 1,
  "status": "completed",
  "new_characters": [],
  "new_facts": [],
  "uncertain_points": [],
  "deviations": [],
  "proposed_additions": []
}
```

其中：
- `deviations` 不得静默省略偏纲
- 临时功能人物/场景名应进入 `proposed_additions`
- Writer 不得把自己的新增内容直接提升成长期真相

发布后，主工作流进入：

`awaiting_external_writer`

并停止，不得主侧代写正文。

---

## 6. 主侧审稿规则

外部 Writer 返回后，主工作流读取：

- `output/current/draft.md`
- `output/current/report.json`
- 当前章 `01_OUTLINE.md`
- `04_BOUNDARIES.md`
- 当前主仓库 Tracking / 章节承诺

审稿优先级：

1. 剧情契约 / 章尾 / 禁止提前释放
2. 人物知识边界与连续性
3. 主角代理权
4. 对标功能是否兑现
5. 节奏与信息密度
6. 语言自然度与 AI 痕迹

主模型**不得因为“想改得更好”直接重写正文**。

### PASS
正文可直接收编。

### PASS WITH MINOR
只允许机械级修正：明显错字、标点、姓名误写、明确连续性笔误。不得把正文改成主模型自己的文风。

### REVISE
创建：

`input/current/REVISION.md`

只写语义级修订目标，不提供大段替换句，不替 Writer 代写。

Writer 修订后输出：

```text
output/current/draft_v2.md
output/current/report_v2.json
```

此时状态为：

`awaiting_writer_revision`

---

## 7. 最终收编

最终 PASS 后：

1. 选定最终 Writer 版本（如 `draft_v2.md`）
2. 回写主项目 `正文/第{N}章_*.md`
3. 审核 `report*.json` 中所有 proposed additions
   - 接受：写入 Tracking / 设定（按等级）
   - 临时：只进逐章记录/事件记录，不升级长期设定
   - 拒绝：不得进入主真相
4. 更新 Tracking，使正文与记忆完全一致
5. `last_committed_chapter = N`
6. `state_revision += 1`
7. 写入下一章承诺
8. 在 Writer 仓库 `archive/chapter-{NNN}/MANIFEST.md` 登记来源版本、审稿状态、最终采用版本与主仓库目标路径
9. 清空 `output/current/` 正文/报告，只保留 `OUTPUT_CONTRACT.md`

只有完成以上步骤，当前章才算正式完成。

---

## 8. 下一章发布

如果用户请求继续写：

- 读取主仓库最新 Tracking
- 读取下一章细纲
- 重新执行主侧 Context Compiler
- 用上一章**最终 PASS 正文**生成 `03_PREVIOUS_PROSE.md`
- 发布下一章 Workspace
- 再次进入 `awaiting_external_writer`

不得拿旧 GPT 草稿、旧 Writer v1 或未 PASS 版本作为下一章语言连续性来源。

---

## 9. 双仓库不变量

1. **主仓库是唯一真相源。**
2. **Writer 仓库是受控工作台，不是真相源。**
3. Writer 不能直接改主仓库。
4. 主模型不直接代写 Writer 正文。
5. 外部 Writer 不知道未来真相，只知道当前章必要边界。
6. 任何正文版本只有在主侧 PASS + Tracking 同步后才正式生效。
7. 换 Writer 模型、换聊天、上下文清空，都不影响小说，因为状态在仓库中。

---

## 10. V1 验证结论

本协议已通过《规则维修员》第1章、第2章连续测试：

- 第1章完成：主侧规划 → Writer v1 → 主侧语义审稿 → Writer v2 → PASS → 主仓库收编 → Tracking 同步
- 第2章完成：主侧发布第二章工作区 → 外部 Writer 可读取上一章最终版本并独立生成正文

因此从 V1 起，`story-long-write` 的正文执行默认采用本桥接协议。
