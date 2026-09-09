# writer-handoff-state.md：双仓交接状态协议

## 目的

把 Main ↔ Writer 的当前交接状态持久化到 GitHub，避免跨聊天后只能靠上下文猜当前进行到哪一步。

状态文件：

`kq-story-writer/input/current/HANDOFF_STATE.json`

Writer 不修改 input；状态切换由 Main 负责写入。

---

## Schema

```json
{
  "schema": "kq-writer-handoff/v2",
  "project": "书名",
  "chapter": 1,
  "run_type": "create|revision",
  "status": "awaiting_external_writer",
  "writer_attempt": 1,
  "delivery_mode": "CHECKPOINTED|ONE_SHOT",
  "delivery_phase": "FRONT|COMPLETE|null",
  "published_at": "ISO-8601",
  "main_source_revision": "optional",
  "writer_repo_base_commit": "optional",
  "revision_base": null,
  "checkpoint": null,
  "expected_output": {
    "draft": "output/current/draft.md",
    "report": "output/current/report.json"
  },
  "note": "optional"
}
```

### delivery 字段

- `delivery_mode=CHECKPOINTED`：默认长篇第一稿，两段交付。
- `delivery_mode=ONE_SHOT`：用户明确要求一次成文时使用。
- `delivery_phase=FRONT`：只等待 `output/current/segment.md`，它不是正式候选稿。
- `delivery_phase=COMPLETE`：Main 已完成 midpoint checkpoint，Writer 基于已写 segment 继续完成整章。
- revision 不使用 FRONT；`delivery_phase` 可为 `null` 或 `COMPLETE`。

`checkpoint` 仅在 CHECKPOINTED 的 COMPLETE 阶段由 Main 写入，可包含：

```json
{
  "actual": 1500,
  "metric": "visible_chars_v1",
  "remaining_user_range": "1100-1900",
  "front_completed_scope": ["已完成批准情节点简写"],
  "remaining_scope": ["尚待完成批准情节点简写"]
}
```

Writer 不修改这些字段。

`revision_base`：

- FIRST_DRAFT：`null`
- 普通第一次返修：`output/current/draft.md`
- 普通第二次返修：Main 明确指定上一版，例如 `output/current/draft_v2.md`
- 历史章节回炉：`input/current/ORIGINAL_DRAFT.md`

Writer 返修前必须先读取该基线，不得仅凭聊天上下文或 REVISION 摘要重构原稿。

---

## 合法状态

- `published`：Workspace 已写完但尚未正式交给 Writer。
- `awaiting_external_writer`：等待第一稿；CHECKPOINTED 的 FRONT 与 COMPLETE 都使用这一状态，由 `delivery_phase` 区分。
- `reviewing`：Main 已收到完整候选，正在 Review / 裁决。
- `awaiting_writer_revision`：REVISION.md 已发布，等待 Writer 返修。
- `accepted`：最终正文已 PASS，但归档 / Tracking 可能尚未完成。
- `archived`：正文、Tracking、Manifest 与热区清理全部完成。
- `blocked`：输入冲突、缺文件、Tracking 冲突或其他阻塞。

禁止自造近义状态名。

---

## 状态转换

ONE_SHOT：

```text
published
→ awaiting_external_writer
→ reviewing
→ accepted
→ archived
```

CHECKPOINTED：

```text
published
→ awaiting_external_writer + FRONT
→ segment.md 返回
→ Main checkpoint
→ awaiting_external_writer + COMPLETE
→ draft.md + report.json 返回
→ reviewing
→ accepted
→ archived
```

需要返修：

```text
reviewing
→ awaiting_writer_revision
→ reviewing
→ accepted
→ archived
```

任一阶段可进入 `blocked`；修复后回到明确的上一个合法执行状态，不靠聊天记忆猜。

---

## Expected Output

### ONE_SHOT 第一稿

```json
"delivery_mode": "ONE_SHOT",
"delivery_phase": "COMPLETE",
"revision_base": null,
"expected_output": {
  "draft": "output/current/draft.md",
  "report": "output/current/report.json"
}
```

### CHECKPOINTED 第一稿前段

