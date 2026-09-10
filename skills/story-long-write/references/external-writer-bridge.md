# external-writer-bridge.md：外部正文 Writer 双仓库桥接协议（V2）

> 本协议是 `story-long-write` 的正文执行层权威覆盖协议。
>
> **上游主工作流继续负责：** 选题、拆文、设定、人物、全书/卷/章规划、对标召回、状态筛选、边界裁决、审稿、Tracking、最终入库。
>
> **外部 Writer 只负责：** 把已经确定的章节语义写成正文，并按要求返修。
>
> **V2 新增：** `EXECUTION_CARD`、Writer execution slices、`story-review` 正式审稿节点、`story-deslop DETECT_ONLY` 专项诊断、确定性只读预检。
>
> **通用语言补丁：** 所有题材先服从自然中文底线；`EXECUTION_CARD` 只传语义不传措辞；完整稿默认运行只读 `language_lint`，Main 仍负责最终中文自然度语义裁决。

---

## 0. 默认拓扑

- 主工作区 / 唯一真相源：当前小说项目所在仓库（本测试环境为 `kekaojip/kq-story-test`）
- 外部 Writer 仓库：`kekaojip/kq-story-writer`
- Writer Skill：`skills/story-writer-runtime/`
- Writer 当前输入：`input/current/`
- Writer 当前输出：`output/current/`
- 主侧审稿 Skill：`skills/story-review/`
- 主侧去 AI 专项诊断：`skills/story-deslop/`，只用 `DETECT_ONLY`

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
- 悬念 / 战斗 / 钩子 / 反转 / 情绪的设计结果
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
- 把 `EXECUTION_CARD` 中已经批准的专项执行意图写到位

Writer **不得**直接修改主仓库大纲、人物设定、世界规则、Tracking、未来剧情或真相文件。

Writer 的表达权不包括继承上游策划措辞：`EXECUTION_CARD`、细纲、review 标签里的分析词只传递意思，正文必须重新使用自然中文成文。

### Reviewer / Deslop 只拥有诊断权

- `story-review`：找问题、分级、给修复方向；不直接改 Writer 正文。
- `story-deslop DETECT_ONLY`：只在需要时定位具体 AI/表面病灶；不直接改正文、不写回文件。
- 最终是否需要返修、返修哪些问题，由 Main 裁决。

---

## 2. 写正文前的主侧准备

仍先完整执行 `workflow-chapter.md` 的：

1. 细纲检查
2. 卷纲 write-stage 取段
3. Tracking / 当前状态筛选
4. 相关人物、规则、势力读取
5. 对标召回与文风裁决
6. Constraint Lock
7. 按 `writer-execution-card.md` 把已批准的正文执行决定编译成 `EXECUTION_CARD`
8. 按 `writer-style-package.md` 编译当前 `05_STYLE_RESOLUTION.md / 06_AUTHOR_PREFERENCES.md`

当这些内容都已确定后，**不要直接生成正文**，而是编译为 Writer Workspace。

通用要求：即使当前书没有特殊作者偏好或 benchmark，Style Package 也必须保留自然现代汉语这一默认语言地基；题材风格只能在其上叠加。

---

## 3. Writer Workspace 固定结构

发布到 `kq-story-writer/input/current/`：

