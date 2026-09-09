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

只写当前章会改变 Writer 表达选择的维度。没有特殊要求的维度不必凑齐。

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

---

## 4. Writer 权限顺序

表达维度：

`当前 00_TASK 明确要求 > 05_STYLE_RESOLUTION > 06_AUTHOR_PREFERENCES > benchmark > genre/craft/general references`

事实维度仍是：

`04_BOUNDARIES > 01_OUTLINE > 02_CURRENT_STATE > 00_TASK > characters/rules > previous prose > benchmark/execution/style`

Style Package 永远不能覆盖事实权威。

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
- 本书 active 偏好是否正确裁决？
- 是否仍残留 Writer 需要跨仓读取的路径？
- benchmark 的低置信风格是否错误覆盖作者明确偏好？
- 05/06 是否只含表达信息，没有未来剧情或隐藏真相？

任一失败先修 Main 输入，不让 Writer 猜。
