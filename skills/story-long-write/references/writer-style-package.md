# writer-style-package.md：Main → Writer 文风与作者偏好编译协议

## 目的

把主仓已经完成的文风裁决与作者 active 偏好压缩成当前章 Writer 可直接执行的表达包，避免 Writer 跨仓读取主小说目录或自行猜测风格。

输出到 Writer：

```text
input/current/05_STYLE_RESOLUTION.md
input/current/06_AUTHOR_PREFERENCES.md
```

这两个文件只控制表达，不拥有剧情、事实、知识边界或 Tracking 权限。

---

## 0. 通用中文语言地基（所有小说默认生效）

Style Package 在编译任何题材、任何书的个性文风之前，必须先保留同一层基础语言地基：

```text
LANGUAGE_BASE: NATURAL_MODERN_CHINESE
GENRE_FLAVOR: LAYERED_ON_TOP
```

默认规则：

- 旁白与贴角色叙述优先使用自然、现代、第一次就能顺着读懂的中文；
- 常见中文搭配优先于模型临时生造的词组、诗化压缩和“看起来更文学”的组合；
- 题材可以改变专名、身份分寸、场景词汇和角色声线，但不得仅因为“修仙 / 古代 / 玄幻 / 悬疑”等标签就把普通叙述自动古文化、文言化或公文化；
- 除非作者、本书文风或当前任务明确要求古典 / 文言 / 特殊实验语体，否则“题材感”不能覆盖自然现代汉语这一地基；
- 普通、顺口、清楚优先于精炼、漂亮、金句化；
- 不为了所谓“人味”机械塞俚语、网络黑话、口头禅或固定口语词；自然白话不等于聊天腔；
- 不把策划、审稿、分析层的抽象术语当成正文声线。类似“杠杆 / 变量 / 压力 / 门槛 / 可验证 / 控制感 / 现实尺子”等词只有在角色、场景本身自然会这样说时才可进入正文。

Main 编译 05 时，如果本书没有更具体的语体覆盖，至少应明确：

```text
base_language: 自然现代汉语；普通顺口优先于文学化加工；题材词可以专业，普通句子不要自动做旧或生造搭配。
```

本节是跨书、跨题材默认值，不应写入某一本测试书的专属设定。

---

## 1. Main 的来源

Main 按现有 `style-resolution.md` 规则读取并裁决：

1. 当前用户请求
2. 本书 `设定/文风.md` / 相关文风设定
3. 本书 active 作者记忆
4. 题材 / 流程 / 全局 active 偏好
5. 当前主 benchmark
6. 通用 reference 默认值

发生冲突时仍按原 style-resolution 的逐维度优先级处理。

---

## 2. `05_STYLE_RESOLUTION.md`

建议格式：

```md
# STYLE RESOLUTION

## ACTIVE
- base_language: 自然现代汉语；普通顺口优先于文学化加工；题材词可以专业，普通句子不要自动做旧或生造搭配。
- POV / narrator_distance: ...
- sentence_rhythm: ...
- paragraphing: ...
- dialogue: ...
- emotion_expression: ...
- exposition: ...
- rhetoric: ...
- punctuation: ...
- ending_behavior: ...

## SOURCE
- current_request: ...
- book_style: ...
- author_memory: ...
- benchmark: ...

## OVERRIDDEN_DEFAULTS
- ...

## MUST_PRESERVE
- 本文件只控制表达；不得改变 00-04、人物、规则和 EXECUTION_CARD 的事实/边界。
```

`base_language` 是跨题材默认地基；只有作者或本书文风明确要求不同语体时才覆盖。其他维度只写当前章会改变 Writer 表达选择的内容，没有特殊要求不必凑齐。

不要把“现代白话”误编译成固定口语配额、对白比例、句长比例、比喻配额或俚语配额。

---

## 3. `06_AUTHOR_PREFERENCES.md`

只放当前任务相关的 active 偏好，目标保持紧凑，默认 ≤2KB。

建议：

```md
# AUTHOR PREFERENCES

- scope: book | global
  kind: prose_style | story_design
  preference: ...
  status: active
```

禁止：

- 把历史已失效偏好一起传入；
- 把工具告警、审稿发现或模型推测写成作者偏好；
- 为了“信息完整”复制整份作者记忆库。

通用语言地基不能依赖 06 才成立；即使当前没有作者记忆，Writer 也必须先服从第 0 节的自然中文默认值。

---

## 4. Writer 权限顺序

表达维度：

`当前 00_TASK 明确要求 > 05_STYLE_RESOLUTION > 06_AUTHOR_PREFERENCES > benchmark > genre/craft/general references`

事实维度仍是：

`04_BOUNDARIES > 01_OUTLINE > 02_CURRENT_STATE > 00_TASK > characters/rules > previous prose > benchmark/execution/style`

Style Package 永远不能覆盖事实权威。

对于没有明确语体覆盖的普通任务，`LANGUAGE_BASE: NATURAL_MODERN_CHINESE` 是通用默认值；genre/craft 只能在其上叠加，不能静默替换。

---

## 5. Revision

返修时继续沿用本轮同一 Style Package，除非：

- 用户明确改变文风要求；
- 本书文风文件在返修前已改变；
- Main 发现旧 Style Package 编译错误。

发生变化时重新编译 05/06，并在 `REVISION.md` 标记原因。

---

## 6. 发布 Gate

发布 Workspace 前检查：

- 当前请求的文风要求是否进入 05？
- 05 是否保留了通用 `base_language`，或存在作者明确覆盖它的证据？
- 本书 active 偏好是否正确裁决？
- 是否仍残留 Writer 需要跨仓读取的路径？
- benchmark 的低置信风格是否错误覆盖作者明确偏好？
- genre / 题材卡是否把普通中文误编译成古风、公文、论文或“小说腔”？
- 05/06 是否只含表达信息，没有未来剧情或隐藏真相？

任一失败先修 Main 输入，不让 Writer 猜。