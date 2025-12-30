# Git Workflow & Branching Model
### Git 工作流与分支管理规范

> **Objective**: 统一代码管理策略，确保代码库的整洁性 (Clean History) 和发布的稳定性 (Stability)。
> **Scope**: 适用于 Frontend (`revieu-web`), Backend (`revieu-backend`), Infrastructure (`revieu-infra`) 所有仓库。

---

## 1. Branching Model (分支模型)

RevieU 采用 **Simplified Gitflow** 模型。我们维护两条长期存在的受保护分支 (Protected Branches)：

| Branch Name | Env | Permission | Description |
| :--- | :--- | :--- | :--- |
| **`main`** | **Production** | Admin Only | **生产环境主分支**。任何时刻该分支的代码都必须是稳定的、可随时发布的。仅接受来自 `dev` 的合并或 `hotfix`。 |
| **`dev`** | **Staging/Dev** | Maintainer | **开发主干分支**。所有新功能 (`feat`) 和非紧急修复 (`fix`) 的汇聚点。CI/CD 会自动将其部署到测试环境。 |

除了上述主分支外，开发过程中会临时存在以下分支：

| Branch Prefix | Source | Target | Description |
| :--- | :--- | :--- | :--- |
| **`feat/...`** | `dev` | `dev` | **功能开发分支**。日常开发使用。 |
| **`fix/...`** | `dev` | `dev` | **Bug 修复分支**。修复测试环境发现的问题。 |
| **`hotfix/...`** | `main` | `main` & `dev` | **紧急修复分支**。仅用于修复生产环境 (Production) 的严重故障。 |
| **`refactor/...`**| `dev` | `dev` | **重构分支**。不改变功能前提下的代码优化。 |

---

## 2. Naming Convention (命名规范)

为了与 GitHub Projects/Issues 自动关联，分支命名必须遵循以下格式：

**Format**: `type/issue-id-short-description`

* **`type`**: 对应分支类型 (`feat`, `fix`, `hotfix`, `docs`).
* **`issue-id`**: GitHub Issue ID (例如 `102`).
* **`short-description`**: 简短描述，使用 kebab-case (短横线连接).

**Examples (示例):**
* ✅ `feat/42-user-login-page` (开发 ID 为 42 的登录页需求)
* ✅ `fix/108-api-timeout` (修复 ID 为 108 的接口超时 Bug)
* ✅ `hotfix/prod-db-connection` (紧急修复生产库连接，可能没有 Issue ID)
* ❌ `zhangsan/login` (禁止使用个人名字)
* ❌ `update-readme` (缺少类型和 Issue ID)

---

## 3. Commit Message Standards (提交信息规范)

