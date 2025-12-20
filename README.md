# RevieU Engineering Handbook
### RevieU 研发手册

> **Single Source of Truth** for RevieU Engineering Team.
> RevieU 研发团队的唯一真理来源。

![Status](https://img.shields.io/badge/Status-Under_Construction-yellow) ![Maintainer](https://img.shields.io/badge/Maintainer-@Arch--Team-blue) ![Access](https://img.shields.io/badge/Access-Internal_Only-red) ![Version](https://img.shields.io/badge/Version-EA-lightgrey) 

---

## 📖 About (简介)

欢迎来到 **RevieU**。本手册旨在定义团队的工程标准、协作流程与技术架构。无论你是前端、后端还是运维工程师，都必须遵循本文档中定义的规范，以确保项目的高内聚、低耦合与可维护性。

**Our Philosophy (核心理念):**
1.  **Code as Documentation**: 代码即文档，保持清晰优于炫技。
2.  **Consistency**: 统一的“平庸”代码 > 混乱的“天才”代码。
3.  **Automation**: 能自动化的流程（Lint, Test, Deploy）绝不依赖人工。
4.  **Docs as Code**: 文档像代码一样需要 Review、需要版本控制。

---

## 🏗️ Repository Matrix (项目仓库矩阵)

RevieU 采用多仓库协作模式，请根据职能 Clone 相应的仓库：

| Repository Name | Scope | Tech Stack | Description |
| :--- | :--- | :--- | :--- |
| **[`revieu-handbook`](https://github.com/RevieU-Corp/revieu-handbook)** | **Docs** | Markdown | 本仓库。包含开发规范、架构文档、Onboarding 指南。 |
| **[`revieu-web`](https://github.com/RevieU-Corp/revieu-web)** | **Frontend** | React, TypeScript, Tailwind | 前端单页应用 (SPA) 代码。 |
| **[`revieu-backend`](https://github.com/RevieU-Corp/revieu-backend)** | **Backend** | Go, Python, Java | 微服务后端代码 (Monorepo)，包含所有 Service。 |

---

## 👥 Team Structure & Contact (团队与联系人)

为了提高沟通效率，我们在 GitHub Organization 中预设了以下 User Groups。在 Issue 或 PR 中请直接使用 Group Alias 进行提问或指派，避免点对点私聊。

| Team Alias (团队代号) | Scope (负责范围) | Access Level (权限) | When to Mention (何时艾特) |
| :--- | :--- | :--- | :--- |
| **`@RevieU-Corp/Arch-Team`** | **Architecture & Docs** | All Repos (Admin) | 架构设计评审、文档纠错、重大技术决策、CI/CD 故障。 |
| **`@RevieU-Corp/Frontend-Devs`** | **Web / UI** | `revieu-web` (Write) | 前端组件复用询问、UI 还原度验收、BFF 层接口联调。 |
| **`@RevieU-Corp/Backend-Devs`** | **Microservices** | `revieu-backend` (Write) | 接口报错 (500/502)、数据库字段新增、API 契约变更。 |

> **Note**: 请根据你的职能加入对应的 Team。如果你无法 Push 代码，请检查你是否在正确的 Team 中。

---

## 📚 Documentation Index (文档索引)

### 01. Workflow & Collaboration (协作流程)
*定义我们如何协同工作，如何管理代码生命周期。*
* [**Git Workflow**](./docs/01-workflow/git-flow.md): 分支策略 (Branching Model)、Commit Message 规范。
* [**Code Review Guide**](./docs/01-workflow/code-review.md): 提交 PR 前的自查清单 (Checklist)。
* [**Definition of Done (DoD)**](./docs/01-workflow/dod.md): 任务完结的标准定义。

### 02. Coding Standards (开发规约)
*各技术栈的硬性代码规范。*
* [**Global API Standards**](./docs/02-standards/api-specs.md): **(核心)** 统一响应体结构 (Response Envelope)、错误码字典、RESTful 命名设计。
* [**Frontend Guidelines**](./docs/02-standards/frontend-react.md): React 组件目录、Hooks 使用规范、CSS 命名。
* [**Backend Guidelines (Polyglot)**](./docs/02-standards/backend-polyglot.md): Go/Python/Node 多语言共存规范、日志格式 (Log Format)、错误处理。

### 03. Architecture & Design (架构设计)
*系统的宏观设计与决策记录。*
* [**System Architecture**](./docs/03-architecture/system-overview.md): 系统拓扑图、微服务拆分逻辑。
* [**Database Schema**](./docs/03-architecture/database-design.md): 数据库设计原则、Migration 流程。
* [**Authentication Flow**](./docs/03-architecture/auth-flow.md): JWT 鉴权流程、网关 (Gateway) 转发逻辑。

### 04. DevOps & Deployment (运维与部署)
*如何构建、部署与监控。*
* [**Environment Setup**](./docs/04-devops/local-setup.md): 本地开发环境搭建 (Docker Compose)。
* [**CI/CD Pipeline**](./docs/04-devops/cicd-pipeline.md): 流水线配置说明。
* [**Integrations & Webhooks**](./docs/04-devops/integrations.md): 消息通知、Webhook 配置说明。

### 05. Security & Compliance (安全与合规)
*定义系统的安全防线与数据合规要求。*
* [**Secret Management**](./docs/05-security/secrets.md): 密钥管理、环境变量配置。
* [**Data Privacy**](./docs/05-security/privacy.md): 个人隐私数据 (PII) 处理、日志脱敏。
* [**Security Scanning**](./docs/05-security/scanning.md): 依赖扫描与静态分析。

### 06. Project & Collaboration (项目协作)
*团队日常运作与决策记录。*
* [**Release Management**](./docs/06-collaboration/release.md): 版本命名习惯与发版流程。
* [**Architecture Decisions**](./docs/06-collaboration/adr.md): 重大决策记录 (ADR) 提交指南。

---

## 🚀 Onboarding Checklist (必读)

请按顺序完成以下步骤：

1.  [ ] 阅读本页面的 **About** 与 **Philosophy**。
2.  [ ] 配置本地开发环境，详见 [**Environment Setup**](./docs/04-devops/local-setup.md)。
3.  [ ] 阅读你所在技术栈的 **Coding Standards**。
4.  [ ] 从 **Project** 领取你的第一个 `Good First Issue`。
5.  [ ] 提交代码前，确保阅读了 [**Git Workflow**](./docs/01-workflow/git-flow.md)。

---

## 🤝 Contribution (如何贡献文档)

本文档是活的 (Living Document)。如果你发现流程有误或规范过时，请遵循以下步骤进行更新：

1.  **Create Branch**: 在本地拉取最新代码，并创建一个新的文档分支。
    * 修改子目录请使用 `docs/01-workflow/short-description` (例如 `docs/01-workflow/update-git-flow`)
    * 修改 README 请使用 `docs/short-description` (例如 `docs/update-git-flow`)
2.  **Edit**: 修改 Markdown 文件，确保格式整洁。
3.  **Push & PR**: 将分支推送到本仓库 (Origin)，发起 Pull Request 并 Assign 给 `@RevieU-Corp/Arch-Team`。
4.  **Discussion**: 对于核心规范（如 API 结构、分支模型）的变更，请务必先在 PR 中讨论，**禁止**在未达成共识前合并。

---

*© 2025 RevieU Team. Internal Use Only.*