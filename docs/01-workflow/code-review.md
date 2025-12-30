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

---

## 6. Reviewer Assignment (审核人指派)

GitHub 提供了多种方式来实现从“手动指定”到“全自动分配”的流程：

### 6.1 Manual Assignment (手动指定)
最直接的方式，在 PR 创建页面或侧边栏侧边栏（Sidebar）：
* **Reviewers**: 搜索并指定个人（如 `@username`）或团队（如 `@org/team`）。
* **Assignees**: 通常指 PR 的负责人（通常是你自己），负责推进 PR 流程。

### 6.2 CODEOWNERS (推荐)
在仓库根目录或 `.github/` 目录下创建 `CODEOWNERS` 文件。一旦 PR 修改了特定文件，GitHub 会**自动**邀请对应的负责人作为 Reviewer。

```text
# 所有 .js 文件归前端组管
*.js    @my-org/frontend-team

# /docs/ 目录下的变更找文档团队
/docs/  @my-org/docs-team
```

### 6.3 Team Code Review Assignment (团队内轮岗)
如果指定了一个大型团队，可以开启“自动分配”策略，避免干扰所有人：
* **Round robin (轮询)**: 按顺序轮流分配。
* **Load balance (负载均衡)**: 优先分配给最近 Review 任务较少的人。
* **设置路径**: Team Settings -> Code review assignment。

### 6.4 Branch Protection Rules (强制审核)
结合“分支保护规则”强制执行审核流程：
1.  开启 **Require a pull request before merging**。
2.  勾选 **Require review from Code Owners**。

这样可以确保关键代码（如 `src/`）必须经过特定人员（Code Owners）的 Approve 才能合并。