RevieU 强制遵循 **[Conventional Commits](https://www.conventionalcommits.org/)** 规范。这有助于自动生成 Changelog 并保持历史清晰。

**Format**: `type(scope): subject`

### 3.1 Type (类型)
* **`feat`**: 新增功能 (A new feature)
* **`fix`**: 修复 Bug (A bug fix)
* **`docs`**: 仅修改文档 (Documentation only changes)
* **`style`**: 格式修改，不影响逻辑 (White-space, formatting, missing semi-colons, etc)
* **`refactor`**: 代码重构 (A code change that neither fixes a bug nor adds a feature)
* **`perf`**: 性能优化 (A code change that improves performance)
* **`test`**: 增加或修改测试 (Adding missing tests or correcting existing tests)
* **`chore`**: 构建过程或辅助工具变动 (Changes to the build process or auxiliary tools)

### 3.2 Scope (范围 - 可选)
指明影响的模块。
* Frontend: `auth`, `ui`, `router`
* Backend: `user-svc`, `db`, `gateway`

### 3.3 Subject (主题)
简短描述改动，**使用英文**，动词开头，不超过 50 个字符。

**Examples (示例):**
* ✅ `feat(auth): add google oauth login support`
* ✅ `fix(user-svc): handle null pointer exception in profile update`
* ✅ `docs(readme): update architecture diagram`
* ❌ `feat: 增加登录` (避免中文)
* ❌ `fix bug` (描述太模糊)

---

## 4. Workflow Lifecycle (开发全流程)

### Step 1: Start a Task (Issue First!)
1.  **Issue First**: 所有开发工作必须先有 Issue。如果没有，请自己在 GitHub 上开一个 Issue。
2.  **Assign**: 将该 Issue 指派给自己，确保团队其他成员知道谁在负责这项任务。
3.  **In Progress**: 在 GitHub Projects 上领取任务，将状态拖至 **In Progress**。
4.  **Local Branch**: 在本地拉取最新代码并创建分支（分支名务必带上 Issue ID）：
    ```bash
    git checkout dev
    git pull origin dev
    git checkout -b feat/101-add-avatar-upload
    ```

### Step 2: Development & Commit
1.  编写代码。
2.  提交代码 (原子化提交，不要把无关的改动混在一起)：
    ```bash
    git add .
    git commit -m "feat(ui): implement avatar upload component"
    ```

### Step 3: Push & Pull Request
1.  推送到远端：
    ```bash
    git push origin feat/101-add-avatar-upload
    ```
2.  在 GitHub 仓库页面创建 **Pull Request (PR)**：
    * **Base**: `dev`
    * **Compare**: `feat/101-add-avatar-upload`
    * **Title**: `feat: Implement avatar upload (#101)` (关联 Issue)
3.  **Assign**: 指派给自己。
4.  **Reviewers**: 根据 `README.md` 中的 Team 说明，邀请对应的组 (如 `@RevieU-Corp/Frontend-Devs`) 进行 Review。

### Step 4: Code Review & Merge
1.  Reviewer 提出修改意见 (Request Changes) 或 批准 (Approve)。

### Step 5: Clean Up
PR 合并后，GitHub 会自动关闭 Issue（如果关联了）。请删除远程和本地的功能分支：
```bash
git branch -d feat/101-add-avatar-upload
```

---

## 5. Hotfix Process (紧急修复流程)

当生产环境 (`main`) 出现严重 Bug 时：

1. 从 `main` 分支切出 `hotfix` 分支：
```bash
git checkout main
git checkout -b hotfix/payment-error
```


2. 修复 Bug 并提交。
3. 开启 PR 合并回 `main`。
4. **关键步骤**: 修复同时必须同步回 `dev`，防止下次发布时 Bug 复活。
* *操作*: 将代码 Cherry-pick 到 `dev` 或另外提一个 PR 合并到 `dev`。

---

## 6. Troubleshooting (紧急救援指南)

**场景**: 你不小心在受保护的 `main` 或 `dev` 分支上直接修改了代码（甚至已经 commit 了），但无法推送到远端，因为分支保护规则会拦截。

**请根据你当前的状态选择救援方案：**

### Case A: 代码还没提交 (Uncommitted Changes)
*状态：你修改了文件，可能执行了 `git add`，但还没执行 `git commit`。*

1.  **暂存修改** (把代码“打包”藏起来，让工作区回到干净状态)：
    ```bash
    git stash
    ```
2.  **切换/新建分支** (切到正确的分支)：
    ```bash
    git checkout dev
    git checkout -b feat/your-feature-name
    ```
3.  **释放修改** (把刚才打包的代码应用到新分支上)：
    ```bash
    git stash pop
    ```
4.  现在的状态就是安全的了，你可以继续开发并提交。

### Case B: 代码已经提交到了本地 (Committed but not Pushed)
*状态：你已经执行了 `git commit`，但还在本地 `main` 分支上。*

1.  **带走提交，新建分支** (创建一个包含你刚才那些 Commit 的新分支)：
    ```bash
    git checkout -b feat/your-feature-name
    ```
2.  **恢复主干** (切回主干，把它重置回远端的状态)：
    ```bash
    git checkout main
    git reset --hard origin/main
    ```
    *(注意：`--hard` 会丢弃 main 上所有未提交的改动，请确保你的代码已经在第 1 步的新分支里了)*

---

## 7. Git Configuration

为了统一换行符和用户信息，请确保正确配置：

```bash
# 避免不同系统换行符冲突
git config --global core.autocrlf input  # macOS/Linux
git config --global core.autocrlf true   # Windows

# 检查你的邮箱是否与 GitHub 账号一致
git config --global user.name "Your Name"
git config --global user.email "your.email@revieu.com"
```

---

## 8. Collaborative Principles (协作准则与风险管理)

随着团队规模扩大，为了减少冲突和重复劳动，必须遵循以下协作原则：

### 8.1 Atomic Updates (原子化更新)
*   **原则**: 每次 Submission 或 Commit 必须是最小原子单位。
*   **文件移动 vs 代码修改**: 
    *   如果你需要移动文件路径 (Refactor Location)，请**仅执行移动操作**并立即 Push。不要在同一个 Commit 中修改该文件的代码。
    *   如果你需要修改代码，请在移动操作完成后，再另起 Commit 进行修改。
*   **禁止混合**: 禁止在一次 PR 中同时包含大规模的文件结构调整和复杂的逻辑修改。

### 8.2 Conflict Management (冲突处理)
*   **谁产生，谁解决**: 发生冲突时，**最后一位提交代码的人**（即触发了冲突的那位）负责处理冲突并确保合并后的稳定性。
*   **及时 Rebase**: 开发过程中应频繁 `git pull origin dev` 并 Rebase 到自己的分支上，尽早发现潜在冲突。

### 8.3 Major Refactoring (大规模破坏性变更通知)
*   进行大规模 Refactor、重构核心组件或破坏性更新前，**必须**在 Discord 或 微信群里提前同步。
*   严禁在未告知团队的情况下对公共模块执行大规模结构调整，这会导致其他正在并行开发的同事面临严重的合并地狱。

### 8.4 Task Visibility (任务透明度)
*   **Issue 是协作的起点**: 严禁在没有 Issue/Task 的情况下直接提交代码。创作 Issue 后必须指派（Assign）负责人。
*   **AI 协作提示**: 提交任务描述给 AI 时，请确保它知道你的 Issue ID，以便生成规范的分支名和 Commit Message。


