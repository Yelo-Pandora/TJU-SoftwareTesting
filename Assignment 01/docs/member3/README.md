# Member3 使用说明

本目录是成员3的交付物说明，目标是让组员能够快速完成环境配置，并基于 member1 的黑盒测试产出继续完成执行层转换工作。

成员3的核心职责不是重新生成测试用例，而是将上游已经生成好的黑盒测试结果，进一步整理为可落地的自动化执行资产，例如：

- API 方向：Postman / Newman 可执行请求与断言
- UI 方向：Playwright 可执行测试步骤与断言

---

## 1. 目录与上下游关系说明

本项目中，成员3的工作依赖以下上游材料：

- `../member1/handoff-member3-execution.md`
  - member1 给 member3 的正式交接文档
- `../member1/02-io-templates-freeze.md`
  - 冻结的 I/O 契约
- `../member1/experiments/real-input-pack.md`
  - 真实输入包（R1-R12）
- `../../blackbox-testing/skills/SKILL3.md`
  - 为 member3 编写的执行转换 skill

职责分工建议理解为：

- member1：负责提示词、输出结构、黑盒测试设计
- member2：负责输入整理与需求标准化
- member3：负责把测试设计转换为可执行测试资产
- member4：负责评估与指标统计

因此，member3 的输入应优先使用 member1 产出的标准化黑盒结果，而不是直接从零开始写测试脚本。

---

## 2. 运行前准备

### 2.1 必需环境

建议至少准备以下环境：

1. 已安装 `claude` CLI
2. 已安装并可用 `ccr`（claude-code-router）
3. 已配置可用 API key（通过环境变量注入）
4. 已安装 `node` 与 `npm`
5. 如需执行 API 自动化，建议安装 `newman`
6. 如需执行 UI 自动化，建议安装 `Playwright`

### 2.2 推荐环境检查

可以先执行以下命令检查 Claude Code 与路由环境是否正常：

```bash
source ~/.zshrc
ccr status
claude --version
node -v
npm -v
```

如果 `ccr status` 显示未运行，可先执行：

```bash
ccr start
```

### 2.3 Newman 环境准备

若你要把 API 用例转换为 Postman / Newman 资产，建议先确认 Newman 可用：

```bash
newman -v
```

如果本机未安装，可以使用：

```bash
npm install -g newman
```

仓库中已有可参考的 Newman 执行方式，例如：

- `../../codebases/realworld/implementations/golang-gin/scripts/run-api-tests.sh`
- `../../codebases/realworld/implementations/golang-hexagonal/tests/run-api-tests.sh`

这些脚本说明了实际执行时通常需要：
- APIURL
- 测试账号邮箱
- 用户名
- 密码
- 请求延迟等全局参数

### 2.4 Playwright 环境准备

若你要把 UI 用例转换为 Playwright 资产，建议先安装 Playwright：

```bash
npm install -D @playwright/test
npx playwright install
```

仓库中的 RealWorld 规范已给出可参考的 Playwright 基础配置与测试集：

- `../../codebases/realworld/specification/upstream/specs/e2e/playwright.base.ts`
- `../../codebases/realworld/specification/upstream/docs/src/content/docs/specifications/frontend/tests.md`

其中已有明确提示：
- 需要为前端实现提供合适的 `baseURL`
- 需要配置 `webServer`
- 需要遵循可观测的 selector / route / text contract

如果只是做“执行转换文档”而不是立刻跑测试，也建议先把 Playwright 环境装好，方便后续快速落地。

---

## 3. member3 的输入要求

member3 最推荐接收的输入，是 member1 定义的 `GeneratedBlackboxOutputV1` 输出格式。

该格式必须保持以下 7 个 section（顺序固定）：

1. `Feature Summary`
2. `Requirements Extracted`
3. `Test Design Strategy`
4. `Test Scenarios`
5. `Detailed Test Cases`
6. `Coverage Summary`
7. `Ambiguities / Missing Information / Assumptions`

并且 `Detailed Test Cases` 至少需要包含以下字段：

