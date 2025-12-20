# Code Review Guide
### 代码评审指南

> **Philosophy**: Code Review (CR) 是我们保证工程质量的最后一道防线，也是团队成员互相学习（Knowledge Sharing）的最佳时机。
> **Golden Rule**: 评论代码，而不是评论人。保持礼貌与建设性。

---

## 1. Author's Responsibility (提交者职责)

在将 Pull Request (PR) 指派给他人之前，请务必完成以下 **Self-Check (自查)**：

### ✅ Pre-submission Checklist
1.  **Self Review**: 自己先在 GitHub Files Changed 页面把代码从头到尾看一遍。
    * *问自己：如果不看代码逻辑，光看变量名，能猜出这段代码在干嘛吗？*
2.  **Lint & Test**: 确保本地 `ESLint` / `Go Vet` 无报错，且相关 Unit Tests 全部通过。
3.  **Clean History**: 是否有多余的 `console.log`、调试用的注释代码？Commit Message 是否清晰？
4.  **Context**: PR 描述（Description）是否贴上了 GitHub Project/Issue 链接？是否附带了 UI 截图（前端）或 API 响应示例（后端）

---

## 2. Reviewer's Guide (审核者指南)

当你被 Assign 了一个 PR，请遵循以下原则：

### ⏳ Response Time (响应时间)
* **24小时原则**: 请在被指派后的 1 个工作日内给出反馈。
* 如果 PR 太大（超过 500 行）没法立刻看，请评论告知：“我收到了，预计明天下午看。”

### 🔍 What to Look For (看什么)

不要把时间浪费在机器能做的事情上（格式、缩进、分号），让 CI 去跑 Lint。**Reviewer 应该关注机器看不出来的逻辑问题：**

#### 🟢 General (通用标准)
* **Design**: 代码结构是否合理？是否过度设计 (Over-engineering)？遵循 DRY (Don't Repeat Yourself) 吗？
* **Readability**: 变量和函数命名是否语义化？魔法数字 (Magic Numbers) 是否提取成了常量？
* **Test Coverage**: 新增的业务逻辑有测试覆盖吗？测试用例是否覆盖了边界条件 (Edge Cases)？
* **Security**: 是否存在 SQL 注入风险？是否在 Log 中打印了敏感数据 (Password/Token)？
---

## 3. Review Etiquette (沟通礼仪)

为了减少摩擦，建议使用以下标签明确你的评论意图：

* **[Blocking]** (必须修): 代码有 Bug、安全漏洞或严重设计缺陷。如果不改，我不会 Approve。
    * *例: "[Blocking] 这里在循环里查询数据库，数据量大时会拖垮 DB，请改成 `In` 查询。"*
* **[Nitpick]** (吹毛求疵/建议): 即使不改也可以合并，但我建议你优化。
    * *例: "[Nitpick] 这个变量名如果叫 `isUserLoggedIn` 会不会更清楚一点？"*
* **[Question]** (提问): 我没看懂，请解释一下（不代表你写错了）。
    * *例: "[Question] 为什么要在这里强制类型转换？是为了兼容旧数据吗？"*

---

## 4. Approval Process (批准流程)

1.  **Request Changes**: 只要有 1 个 `[Blocking]` 问题，就必须选择 "Request Changes"。
2.  **Comment**: 只有 `[Nitpick]` 或 `[Question]`，可以选择 "Comment"。
3.  **Approve**: 问题都修复了，或者认为代码已经 Ready to Merge。

> **Note**: 对于涉及核心架构变更的 PR，必须至少获得一名 `@RevieU-Corp/Arch-Team` 成员的 Approval。

---

## 5. Handling Disagreements (处理分歧)

如果 Author 和 Reviewer 僵持不下：

1.  不要在 GitHub 评论区进行长篇大论的辩论（超过 3 个回合）。
2.  直接线下沟通。
3.  如果无法达成一致，请 Tech Lead 或架构师进行仲裁。