```text
00_TASK.md
01_OUTLINE.md
02_CURRENT_STATE.md
03_PREVIOUS_PROSE.md
04_BOUNDARIES.md
05_STYLE_RESOLUTION.md
06_AUTHOR_PREFERENCES.md
characters/
rules/
benchmark/
  EXECUTION_CARD.md   # V2，可选但推荐；至少有 PROSE CORE 时生成
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

### `05_STYLE_RESOLUTION.md`
Main 预编译的当前表达裁决。没有明确特殊语体覆盖时必须保留 `base_language: 自然现代汉语` 的默认地基；不得把题材标签自动翻译成古风、公文、论文或生造小说腔。

### `06_AUTHOR_PREFERENCES.md`
只放当前任务相关的 active 作者偏好。为空不代表自然中文底线失效。

### `characters/`
只放本章出场或本章不读就会写错的角色切片。**不得默认复制完整人物档案中的未来秘密。**

### `rules/`
只放当前章必须使用的世界/能力/设施规则切片。

### `benchmark/`
只放主工作流已经选中的本章技法，不允许 Writer 自己扫描全部拆文库寻找“更好的写法”。

### `benchmark/EXECUTION_CARD.md`
按 `references/writer-execution-card.md` 编译。

它只传递主模型已经做完的当前章执行决定，例如：

- `PROSE CORE`
- `SUSPENSE`
- `COMBAT`
- `HOOK`
- `REVERSAL`
- `EMOTION`
- `DIALOGUE`

不存在的模块不生成。不得把完整设计方法论原样复制给 Writer。

卡片必须保留 `SEMANTIC ONLY` 措辞隔离：策划/分析用词不是正文语言样本。

---

## 4. 发布卫生规则

发布新章节前必须：

1. 更新 `input/current/00–06`
2. 更新相关 `characters/`、`rules/`、`benchmark/`
3. 重新生成当前章 `EXECUTION_CARD`；禁止沿用上一章卡片
4. 删除上一章遗留的 `input/current/REVISION.md`
5. 清空上一章 `output/current/` 的正文与报告，只保留 `OUTPUT_CONTRACT.md`
6. 确认 Writer 当前任务章节号与主仓库目标章节一致

Writer 默认禁止读取 `archive/`。只有主工作流在当前任务中明确授权某个历史材料时才可读取。

---

## 5. Writer 执行契约

Writer 从根目录 `START_HERE.md` 进入，读取当前任务，输出：

```text
output/current/draft.md
output/current/report.json
```

Writer Runtime V2 默认：

- `Human Writing L2 / FIRST_DRAFT` 先执行 `Natural Chinese Floor`，再控制自然成文与完成度波动；
- 根据 `EXECUTION_CARD` 模块按需加载 execution slices；
- 写完完整稿后运行 `check-outline-copy`、`check-degeneration`、`visible_chars_v1` 字数测量、`check-ai-patterns` 只读 `language_lint`；
- 这些预检只提供 evidence，不自动触发全章去 AI 清洗；
- `global_ai_wash` 继续保持默认关闭。

`report.json` 至少包含：

```json
{
  "chapter": 1,
  "status": "completed",
  "new_characters": [],
  "new_facts": [],
  "uncertain_points": [],
  "deviations": [],
  "proposed_additions": [],
  "preflight": {
    "language_lint": {
      "status": "clean|findings|not_run",
      "blocking_count": 0,
      "advisory_count": 0,
      "summary": "..."
    }
  }
}
```

其中：
- `deviations` 不得静默省略偏纲
- 临时功能人物/场景名应进入 `proposed_additions`
- Writer 不得把自己的新增内容直接提升成长期真相
- `preflight` finding 只是诊断证据，不等于自动改文命令
- 完整稿若 `language_lint` 未运行，必须明确 `not_run + 原因`，不能伪装成 clean
- `language_lint=clean` 不代表中文搭配语义审查自动通过，Main / Reviewer 仍需通读自然度

发布后，主工作流进入：

`awaiting_external_writer`

并停止，不得主侧代写正文。

---

## 6. V2 主侧审稿闭环

外部 Writer 返回后，主工作流读取：

- `output/current/draft.md`
- `output/current/report.json`
- 当前章 `01_OUTLINE.md`
- `04_BOUNDARIES.md`
- `05_STYLE_RESOLUTION.md`
- `06_AUTHOR_PREFERENCES.md`
- `EXECUTION_CARD.md`（若存在）
- 当前主仓库 Tracking / 章节承诺

### 6.1 先运行 `story-review`

`story-review` 是正式 Reviewer，不是改稿器。

默认目标：对当前 Writer 候选进行章节级审查；运行时支持 lean 时优先 lean，不支持子代理或复杂模式时 fallback 到 solo/direct，但**无论模式都只输出 findings，不修改正文**。

审稿优先级：

1. 剧情契约 / 章尾 / 禁止提前释放
2. 人物知识边界与连续性
3. 主角代理权
4. `EXECUTION_CARD` 的批准功能是否兑现
5. 对标功能是否兑现
6. 节奏与信息密度
7. **中文自然度 Gate：搭配、句法、策划语泄漏、诗化压缩、题材导致的普通句做旧**
8. 对话、人物行为、其他正文自然度
9. AI 痕迹 / 模型退化 / 格式

中文自然度是独立验收项，不因为剧情、Tracking、边界全部正确而自动通过。

必须检查：

- 句子是否语义能解释但不像自然中文会这样搭；
- 是否需要读者频繁在脑中改写成另一句中文才能顺读；
- 是否把策划、论文、项目管理或审稿层术语扩散成叙述声线；
- 是否为了题材感、文学感把普通叙述自动古文化、公文化、翻译化或诗化压缩；
- 是否连续制造“漂亮闭合 / 金句 / 段尾落锤”打断阅读；
- 是否把必要的自然连接过度删除，形成提纲/电报体。

同类问题成片出现并明显增加阅读负担时，至少按 S2 处理并 `REVISE`。现有 detector 没命中，不得作为中文自然度 PASS 的充分理由。

Writer `report.preflight.language_lint` 同时作为正式审稿输入：

- `blocking_count > 0` 必须被 Reviewer / Main 逐项裁决，未处理或未明确说明合法豁免时不得直接 PASS；
- advisory 只作通读提示，不机械升级；
- 不因为 findings 自动改稿，修复仍走 `REVISION.md`。

Reviewer findings 使用 `story-review` 自身统一 schema：

```yaml
- severity: S1 | S2 | S3 | S4
  category: structure | character | prose | consistency | platform | factual | format | causal | rule_boundary
  location: 文件路径:行号 或 章节/段落描述
  evidence: "证据"
  issue: "问题"
  fix: "修复方向"