```json
"delivery_mode": "CHECKPOINTED",
"delivery_phase": "FRONT",
"revision_base": null,
"expected_output": {
  "draft": "output/current/segment.md",
  "report": null
}
```

`segment.md` 只是 midpoint checkpoint 材料，不得进入 Review、Tracking 或正式收编。

### CHECKPOINTED 第一稿完整阶段

Main 测完 segment 后原子更新状态：

```json
"delivery_mode": "CHECKPOINTED",
"delivery_phase": "COMPLETE",
"revision_base": null,
"checkpoint": {
  "actual": 1500,
  "metric": "visible_chars_v1",
  "remaining_user_range": "1100-1900",
  "front_completed_scope": [],
  "remaining_scope": []
},
"expected_output": {
  "draft": "output/current/draft.md",
  "report": "output/current/report.json"
}
```

Writer 必须读取现有 `output/current/segment.md`，保持其正文前缀不变，只续写剩余批准内容；最终 `draft.md` = 原 segment 原文 + 后续正文。

### 第一次返修

```json
"revision_base": "output/current/draft.md",
"expected_output": {
  "draft": "output/current/draft_v2.md",
  "report": "output/current/report_v2.json"
}
```

只有 Main 明确授权第二次返修时才使用 `draft_v3.md / report_v3.json`，并显式写出对应 `revision_base`。

---

## 新会话恢复

Main 新会话或 compact 后：

1. 先读主项目 Tracking / `追踪/上下文.md`；
2. 再读 Writer `HANDOFF_STATE.json`；
3. 按 `delivery_mode / delivery_phase / expected_output` 检查对应文件是否存在；
4. CHECKPOINTED + FRONT 且 `segment.md` 已存在 → 不进入 reviewing；先测 checkpoint，再切 COMPLETE；
5. CHECKPOINTED + COMPLETE 且仅有 `segment.md` → 继续等待完整 `draft.md`，不得把 segment 当候选稿；
6. 若是 revision，额外确认 `revision_base` 存在且与本轮被审稿版本一致；
7. 若状态和文件不一致，标记 `blocked` 并先对账，不自动覆盖任何版本。

Examples：

- ONE_SHOT 状态 `awaiting_external_writer` 但 draft 已存在 → 进入 `reviewing` 前先确认 draft 对应当前 chapter。
- CHECKPOINTED FRONT 已有 segment → Main 做 checkpoint，不 review segment。
- CHECKPOINTED COMPLETE 只有 segment → 继续等待，不误判完成。
- 状态 `awaiting_writer_revision` 但 `revision_base` 缺失 → blocked，不让 Writer 猜原文。
- 状态 `awaiting_writer_revision` 但只存在旧 `draft.md` 且 expected output 是 v2 → 继续等待，不把旧稿误当返修稿。
- 状态 `accepted` 但 Tracking 未同步 → 不允许发布下一章。

---

## Archived 状态保留规则

归档完成后，`HANDOFF_STATE.json` **必须保留**，并写成 `status: archived`，用于跨聊天恢复最后一次已完成交接。

因此，任何通用“清理 input/current 热区”的规则都不得在归档收尾时删除这个文件。

正确顺序：

1. 最终 PASS；
2. Tracking 同步；
3. 生成 Archive Manifest；
4. 清理 00-06、REVISION、ORIGINAL_DRAFT、NEXT_CONTEXT、characters/rules/benchmark 与 output 热文件；
5. 保留并更新 `HANDOFF_STATE.json` 为 `archived`。

发布下一任务时，再以新任务内容**原子替换**旧 `HANDOFF_STATE.json`，不得先删除后留下无状态窗口。

本节对任何泛化的“清理 HANDOFF_STATE”描述具有优先权。

---

## 不变量

- 状态文件不是小说真相源。
- 状态文件不存完整正文或完整 Tracking。
- `segment.md` 不是正式候选正文。
- `accepted` 不等于 `archived`。
- 只有 `archived` 的当前章才允许正常发布下一章。
- Revision 必须有可读取的 `revision_base`。
- Writer 不修改 HANDOFF_STATE；Writer 只按当前输入生成 expected output。