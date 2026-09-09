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
  "published_at": "ISO-8601",
  "main_source_revision": "optional",
  "writer_repo_base_commit": "optional",
  "expected_output": {
    "draft": "output/current/draft.md",
    "report": "output/current/report.json"
  },
  "note": "optional"
}
```

---

## 合法状态

- `published`：Workspace 已写完但尚未正式交给 Writer。
- `awaiting_external_writer`：等待第一稿。
- `reviewing`：Main 已收到候选，正在 Review / 裁决。
- `awaiting_writer_revision`：REVISION.md 已发布，等待 Writer 返修。
- `accepted`：最终正文已 PASS，但归档 / Tracking 可能尚未完成。
- `archived`：正文、Tracking、Manifest 与热区清理全部完成。
- `blocked`：输入冲突、缺文件、Tracking 冲突或其他阻塞。

禁止自造近义状态名。

---

## 状态转换

```text
published
→ awaiting_external_writer
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

### 第一稿

```json
"expected_output": {
  "draft": "output/current/draft.md",
  "report": "output/current/report.json"
}
```

### 第一次返修

```json
"expected_output": {
  "draft": "output/current/draft_v2.md",
  "report": "output/current/report_v2.json"
}
```

只有 Main 明确授权第二次返修时才使用 `draft_v3.md / report_v3.json`。

---

## 新会话恢复

Main 新会话或 compact 后：

1. 先读主项目 Tracking / `追踪/上下文.md`；
2. 再读 Writer `HANDOFF_STATE.json`；
3. 按 `expected_output` 检查对应文件是否存在；
4. 若状态和文件不一致，标记 `blocked` 并先对账，不自动覆盖任何版本。

Examples：

- 状态 `awaiting_external_writer` 但 draft 已存在 → 进入 `reviewing` 前先确认 draft 对应当前 chapter。
- 状态 `awaiting_writer_revision` 但只存在旧 `draft.md` → 继续等待，不把旧稿误当返修稿。
- 状态 `accepted` 但 Tracking 未同步 → 不允许发布下一章。

---

## 不变量

- 状态文件不是小说真相源。
- 状态文件不存完整正文或完整 Tracking。
- `accepted` 不等于 `archived`。
- 只有 `archived` 的当前章才允许正常发布下一章。
- Writer 不修改 HANDOFF_STATE；Writer 只按当前输入生成 expected output。
