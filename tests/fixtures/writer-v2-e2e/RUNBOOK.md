# Writer V2 E2E Runbook

## 目标

使用固定合成项目 `tests/fixtures/writer-v2-e2e/雾港停电/` 对长篇双仓工作流做完整验车。

原版对照：`kekaojip/kq-story@main`，只读。

## 测试分支

每次运行前，把以下固定测试分支重置到各自最新 `main`：

- `kekaojip/kq-story-test@test/writer-v2-e2e`
- `kekaojip/kq-story-writer@test/writer-v2-e2e`

所有本轮 Workspace、正文、Tracking、Archive、Handoff 写入都只能发生在这两个测试分支；不要修改两个仓库的 `main`。

## 固定项目

主仓测试项目：

`tests/fixtures/writer-v2-e2e/雾港停电/`

固定目标：

- 当前已提交：第1章
- 本次写：第2章《广播没有电》
- 单元：D1-01
- 字数范围：2600-3400，`visible_chars_v1`
- 默认模式：`CHECKPOINTED`
- POV：第三人称限知，贴陆沉

不得另找项目，也不得新建另一部测试小说。

## 运行前框架读取

主仓读取：

- `FRAMEWORK_LOCK.md`
- `AGENTS.md`
- `skills/story-long-write/SKILL.md`
- `skills/story-long-write/references/external-writer-bridge.md`
- `skills/story-long-write/references/workflow-v2-override.md`
- `skills/story-long-write/references/writer-handoff-state.md`
- `skills/story-long-write/references/writer-style-package.md`
- `skills/story-long-write/references/writer-execution-card.md`

Writer 读取：

- `FRAMEWORK_LOCK.md`
- `START_HERE.md`
- `skills/story-writer-runtime/SKILL.md`
- `skills/story-writer-runtime/V2_RUNTIME_PATCH.md`
- `output/current/OUTPUT_CONTRACT.md`
- `skills/human-writing-l2/SKILL.md`

不得修改框架文件。

## 基线门槛

开始正文前至少核验：

1. `tracking_commit.py check` 的语义等价检查：Tracking state 与派生视图一致。
2. `check-outline-contract.js` 对第2章细纲通过 blocking 项。
3. `outline_view.py --unit D1-01 --stage write` 能得到卷级常任 + D1-01 单元级材料。
4. 第1章正文存在，可用于 `03_PREVIOUS_PROSE.md` 和 `heading_literal` 继承。
5. `设定/文风.md`、`设定/题材正文提示卡.md`、单元情绪引擎、章功能分配存在，允许走成熟项目召回降档。

若 fixture 自身基线不满足，报告 `FIXTURE_INVALID`，不要靠杜撰修过去。

## 正文完整闭环

严格运行：

```text
Main 读取细纲 / Tracking / 人物 / 规则 / 文风
→ 编译 EXECUTION_CARD + Style Package + Workspace
→ HANDOFF: CHECKPOINTED / FRONT
→ Writer 只写 segment.md
→ Main midpoint visible_chars_v1 checkpoint
→ HANDOFF: CHECKPOINTED / COMPLETE
→ Writer 冻结 segment 前缀并完成 draft.md + report.json
→ deterministic preflight
→ story-review（只诊断）
→ 必要时 story-deslop DETECT_ONLY
→ Main PASS / PASS WITH MINOR / REVISE
→ 必要时 Writer versioned revision
→ 如果最终完整稿 over 且剧情/边界正确，可测试一次 COMPRESS_ONCE
→ 最终 PASS
→ 主仓正式正文 + Tracking append
→ Writer Archive Manifest
→ HANDOFF_STATE = archived
```

### FRONT 约束

- 只产出 `output/current/segment.md`
- 不生成最终 report
- 不 Review
- 不 Tracking
- 在自然场景/因果停顿处停止

### COMPLETE 约束

- 必须读取既有 `segment.md`
- segment 为冻结前缀，不回改
- 只完成 remaining_scope
- 最终生成 `draft.md + report.json`

## 完整 report 必查

至少确认：

- `chapter`
- `status`
- `new_characters`
- `new_facts`
- `uncertain_points`
- `deviations`
- `outline_coverage`
- `unwritten`
- `references_read`
- `proposed_additions`
- `preflight`

Main 必须用真实正文复核 `outline_coverage`，Writer 自报 landed 不等于 PASS。

## 标题

第1章基线标题是：

`# 第1章 停电之后`

第2章应继承同一 heading shape，得到：

`# 第2章 广播没有电`

## COMPRESS_ONCE

只在真实完整稿测得 `over` 且剧情覆盖、Truth、Boundary 已正确时触发。不得故意写超长来测试它。

若未触发，报告 `NOT_TRIGGERED`。

## 原版功能对照

测试结束后，对照 `kq-story@main` 的原始单章写作功能，检查当前双仓是否仍承载：

- 细纲/卷纲/Tracking 输入
- 文风与作者偏好
- benchmark/题材召回
- checkpoint 两段交付
- 情节点覆盖证明
- 本章没写成的反馈
- references read 可观测性
- 新增物申报
- heading shape
- 字数与 compress-once
- 审稿/去AI诊断
- Tracking/伏笔/时间线
- 跨聊天 handoff/archive

只有“原版存在且当前找不到任何等价承载点”才记为功能遗漏。

## 最终报告格式

```text
【测试章节】
项目：雾港停电
章节：第2章 广播没有电

【基线】
FIXTURE: PASS / FAIL
OUTLINE CONTRACT: PASS / FAIL
TRACKING BASELINE: PASS / FAIL
OUTLINE VIEW: PASS / FAIL

【流程结果】
WORKSPACE: PASS / FAIL / BLOCKED
CHECKPOINT FRONT: PASS / FAIL / NOT_RUN
MIDPOINT WORDCOUNT: PASS / FAIL / NOT_RUN
CHECKPOINT COMPLETE: PASS / FAIL / NOT_RUN
OUTLINE COVERAGE: PASS / FAIL / NOT_RUN
UNWRITTEN FEEDBACK: PASS / FAIL / NOT_RUN
REFERENCES READ: PASS / FAIL / NOT_RUN
HEADING LITERAL: PASS / FAIL / NOT_RUN
PREFLIGHT: PASS / FAIL / NOT_RUN
STORY REVIEW: PASS / FAIL / NOT_RUN
DESLOP DETECT_ONLY: PASS / NOT_TRIGGERED / FAIL / NOT_RUN
REVISION: PASS / NOT_TRIGGERED / FAIL / NOT_RUN
COMPRESS_ONCE: PASS / NOT_TRIGGERED / FAIL / NOT_RUN
TRACKING AFTER PASS: PASS / FAIL / NOT_RUN
ARCHIVE / HANDOFF: PASS / FAIL / NOT_RUN

【发现的问题】
只列真实运行问题。

【功能遗漏】
原版功能：
当前缺口：
严重级别：
建议：

若没有：未发现新的功能遗漏。
```

`NOT_RUN` 不能写成 `FAIL`；只有实际执行并失败才写 `FAIL`。