```

### 6.2 仅在需要时运行 `story-deslop DETECT_ONLY`

触发条件：

- `story-review` 出现明确 `prose` / AI-naturalness / 中文搭配自然度 / 过度工整 / 解释腔 / 模型句式类 finding；
- Writer `preflight` 或作者明确指出 AI 表面病灶；
- Main 需要把模糊的“太 AI / 不像人话”定位成具体局部问题。

调用必须明确：

> **仅标注 / 只检测 / 不要改。**

只执行 `story-deslop` 的扫描与诊断阶段：

- 不进入逐项清除；
- 不自动写回正文；
- 不运行“全文三遍清洗”作为生产默认；
- 输出位置、Gate / 类型、证据、建议局部动作。

若没有具体自然度问题，不调用 Deslop。

### 6.3 Main 合并 findings 并裁决

Main 合并：

```text
Writer report / preflight
+ story-review findings
+ 必要时 story-deslop DETECT_ONLY findings
+ Main 的 Truth / Boundary 裁决
```

然后只允许：

- `PASS`
- `PASS WITH MINOR`
- `REVISE`

主模型**不得因为“想改得更好”直接重写正文**。

### PASS
正文可直接收编，但必须同时满足 Truth / Boundary 与中文自然度 Gate；`language_lint` blocking 未裁决时不得 PASS。

### PASS WITH MINOR
只允许机械级修正：明显错字、标点、姓名误写、明确连续性笔误。不得把正文改成主模型自己的文风；成片中文自然度问题不能降级为 MINOR。

### REVISE
创建：

`input/current/REVISION.md`

每条修订目标建议使用：

```text
LOCATION:
DEFECT:
MUST_PRESERVE:
TARGET:
SCOPE:
```

要求：

- 只写具体 defect 与语义级修订目标；
- 不提供大段成品替换句；
- `MUST_PRESERVE` 明确不能被返修破坏的剧情/语气/事实；
- `SCOPE` 限制在命中段落及必要邻接；
- 多个 finding 先由 Main 去重、合并、排序，不把 Reviewer 原报告整包倒给 Writer。

Writer 修订时：

- 生造搭配 / 翻译式组合 / 诗化压缩 / 策划语泄漏 / 自然度 / 过度完成 / 对白说满 / 局部模型腔 → `Human Writing L2 LOCAL_REVISION`
- 真值 / 边界 / 连续性 / 格式 → 最小修复
- 涉及悬念、战斗、钩子、反转执行 → 继续服从原 `EXECUTION_CARD`，不得重设计

Writer 修订后输出：

```text
output/current/draft_v2.md
output/current/report_v2.json
```

此时状态为：

`awaiting_writer_revision`

V2 默认最多一次 Writer 修订；只有 S1/S2 仍未解决时才考虑第二次，不为纯文风偏好无限循环。

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
- 重新编译该章 `EXECUTION_CARD`
- 用上一章**最终 PASS 正文**生成 `03_PREVIOUS_PROSE.md`
- 发布下一章 Workspace
- 再次进入 `awaiting_external_writer`

不得拿旧 GPT 草稿、旧 Writer v1 或未 PASS 版本作为下一章语言连续性来源。

---

## 9. 双仓库不变量

1. **主仓库是唯一真相源。**
2. **Writer 仓库是受控工作台，不是真相源。**
3. Writer 不能直接改主仓库。
4. Main / Reviewer / Deslop 不直接代写 Writer 正文。
5. 外部 Writer 不知道未来真相，只知道当前章必要边界。
6. `EXECUTION_CARD` 只传已批准执行决定，不新增真相，且策划措辞不得直接继承为正文声线。
7. `story-review` 只诊断；`story-deslop` 在生产闭环中默认只 DETECT_ONLY。
8. Human Writing L2 负责第一稿自然成文与自然中文底线；Deslop 不得反向恢复全章自动 AI wash。
9. 完整稿默认只读 `language_lint`；它负责体检，不拥有自动改文权，也不替代人工中文自然度 Gate。
10. 任何正文版本只有在主侧 PASS + Tracking 同步后才正式生效。
11. 换 Writer 模型、换聊天、上下文清空，都不影响小说，因为状态在仓库中。

---

## 10. 回滚与基线

V2 改造前主仓基线分支：

`backup/pre-writer-v2-20260910`

Writer 仓同名基线分支：

`backup/pre-writer-v2-20260910`

本次自然中文修复前双仓基线分支：

`backup/pre-natural-chinese-fix-20260910`

若本次修复导致正文过度口语化、指标化或产生新模板，可按双仓该分支回滚；不得恢复 global AI wash 来替代语言体检。

若 V2 A/B 表现不如旧 Runtime，可以整体回滚；不得删除旧 `story-review`、`story-deslop` 或 Human Writing L2 历史版本来“简化”架构。

---

## 11. V1 验证历史

V1 曾通过《规则维修员》第1章、第2章连续测试，证明双仓 Writer 架构可工作。

V2 在 V1 基础上补回：正文执行知识、Review / Deslop 诊断闭环与 Main → Writer 执行意图编译层。当前通用语言补丁进一步补回“自然中文底线 + 默认只读语言体检 + Main 语义 Gate”，不改变双仓职责边界。