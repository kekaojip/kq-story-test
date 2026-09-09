# kq-story-test — 网文写作工具集（通用 Agent / Web AI）

> **FRAMEWORK LOCK：执行任何工作前先读取仓库根目录 `FRAMEWORK_LOCK.md`。除非 KQ 明确批准，禁止修改、删除、改名、移动、重构或实质替换受保护框架；允许在不改变现有行为的前提下做增量扩展。**

本项目按通用文件方式接入 oh-story skills。若当前平台没有 Claude Code / OpenCode / Codex / Antigravity / ZCode / OpenClaw 的 hooks 或 custom agents，仍可直接让 Agent 读取 `skills/*/SKILL.md` 和 `skills/*/references/` 执行；只是运行时硬拦截和多 agent 自动协作不会自动生效。

## Skill 路由表

优先用自然语言点名 skill；如果平台支持自定义命令，也可以把下表映射成命令。

| 意图 | Skill | 说明 |
|------|-------|------|
| 写长篇 / 开书 / 续写 | story-long-write | 长篇网文写作（逐章推进） |
| 写短篇 | story-short-write | 短篇网文写作（情绪驱动） |
| 长篇拆文 | story-long-analyze | 长篇小说深度拆解 |
| 短篇拆文 | story-short-analyze | 短篇小说拆文分析 |
| 长篇扫榜 | story-long-scan | 长篇小说榜单与市场趋势 |
| 短篇扫榜 | story-short-scan | 短篇小说榜单与情绪风口 |
| 去 AI 味 | story-deslop | 去除 AI 写作痕迹；双仓生产闭环默认只作按需 DETECT_ONLY 诊断 |
| 封面 | story-cover | 生成封面图 |
| 审查 | story-review | 多视角审查；无 custom agents 时降级为单线程审查 |
| 导入 | story-import | 逆向导入已有小说到项目结构 |
| 网文工具箱 | story | 模糊意图自动分发 |
| 准备写书 / 部署 | story-setup | 通用项目结构与 skill 使用入口 |
| 浏览器登录态 / 抓取 | browser-cdp | 浏览器 CDP 工具；需平台允许本地脚本/浏览器控制 |

## 文件结构

- `skills/` — 项目本地 skills；Agent 应先读对应 `SKILL.md`，再按需读取 `references/`
- `拆文库/` — 拆文分析结果存放目录
- `{书名}/正文/` — 长篇小说正文章节
- `{书名}/正文.md` — 短篇小说正文
- `{书名}/设定/` — 角色设定、世界设定
- `{书名}/大纲/` — 卷纲、细纲
- `{书名}/追踪/` — `_tracking-state.json` 唯一结构化权威、固定 7 栏 `上下文.md` 续写状态卡、逐章紧凑记录、核心角色独立派生快照、伏笔当前视图、作者与读者双时间线；全部通过追踪工具生成
- `{书名}/对标/` — 对标作品分析

## 通用使用约定

- 写正文前先有大纲：长篇需要 `大纲/细纲_第N章*.md`，短篇需要 `小节大纲.md`。
- 无 hooks 的平台不会自动拦截越权写正文，Agent 必须在执行写作 skill 时自行检查大纲、上下文和追踪文件。
- 无 custom agents 的平台按 solo/direct 执行；遇到 skill 要求调用 story-architect、narrative-writer 等 agent 时，改由当前 Agent 直接完成，并在结果里说明降级。
- **长篇双仓 V2 强制覆盖层**：任何 `story-long-write` 的正文创建、续写、日更、回炉、重写、审稿、去 AI、最终收编或 Tracking 提交，除 `external-writer-bridge.md` 外还必须完整读取 `skills/story-long-write/references/workflow-v2-override.md`。该文件对旧 `workflow-chapter.md / workflow-daily.md / workflow-revision.md` 中冲突的单仓直写、先 Tracking、批末自动改文条款具有覆盖权。文风/作者偏好按 `writer-style-package.md` 编译，跨聊天交接状态按 `writer-handoff-state.md` 持久化。
- **长篇双仓 V2 正文审查自锁**：外部 Writer 返回候选后，先按 `story-long-write/references/external-writer-bridge.md` 运行 `story-review` 只读审稿；只有命中具体 AI / 过度工整 / 解释腔等 prose 病灶时，才调用 `story-deslop` 的“仅标注 / 只检测 / 不要改”模式辅助定位。Main 合并 findings 后只做 `PASS / PASS WITH MINOR / REVISE` 裁决；需要文风/自然度修复时写 `REVISION.md` 交同一 Writer 局部返修。**禁止把旧“每章自动全文去 AI 清零”当作双仓生产默认。** `check-ai-patterns.js` 等仍可作为证据，但 finding 不等于必须修改。
- **短篇与显式独立去 AI 请求**仍按各自 Skill 的原协议执行；本条只覆盖 `story-long-write` 的 External Writer V2 正文闭环。
- Compact / 新会话后优先读取 `{书名}/追踪/上下文.md` 恢复当前写作状态。

## Compact 后恢复上下文

写作中的关键上下文：
1. 当前写作项目名称和进度
2. 最近讨论的角色设定变更
3. 未完成的伏笔列表
4. 当前章节的情绪/节奏目标

如果存在 `{书名}/追踪/上下文.md`，compact 后首先读取该文件恢复上下文。
