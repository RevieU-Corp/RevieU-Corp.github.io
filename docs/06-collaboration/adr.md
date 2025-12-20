# Architecture Decision Records (ADR)
### 架构决策记录指南

> **Definition**: ADR 是一种轻量级的文档，用于捕捉重要的架构决策及其背景 (Context) 和后果 (Consequences)。
> **Goal**: 避免“重复讨论已决定的事”，并为后来者解释“为什么当时我们这么做”。

---

## 1. When to write an ADR? (何时写?)

并不是所有的决定都要写 ADR。只有那些具有**重大影响 (Significant Impact)** 且 **难以逆转 (Hard to Reverse)** 的决策才需要记录。

### ✅ 需要写 ADR 的场景
* **技术选型**: “我们决定后端微服务主要使用 Go，辅助服务使用 Python。”
* **引入中间件**: “引入 Kafka 作为消息队列。”
* **核心模式**: “决定采用 JWT + 网关的鉴权模式，而不是 Session。”
* **数据库变更**: “从 MySQL 迁移到 PostgreSQL。”
* **规范变更**: “决定强制所有 API 使用 `snake_case` 返回字段。”

### ❌ 不需要写 ADR 的场景
* **日常开发**: “把这个函数重构了一下。”
* **小依赖**: “引入了一个处理时间的工具库 (`dayjs`)。”
* **Bug 修复**: “修复了登录接口的空指针异常。”

---

## 2. ADR Lifecycle (生命周期)

ADR 不是写完就死的，它有状态流转：



1.  **Proposed (提议中)**:
    * 你有一个想法（例如：引入 Redis 缓存）。
    * 创建一个 PR，包含 Markdown 格式的 ADR 草稿。
    * Assign 给 `@RevieU-Corp/Arch-Team` 进行评审。
2.  **Accepted (已采纳)**:
    * 团队达成共识，PR 合并。
    * 该决策正式生效，后续开发必须遵循。
3.  **Rejected (已拒绝)**:
    * 经过讨论，团队认为方案不可行或成本太高。
    * **保留文档**，标记为 Rejected。
    * *价值*: 防止未来有人提出同样的馊主意，我们可以直接把文档甩给他：“看，两年前我们讨论过，结论是不行。”
4.  **Deprecated (已废弃)**:
    * 随着时间推移（如 2 年后），该决策不再适用。
    * 新建一个 ADR (例如 ADR-050) 来取代旧的 ADR (例如 ADR-005)。

---

## 3. Storage & Naming (存储与命名)

所有的 ADR 统一存放在 `revieu-handbook/docs/adrs/` 目录下。

**命名格式**: `NNN-short-title.md` (三位数字编号)

* `001-record-architecture-decisions.md` (这是第一个 ADR，决定我们要开始写 ADR 了)
* `002-use-go-for-microservices.md`
* `003-adopt-apisix-gateway.md`

---

## 4. ADR Template (标准模板)

请复制以下模板用于创建新的 ADR。我们采用业界标准的 **Michael Nygard 格式**。

```markdown
# ADR-000: [Title of the Decision]

* **Status**: Proposed / Accepted / Rejected / Deprecated
* **Date**: YYYY-MM-DD
* **Author**: @YourGithubHandle
* **Deciders**: @Arch-Team, @CTO

## Context (背景)
[描述我们要解决的问题是什么？当前的痛点是什么？]
例如：目前的单体应用部署太慢，每次修改一个小功能都要重新编译整个后端，导致发布效率极低。

## Decision (决策)
[我们决定怎么做？]
例如：我们将拆分后端为微服务架构。核心计算服务使用 Go 语言，AI 推荐服务使用 Python。服务间通信使用 gRPC。

## Consequences (后果)
[这个决定带来了什么好处？又引入了什么坏处/成本？这里必须诚实。]

### ✅ Positive (收益)
* 可以独立部署，互不影响。
* Go 语言并发性能更好，节省服务器成本。
* Python 团队可以专注于算法，不需要管 Web 逻辑。

### ❌ Negative (成本/风险)
* 运维复杂度大幅上升，需要维护 K8s。
* 分布式事务处理变得困难（不再有 ACID）。
* 团队需要学习 gRPC 协议。

## References (参考)
* [Link to POC code]
* [Link to technical blog]