- `Test Case ID`
- `Title`
- `Requirement Reference`
- `Preconditions`
- `Test Data`
- `Steps`
- `Expected Result`
- `Priority`
- `Risk/Notes`

如果输入不满足以上结构，member3 不应直接假装可以执行，而应该先标记结构问题并要求补齐。

---

## 4. member3 的工作目标

成员3的目标是把“测试设计”继续转成“可执行说明”或“可执行脚本骨架”。

### 4.1 API 方向

对于 API 用例，重点是把黑盒用例映射为：

- 环境准备（env / token / 数据准备）
- 请求方法、路径、参数、请求体
- 请求顺序
- 响应断言
- 清理动作（如需要）

映射关系建议统一为：

- `Preconditions` -> 环境、鉴权、数据准备
- `Test Data` -> query / path / header / body 数据
- `Steps` -> 请求顺序与执行步骤
- `Expected Result` -> 状态码、响应体、字段、错误信息等断言

### 4.2 UI 方向

对于 UI 用例，重点是把黑盒用例映射为：

- 页面初始状态
- 账号状态 / fixture 状态
- 用户操作序列
- 页面可见行为断言
- 路由变化 / 列表状态 / 表单状态断言

映射关系建议统一为：

- `Preconditions` -> 账号状态、页面状态、前置数据
- `Test Data` -> 表单输入、路由参数、显示数据
- `Steps` -> 操作步骤
- `Expected Result` -> locator 断言、UI 状态断言、持久化结果断言

---

## 5. 推荐使用方式

### 方式A：使用 SKILL3 做执行转换

推荐直接使用以下 skill：

- `../../blackbox-testing/skills/SKILL3.md`

该 skill 的作用是：
- 读取 member1 风格的黑盒输出
- 校验结构是否可消费
- 将测试用例分类为 API / UI / Hybrid
- 生成可直接转写为 Newman / Playwright 的执行资产说明
- 标记 blocked / clarification 问题

推荐输入材料：
- member1 生成的完整黑盒输出
- `real-input-pack.md` 中的原始 requirement ID 参考
- 目标执行方向（API / UI / both）
- 如有的话，再补充 baseURL、认证方式、现有脚本目录

### 方式B：手工按交接文档执行转换

如果暂时不依赖 skill，也可以手工按以下文档做：

- `../member1/handoff-member3-execution.md`

执行时至少保证：
- 不丢失 `Requirement Reference`
- `Expected Result` 被改写为显式断言
- 输入不足时标记 blocked，而不是自行补全未知行为
- API / UI 场景分别输出，不混在一起

---

## 6. 推荐工作流程

建议按以下流程开展 member3 工作：

### 第一步：确认输入结构合法

检查上游输出是否满足 7 段结构，以及 `Detailed Test Cases` 字段是否齐全。

若不合法，优先回退给 member1 / member2 修正，不要直接进入脚本开发。

### 第二步：建立执行清单

为每条测试用例补充以下信息：

- `Test Case ID`
- `Requirement Reference`
- `Target`（API / UI / Hybrid）
- `Priority`
- `Status`（Ready / Blocked / Needs Clarification）
- `Notes`

这样便于后续统计执行覆盖率与阻塞项。

### 第三步：完成执行映射

将每条用例翻译为：

- setup
- data
- action / request
- assertion
- cleanup

其中最关键的是：
- 把模糊的 `Expected Result` 改写为可验证断言
- 但只在输入有依据时才细化，不能凭空编造

### 第四步：分别输出 API / UI 资产

建议最终至少拆成两部分：

- API Automation Assets
- UI Automation Assets

这样更方便分别交给 Newman 和 Playwright 落地。

### 第五步：汇总 blocked 问题

以下情况应优先列为 blocked 或待确认：

- 没有 `Requirement Reference`
- `Expected Result` 太模糊，无法断言
- 缺少接口路径、请求字段、认证方式
- 缺少页面语义、路由信息、可定位元素信息
- 多条用例高度重复，需合并

---

## 7. 真实输入与建议练习范围

本项目推荐从 `../member1/experiments/real-input-pack.md` 开始练习，该输入包包含 `R1-R12`，覆盖：

