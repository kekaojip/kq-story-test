# Writer V2 E2E 固定测试夹具

本目录只用于 `story-long-write` 双仓 Writer V2 端到端验收，不是正式小说项目。

## 固定项目

`雾港停电/`

基线状态：

- 已有第 1 章正式正文；
- Tracking 截至第 1 章；
- 第 2 章细纲完整；
- 自定义文风与题材正文提示卡存在；
- 卷纲包含 `D1-01` 单元卡，可走 `outline_view.py --unit D1-01 --stage write`；
- 第 2 章目标字数为 `2600-3400`，适合测试 CHECKPOINTED 前后段交付。

## 不变量

1. `tests/fixtures/writer-v2-e2e/` 是只读基线，不在这里直接做 Tracking commit 或写测试正文。
2. 每次 E2E 测试应在临时/专用测试分支上运行，或先从当前 main 创建测试分支，再在该分支中使用此项目。
3. 原版 `kq-story` 只作对照，不修改。
4. Writer 输出仍写入 `kq-story-writer` 的对应测试分支。
5. 真实测试必须走：Workspace → CHECKPOINT FRONT → midpoint → COMPLETE → report → review → 必要 revision/compress → PASS → Tracking → archive/handoff。

## 测试章节

- 项目：`雾港停电`
- 目标章节：第 2 章
- 批准章名：`广播没有电`
- 单元：`D1-01`
- 题材：都市悬疑 / 封闭空间异常
- POV：第三人称限知，贴陆沉

本夹具的内容全部为合成测试材料，可反复用于版本对照。