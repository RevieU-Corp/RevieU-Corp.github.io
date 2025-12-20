# Polyglot Backend Guidelines
### 多语言后端通用规范

> **Objective**: 在微服务架构中，允许“为特定场景选择最合适的语言”，但严禁“各自为政”。
> **Core Philosophy**: **Internal Diversity, External Consistency.** (内部实现多样化，外部行为标准化)。

---

## 1. The "Black Box" Contract (黑盒契约)

无论服务是用 Go, Python, Node.js 还是 Java 编写，它在基础设施（Kubernetes/Docker）眼中必须表现一致。所有服务必须遵守以下**硬性指标**：

### 1.1 Configuration (配置)
* **12-Factor App**: 严禁将配置（数据库密码、API Key、端口号）硬编码在代码中。
* **Environment Variables**: 所有配置必须通过**环境变量 (Environment Variables)** 注入。
* **Validation**: 服务启动时必须检查关键配置是否存在。如果缺少 `DB_HOST` 等必要配置，服务必须**立即崩溃 (Fail Fast)**，而不是带病运行。

### 1.2 Network & Ports (网络端口)
* **Service Port**: 容器内统一监听 **`8080`** 端口（除非特殊说明）。
* **Metric Port** (可选): 如果有 Prometheus 监控指标，暴露在 `9090`。

### 1.3 Health Checks (健康检查)
所有服务必须显式暴露以下两个接口，供 K8s 探针调用：
* `GET /health/liveness`: 返回 200 OK。表示进程还在运行。
* `GET /health/readiness`: 返回 200 OK。表示数据库/缓存已连接，可以开始接客了。

---

## 2. Observability Standards (可观测性标准)

这是多语言架构中最容易失控的部分。为了保证日志系统 (ELK) 和链路追踪 (Jaeger/Tempo) 能正常工作，所有语言必须吐出**格式完全一致**的数据。

### 2.1 Structured Logging (结构化日志)
严禁使用 `print` 或 `fmt.Println` 输出纯文本日志。**必须输出单行 JSON 格式**。

**❌ Bad (Text):**
`[INFO] 2025-12-20 User 123 logged in.`

**✅ Good (JSON):**
```json
{
  "level": "info",
  "ts": "2025-12-20T10:00:00.000Z",
  "service": "user-service",
  "trace_id": "a1b2c3d4e5",
  "msg": "User logged in",
  "user_id": 123,
  "ip": "192.168.1.1"
}

```

### 2.2 Distributed Tracing (链路追踪)

* **Trace Context**: 所有服务收到请求时，必须尝试从 HTTP Header 解析 `X-Request-ID` (或 `traceparent`)。
* **Propagation**: 在调用下游服务（DB, Redis, 或其他 HTTP 服务）时，必须将该 ID **透传**下去。
* **Generation**: 如果 Header 里没有 ID（链路的起点），则必须生成一个新的 UUID。

---

## 3. Layered Architecture (分层架构模式)

虽然语言语法不同，但代码的**逻辑分层**必须保持一致。这降低了跨语言阅读代码的认知负担。

任何后端服务都应该包含（但不限于）以下三层：

| Layer Name | Alias | Responsibility |
| --- | --- | --- |
| **Interface Layer** | `Handler` / `Controller` / `Router` | 解析 HTTP 请求，校验参数，调用 Service。**不包含业务逻辑。** |
| **Business Layer** | `Service` / `Logic` / `Usecase` | 核心业务逻辑，事务控制，编排多个 Repository。 |
| **Data Layer** | `Repository` / `DAO` / `Store` | 直接与数据库/缓存交互。**只有这一层能写 SQL。** |

**禁止越级调用**：Handler 层不能直接去调数据库，必须通过 Service 层。

---

## 4. Development Standards (开发规范)

### 4.1 Dependency Management (依赖管理)

* **Lock Files**: 必须提交锁文件，确保所有开发者的依赖版本严格一致。
* Go: `go.sum`
* Python: `poetry.lock` 或 `requirements.txt` (含版本号)
* Node: `package-lock.json` / `pnpm-lock.yaml`


* **No Global Install**: 严禁依赖宿主机的全局环境。Python 必须用 venv/conda，Node 必须用本地 node_modules。

### 4.2 Linting & Formatting (代码质量)

所有语言必须配置对应的 Linter 和 Formatter，并在 CI 中强制执行。

* **Go**: `golangci-lint` (Standard)
* **Python**: `Ruff` + `Mypy`
* **Node.js**: `ESLint` + `Prettier`

### 4.3 Error Handling (错误处理)

* **Catch Boundaries**: 不要在每一行代码都 `try-catch`。错误应该向上抛出，直到 Interface 层统一捕获处理。
* **Panic/Crash**: 只有在启动阶段配置错误时才允许 Panic 退出。处理 HTTP 请求时，**严禁**导致整个进程崩溃。

---

## 5. Docker Guidelines (容器化规范)

Dockerfile 是抹平语言差异的最终手段。

1. **Multi-stage Build (多阶段构建)**:
* 必须将编译环境（含 GCC、Go 编译器、Node_modules 缓存）与运行环境分离。
* 生产镜像产物必须尽可能小（使用 `alpine` 或 `distroless`）。


2. **Non-root User**:
* **安全红线**：禁止以 `root` 用户运行进程。Dockerfile 最后必须切换到 `USER app` 或类似非特权用户。


3. **Timezone**:
* 容器内时区统一设置为 `UTC`。



### Template Example (Dockerfile 伪代码)

```dockerfile
# Stage 1: Build
FROM <language-sdk>:latest AS builder
WORKDIR /app
COPY . .
RUN <build-command>  # e.g., go build, npm install

# Stage 2: Runtime
FROM alpine:latest
WORKDIR /app
# 创建非 root 用户
RUN addgroup -S nonroot && adduser -S nonroot -G nonroot
# 仅复制编译产物
COPY --from=builder /app/binary .
# 切换用户
USER nonroot
CMD ["./binary"]

```

---

## 6. Language Specifics (语言特性附录)

虽然我们要通用，但针对当前已引入的语言，做以下特别约定：

### 🟢 Go (Golang)

* **Layout**: 遵循 [Standard Go Project Layout](https://github.com/golang-standards/project-layout)。
* `cmd/`: Main application entry points.
* `internal/`: Private application and library code.


* **Error Handling**: 必须显式处理 `err`，严禁使用 `_` 忽略错误。

### 🔵 Python

* **Type Hints**: 新代码必须包含 Python 3.10+ 类型提示。
* **Async**: 对于 I/O 密集型服务 (FastAPI)，必须使用 `async/await`，禁止使用同步阻塞代码。

### 🟡 Node.js / TypeScript

* **TypeScript**: **严禁**使用纯 JavaScript 开发后端。必须使用 TypeScript 且开启 `strict: true`。
* **Async**: 必须使用 `async/await`，禁止使用 Callback Hell。

---