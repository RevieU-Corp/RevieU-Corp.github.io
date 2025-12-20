# Repository Integrations & Webhooks
### 仓库集成与钩子配置

> **Scope**: 本文档记录了 RevieU 项目所有外部系统的集成点 (Integrations) 和自动化触发器 (Webhooks)。
> **Security Note**: 文档中仅记录集成逻辑。**严禁**明文记录任何 API Key、Webhook Secret 或 Token。请前往 CI/CD 环境变量或 K8s Secrets 查看具体凭证。

---

## 1. Integration Topology (集成拓扑)

我们的代码仓库 (`revieu-backend` / `revieu-web`) 不仅仅是代码存储地，它通过 Webhooks 与整个研发工具链相连。

* **CI System**: GitHub Actions (自动构建与测试)
* **Code Quality**: SonarQube / CodeClimate (代码质量扫描)
* **Notifications**: Feishu (飞书) / Slack (研发群通知)
* **Monitoring**: Sentry (错误追踪)

---

## 2. Active Webhooks (活跃钩子配置)

### 2.1 CI/CD Pipeline Triggers
* **Provider**: GitHub Actions (Native Integration)
* **Trigger Events**:
    * `push` (branches: `main`, `develop`) -> 触发构建与部署。
    * `pull_request` (opened, synchronize) -> 触发 Lint 和 Unit Test。
* **Configuration**: 详见各仓库 `.github/workflows/*.yml` 文件。

### 2.2 ChatOps Notifications (飞书/Slack)
* **Purpose**: 将关键研发事件同步到 "RevieU 研发大群"。
* **Provider**: Feishu Webhook Bot
* **Trigger Events**:
    * `pull_request` (merged): 通知大家有新功能合入。
    * `release` (published): 通知生产环境发布成功。
    * **CI Failure**: 构建失败时，@对应提交人。
* **Setup**:
    * Webhook URL 存储在 GitHub Repository Secrets 中 (`FEISHU_WEBHOOK_URL`)。
    * 由 GitHub Actions 的 `send-notification` 步骤调用。

### 2.3 Code Quality Analysis
* **Provider**: SonarQube (或类似工具)
* **Mechanism**:
    * 并非通过直接的 Webhook 触发。
    * 而是集成在 CI Pipeline 中，作为 `Scan` 步骤运行。
    * 扫描结果会回写到 GitHub PR 的 `Checks` 页面。

---

## 3. External Service Configurations (外部服务依赖)

除 Webhooks 外，应用运行还需要连接以下第三方服务：

| Service | Purpose | Integration Method | Configuration Key (Env) |
| :--- | :--- | :--- | :--- |
| **Sentry** | Error Tracking | SDK Integration | `SENTRY_DSN` |
| **Google OAuth** | Social Login | OAuth 2.0 Protocol | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` |
| **AWS S3 / OSS** | File Storage | SDK / API | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` |

> **注意**: 如果在 Sentry 中看不到报错，或者 Google 登录失败，请首先检查上述环境变量是否在 K8s ConfigMap/Secret 中正确配置。

---

## 4. Troubleshooting Webhooks (故障排查)

当出现“代码推上去了，但没反应”的情况时，请按以下步骤自查：

### Step 1: Check GitHub Actions Tab
* 进入 GitHub 仓库页面 -> 点击 **Actions**。
* 查看是否有正在运行或失败的 Workflow。
* 如果列表是空的，说明 Webhook 根本没触发，或者 YAML 语法有严重错误导致无法解析。

### Step 2: Verify Webhook Deliveries (Admin Only)
* 进入仓库 **Settings** -> **Webhooks**。
* 点击对应的 Webhook URL (例如飞书机器人的地址)。
* 切换到 **Recent Deliveries** 标签页。
* **Status Icons**:
    * ✅ **Green (200 OK)**: 发送成功。如果是机器人没弹消息，可能是消息格式 (JSON) 写错了。
    * ❌ **Red (4xx/5xx)**: 发送失败。
        * `401/403`: Secret/Token 可能过期或被重置了。
        * `502/504`: 目标服务挂了。

### Step 3: Local Simulation (本地模拟)
对于复杂的 Webhook 逻辑（比如自建的自动化脚本），可以使用工具在本地回放 Payload：
* 复制 Recent Deliveries 中的 `Payload` JSON。
* 使用 Postman 发送到本地测试服务，Debug 逻辑。

---

## 5. Maintenance (维护指南)

* **Secret Rotation**: 如果怀疑 Webhook Secret 泄露，请立即在目标平台（如飞书后台）重新生成，并同步更新 GitHub Secrets。
* **Pruning**: 定期清理不再使用的 Webhooks（比如之前测试用的钩子），防止向未知的 URL 发送敏感数据。