### API 场景
- 用户注册 / 登录
- 当前用户信息获取与更新
- 文章创建、修改、删除
- 收藏 / 关注切换
- 评论创建与删除
- 过滤与分页
- 401 / 403 / 404 / 422 / 409 等错误响应

### UI 场景
- Todo 创建 / 编辑 / 删除
- 清空已完成
- 全选状态同步
- 路由过滤：`#/`、`#/active`、`#/completed`
- 空输入与 trim 规则

建议优先挑选：
- 1~2 个高优先 API 用例
- 1~2 个高优先 UI 用例

先完成从黑盒输出到执行资产的完整样例，再逐步扩展到全量。

---

## 8. 输出质量要求

member3 的输出建议满足以下标准：

1. **可追踪**
   - 每个执行资产都能对应原始 `Requirement Reference`

2. **可执行**
   - 至少能直接转写成 Newman / Playwright 的步骤和断言

3. **不编造**
   - 不自行发明接口、选择器、鉴权规则、内部状态

4. **保留边界/异常用例**
   - 执行转换阶段不能把边界值、非法输入、顺序状态类用例丢掉

5. **显式标记阻塞**
   - 无法确定的信息必须列出，不可沉默处理

---

## 9. 常见问题排查

### Q1：上游输出格式不稳定，无法继续转换怎么办？

建议：
- 先对照 `../member1/02-io-templates-freeze.md` 检查结构
- 再对照 `../../blackbox-testing/skills/SKILL3.md` 检查输入要求
- 若 section 或字段缺失，先回退修正，再继续执行转换

### Q2：API 用例缺少明确 endpoint 或请求字段怎么办？

建议：
- 先标记为 `Blocked`
- 在备注中明确缺少的字段
- 必要时回查 RealWorld 规范或上游 requirement source
- 不要自己猜测接口定义

### Q3：UI 用例无法直接写 locator 怎么办？

建议：
- 优先使用用户可观察的文本、标签、角色语义
- 如果规格中没有足够信息，就保留 placeholder 并标注待确认
- 不要随意编造 CSS/XPath

### Q4：两条测试用例内容很像，是否可以合并？

建议：
- 只有在“测试意图 + 数据类型 + 预期结果”都相同的情况下才合并
- 如果一条是边界值、一条是异常值，通常不应合并

---

## 10. 与组员协作建议

- 与 member1 对齐：确认输出结构、字段命名、歧义记录方式
- 与 member2 对齐：确认 requirement ID、输入来源、是否已标准化
- 与 member4 对齐：确认执行结果将如何进入 coverage / executability 统计

特别注意：
- member3 不应私自修改上游字段名
- member3 不应移除 requirement traceability
- 若发现输入契约不稳定，应尽早反馈，而不是在脚本阶段硬修

---

## 11. 提交前检查建议

提交前建议至少确认：

- [ ] 已具备 Claude Code / ccr / Node 基础环境
- [ ] API 方向需要的 Newman 可运行
- [ ] UI 方向需要的 Playwright 可运行
- [ ] 已阅读 `handoff-member3-execution.md`
- [ ] 已确认输入满足 7 段结构与字段契约
- [ ] 已保留全部 `Requirement Reference`
- [ ] 已对 blocked / ambiguity 问题单独列出
- [ ] 已能给出至少一组 API 执行样例
- [ ] 已能给出至少一组 UI 执行样例

---

## 12. 最短上手路径

如果你想最快开始 member3 工作，建议按下面顺序：

1. 阅读 `../member1/handoff-member3-execution.md`
2. 阅读 `../member1/02-io-templates-freeze.md`
3. 阅读 `../member1/experiments/real-input-pack.md`
4. 使用 `../../blackbox-testing/skills/SKILL3.md` 对上游黑盒输出做一次执行转换
5. 先产出一份 API Automation Assets 和一份 UI Automation Assets 示例
6. 最后整理 blocked / assumptions / open questions

这样可以最快形成可提交、可协作、可继续扩展的 member3 交付物。
