# Release Management & Versioning
### 版本管理与发版流程

> **Objective**: 确保每一次发布都是**可追踪 (Traceable)**、**可回滚 (Revertible)** 且**版本清晰 (Semantic)** 的。
> **Standard**: 我们严格遵循 [Semantic Versioning 2.0.0](https://semver.org/)。

---

## 1. Versioning Strategy (版本命名策略)

### 1.1 Semantic Versioning (语义化版本)
格式：**`MAJOR.MINOR.PATCH`** (主版本号.次版本号.修订号)

| Segment | Meaning | When to increment? | Example |
| :--- | :--- | :--- | :--- |
| **MAJOR** | **破坏性变更** | API 不兼容、重构核心架构。 | `1.0.0` -> `2.0.0` |
| **MINOR** | **新功能** | 增加向下兼容的新特性 (Feature)。 | `1.1.0` -> `1.2.0` |
| **PATCH** | **修复 Bug** | 向下兼容的问题修正 (Hotfix)。 | `1.2.0` -> `1.2.1` |



### 1.2 Docker Image Tagging (镜像标签)
在 Kubernetes 生产环境中，**严禁使用 `latest` 标签**。必须使用明确的版本号或 Commit SHA。

* **Production**: `revieu-backend:v1.2.1`
* **Staging**: `revieu-backend:v1.3.0-beta.1`
* **Dev**: `revieu-backend:sha-a1b2c3d` (使用 Git Commit Hash 前7位)

---