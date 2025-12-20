# Definition of Done (DoD)
### 完成的标准定义

> **Philosophy**: "Done" means **Deployable**.
> “做完”不仅仅意味着代码写完了，而是意味着该功能已经准备好随时发布到生产环境，且没有任何已知缺陷。

---

## 1. General Criteria (通用标准)
*所有提交（Feature, Bugfix, Refactor）都必须满足的基础底线。*

- [ ] **CI Passed**: 所有自动化流水线（Lint, Unit Test, Build）均为绿色通过状态。
- [ ] **Code Review**: 获得至少 1 位 Reviewer 的 Approval，并解决了所有 `[Blocking]` 级别的评论。
- [ ] **Clean History**: Git 分支没有乱七八糟的 Merge Commit，且 Commit Message 遵循规范。
- [ ] **No Dead Code**: 删除了所有调试用的 `console.log`, `print()`, 注释掉的废弃代码。
- [ ] **Configuration**: 如果引入了新的环境变量（如 `DB_HOST`），已在 `revieu-infra` 仓库或 `.env.example` 中同步更新。

---

## 2. Specific Criteria (特定场景标准)

### 🎨 For Frontend (RevieU Web)
- [ ] **UI/UX Match**: 页面还原度符合设计稿（Figma）要求（间距、字体、颜色）。
- [ ] **Responsive**: 在移动端（Mobile）和桌面端（Desktop）显示正常，无布局错乱。
- [ ] **Loading State**: 所有异步请求都有 Loading 骨架屏或转圈动画，禁止出现“点击无反应”。
- [ ] **Error Handling**: 接口报错（404/500）时，有友好的 Toast 提示或 Error Boundary 兜底页面。
- [ ] **Cross-Browser**: 在主流浏览器 (Chrome, Safari, Firefox) 下验证通过。

### ⚙️ For Backend (Microservices)
- [ ] **API Tests**: 新增接口包含对应的单元测试 (Unit Test) 或集成测试 (Integration Test)。
- [ ] **Documentation**: Swagger/OpenAPI 文档已自动生成或手动更新，且在 Apifox 中可调通。
- [ ] **Error Codes**: 抛出的异常使用了标准的错误码 (Error Code)，而不是仅仅返回 HTTP 500。
- [ ] **Performance**: 关键路径不存在 N+1 查询问题，且大列表接口支持分页 (Pagination)。
- [ ] **Logging**: 关键业务流程有 Info 级别的日志，且包含 `trace_id`。

### 🐛 For Bug Fixes
- [ ] **Reproduction**: 包含了一个能复现该 Bug 的测试用例（防止回归）。
- [ ] **Root Cause**: 在 PR 描述中简要说明了 Bug 产生的原因。
- [ ] **Verification**: 验证了修复不仅解决了当前 Bug，且没有破坏现有功能。

---

## 3. Reviewer Checklist (验收者自查)
*Reviewer 在点击 Approve 之前，请确认：*

> "Would I be comfortable if this code is deployed to Production right now?"
> 如果这段代码现在就上线，我会感到安心吗？

- 如果代码很复杂，我是否看得懂？（看不懂通常意味着代码写得烂，而不是你水平低）。
- 是否有显而易见的安全性问题（SQL 注入、XSS、权限绕过）？
- 这个改动是否需要更新 Wiki 或 Handbook？

---

## 4. Definition of Ready (DoR) - 什么时候可以开始做？
*DoD 是定义什么时候结束，DoR 是定义什么时候开始。防止开发人员接手“一句话需求”。*

在将 Issue 拖入 **In Progress** 之前，请确保：
1.  **Clear Goal**: 清楚要做什么，解决了谁的问题。
2.  **Resources**: UI 设计稿已定稿，后端接口定义已提供（如果是前端任务）。
3.  **Acceptance Criteria (AC)**: 验收标准已明确列出（例如：“点击按钮后，3秒内跳转到首页”）。

---