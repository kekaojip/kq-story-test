# two-ai-hard-gate.md：长篇双 AI 物理隔离硬门禁

状态：ENFORCED
适用：`story-long-write` 的 External Writer 双仓生产链

## 0. 最高规则

长篇正文生产必须由两个**独立 AI 会话**完成，禁止同一个 AI 会话同时扮演 Main 与 Writer。

固定拓扑：

```text
AI-A / Main
= kq-story-test 主工作流
= 规划、真相、Workspace 编译、审稿、Tracking
        ↓ GitHub input/current
人工中转：KQ 复制提示词到另一个 AI 会话
        ↓
AI-B / External Writer
= kq-story-writer
= 只负责正文生成 / 返修
        ↓ GitHub output/current
人工中转：KQ 回到 AI-A
        ↓
AI-A / Main
= Review / Revision / PASS / Tracking / Archive
```

这不是“逻辑角色分工”，而是**物理会话隔离**。

---

## 1. Main 的硬禁止项

只要当前会话是 Main，进入正文阶段后：

- 禁止生成 `segment.md`、`draft.md`、`draft_v2.md`、`draft_v3.md` 的正文内容；
- 禁止“模拟 Writer”“代跑 Writer”“为了测试先写一版”“当前会话直接降级完成”；
- 禁止因为平台没有 custom agents / subagents / hooks，就把 External Writer 降级成当前 Agent；
- 禁止调用 Writer Runtime 后继续在同一会话里生成正文；
- 禁止把 Writer 输出当作自己可以直接润色重写的草稿；
- 禁止在 Workspace 发布后继续执行 Writer 的 FRONT / COMPLETE / REVISION。

如果任何旧文档、fallback、solo/direct 说明允许当前 Agent 代替 `narrative-writer`，在 `story-long-write` External Writer 生产链中一律由本文件覆盖。

### 1.1 Main 对 Writer 仓的访问面只允许当“投递箱 / 收件箱”

Main 可以访问 `kq-story-writer`，但只能为交接做文件操作：

允许：

- 写入 / 更新 `input/current/**`；
- 写入 `HANDOFF_STATE.json`；
- 为发布卫生清理当前任务运行数据；
- Writer 完成后读取 `output/current/**`；
- 最终 PASS 后执行 archive / manifest 所需文件操作。

Main 在正常生产阶段**禁止读取或执行**：

- `kq-story-writer/START_HERE.md`
- `kq-story-writer/skills/**`
- `kq-story-writer/skills/human-writing-l2/**`
- Writer Runtime 的正文生成说明

Main 不需要知道 Writer “怎么写”，只需要按主仓 Bridge 契约“投递什么”和“收回什么”。

如果 Main 为了继续当前章节而开始读取 Writer Skill / Human Writing L2，并准备自己生成正文，视为：

`TWO_AI_ISOLATION_VIOLATION`

必须停止。

### 唯一例外

只有 KQ 在**当前任务中明确说**“这次允许主 AI 直接写正文 / 不走外部 Writer / fallback 直写”，才可绕过本门禁。

没有这句明确授权，就绝不 fallback。

---

## 2. Main 的强制停止点

Main 完成以下动作后必须停止：

1. 完成当前章规划、Truth / Boundary / Style / Execution Card；
2. 把完整 Workspace 推送到 `kekaojip/kq-story-writer/input/current/`；
3. 写好 `HANDOFF_STATE.json`；
4. 状态进入 `awaiting_external_writer` 或 `awaiting_writer_revision`；
5. 确认 Writer 需要的输入已经存在。

此后 Main 不得进入任何 Writer 执行阶段。

Main 的当前回复必须以“外部 Writer 提示词”结束，然后停止。

---

## 3. Main 发布后必须给 KQ 的提示词

Workspace 推送成功后，Main 必须直接给 KQ 一段可完整复制到另一个 AI 会话的提示词。

FIRST_DRAFT 模板：

```text
你现在是独立 External Writer AI。

只读取并操作：
kekaojip/kq-story-writer
branch: <Main 实际发布分支，默认 main>

禁止读取：
kekaojip/kq-story-test
以及任何其他小说仓库、聊天里的旧正文或未来剧情资料。

先读取：
1. FRAMEWORK_LOCK.md
2. EXTERNAL_WRITER_SESSION_LOCK.md
3. START_HERE.md
4. skills/story-writer-runtime/SKILL.md
5. skills/story-writer-runtime/V2_RUNTIME_PATCH.md
6. output/current/OUTPUT_CONTRACT.md
7. input/current/HANDOFF_STATE.json
8. 再按 START_HERE / Runtime 读取 input/current 当前任务。

严格按照 HANDOFF_STATE 的 delivery_mode / delivery_phase / expected_output 执行。
只负责正文生成，不规划下一章，不修改 input，不读取主工作流仓库。

如果是 CHECKPOINTED / FRONT，只生成 segment.md 后停止。
如果是 CHECKPOINTED / COMPLETE，读取并冻结已有 segment.md，再生成完整 draft.md + report.json。
如果是 REVISION，严格按 revision_base + REVISION.md 生成版本化返修稿。

完成后停止，只告诉我：
WRITER_DONE
以及本次实际生成的 output/current 文件名。
```

Main 应根据当前 HANDOFF 动态保留适用阶段，但不得把主仓剧情资料重新塞进提示词。**Writer 的全部任务上下文只来自 Writer 仓 `input/current/`。**

---

## 4. Writer 返回后的恢复规则

KQ 回到 Main 会话并表示 Writer 已完成后，Main 才允许继续：

- 读取 `kq-story-writer/output/current/` 当前 expected output；
- 运行 `story-review`；
- 必要时 `story-deslop DETECT_ONLY`；
- PASS / REVISE 裁决；
- 若 REVISE，则重新发布 `REVISION.md + HANDOFF_STATE`，再次停止并给 KQ 新的 External Writer 提示词；
- 最终 PASS 后才 Tracking / Archive / 下一章。

Main 不得因为“Writer 已返回”而自行改写正文。

---

## 5. 测试与新窗口同样适用

即使用户说“跑完整工作流测试”“端到端测试”“从零开始跑”，也不得由一个 AI 会话把 Main + Writer 一口气跑完。

正确的 E2E 是**跨两个 AI 会话的人类中转 E2E**：

```text
Main 跑到 Workspace 发布
→ STOP + 给提示词
→ KQ 发给另一个 AI
→ Writer 生成
→ KQ 回 Main
→ Main 继续审稿
```

任何“一窗口完整跑完双仓正文”的行为都判定为：

`TWO_AI_ISOLATION_VIOLATION`

并应立即停止，而不是继续生成正文。

---

## 6. 冲突优先级

在 `story-long-write` External Writer 生产链中，本文件对以下旧行为具有强制覆盖权：

- 当前 Agent solo/direct 代替 narrative-writer；
- 无 custom agents 时的单会话降级；
- 为测试方便同会话生成 Writer 输出；
- Workspace 发布后 Main 继续写正文；
- Main 读取 Writer Runtime 后自行执行 Writer。

本门禁不改变 Main / Writer 原有职责，只把已经定义的双仓职责进一步升级为**双会话硬隔离**。
