
# Global API Standards
### 全局接口规范与契约

> **Scope**: 适用于 RevieU 所有后端微服务 (Go/Python/Node) 对外提供的 HTTP/RESTful 接口。
> **Principle**: 所有服务必须“说同一种语言”。无论后端内部实现多么复杂，返回给前端的 JSON 结构必须严格一致。

---

## 1. URL & Method Conventions (URL 命名)

我们遵循标准的 **RESTful** 风格。

* **Prefix**: 所有接口统一加 `/api/v1` 前缀。
* **Case**: URL 路径使用 `kebab-case` (短横线命名)，**禁止**使用 `camelCase` 或 `snake_case`。
    * ✅ `GET /api/v1/user-profiles`
    * ❌ `GET /api/v1/user_profiles`
* **Resources**: 使用复数名词 (Plural Nouns)。
    * ✅ `GET /api/v1/reviews`
    * ❌ `GET /api/v1/review`
* **Methods**:
    * `GET`: 获取资源 (Idempotent)
    * `POST`: 创建资源
    * `PUT`: 全量更新资源
    * `PATCH`: 部分更新资源
    * `DELETE`: 删除资源

---

## 2. Request Headers (请求头)

前端发起请求时，必须包含以下标准 Header：

| Header Name | Required | Example | Description |
| :--- | :--- | :--- | :--- |
| **`Content-Type`** | Yes | `application/json` | 仅支持 JSON 格式交互。 |
| **`Authorization`** | Yes | `Bearer <JWT_TOKEN>` | 身份认证令牌。 |
| **`X-Request-ID`** | **Yes** | `a1b2-c3d4-e5f6` | **全链路追踪 ID**。通常由 API Gateway 生成，若未生成，前端需生成一个 UUID 传入。 |
| **`X-Language`** | No | `zh-CN` | 国际化语言标识。 |

---

## 3. Response Envelope (统一响应体)

无论请求成功还是失败，后端**必须**返回以下标准 JSON 结构（Envelope Pattern）。前端通过 `code` 判断业务逻辑，而非 HTTP Status。

### 3.1 Standard Structure (标准结构)

```json
{
  "code": 0,          // 业务状态码 (0 或 200 代表成功，非 0 代表失败)
  "msg": "success",   // 提示信息 (用于前端直接 toast 展示)
  "data": { ... },    // 实际业务数据 (Object 或 Array)
  "trace_id": "..."   // 方便排查问题的 Request ID (透传)
}

```

### 3.2 Success Example (成功示例)

HTTP Status: `200 OK`

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "id": 101,
    "username": "zhangsan",
    "email": "zhangsan@revieu.com"
  },
  "trace_id": "c71a3683-1256-433b-8523-952044874415"
}

```

### 3.3 Error Example (失败示例)

HTTP Status: `200 OK` (或者 400/401/403/500，取决于 HTTP 语义，但 JSON 结构不变)

```json
{
  "code": 10401,            // 具体的业务错误码 (例如: 余额不足)
  "msg": "Insufficient balance", // 错误提示
  "data": null,
  "trace_id": "c71a3683-1256-433b-8523-952044874415"
}

```

> **注意**: 后端在发生 `Panic` 或 `Exception` 时，**严禁**直接返回语言层面的堆栈信息 (Stack Trace)，必须捕获并包装成上述 JSON 格式返回 500 错误。

---

## 4. Pagination (分页规范)

列表查询必须支持分页，**严禁**一次性返回所有数据。

### 4.1 Request (请求参数)

使用 Query Parameters 传递：

* `page`: 页码，从 1 开始 (Default: 1)。
* `page_size`: 每页条数 (Default: 20, Max: 100)。

### 4.2 Response (响应结构)

分页数据统一放在 `data` 内部，结构如下：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "list": [ ... ],        // 当前页的数据列表
    "pagination": {         // 分页元数据
      "total": 100,         // 总条数
      "page": 1,            // 当前页码
      "page_size": 20,      // 每页条数
      "total_pages": 5      // 总页数
    }
  },
  "trace_id": "..."
}

```

---

## 5. Data Formats (数据格式约定)

为了避免前后端解析差异，特定类型数据遵循以下格式：

| Type | Format | Example | Note |
| --- | --- | --- | --- |
| **Date/Time** | **ISO 8601** | `2025-12-20T14:30:00Z` | 统一使用 UTC 时间，由前端转换时区显示。**严禁**使用时间戳 (Timestamp)。 |
| **Money** | **String** | `"19.99"` | **严禁**使用 Float/Double 传输金额（精度丢失问题）。或者使用整数传输“分” (`1999`)。 |
| **Long ID** | **String** | `"9223372036854775807"` | 超过 JS Number 最大值的 ID (如 Snowflake ID) 必须转为 String 传输。 |
| **Boolean** | **Bool** | `true` / `false` | 不要使用 0/1 或 "true"/"false"。 |
| **Null** | **Null** | `null` | 字段无值时返回 null，**不要**返回空字符串 "" 或空对象 {}。 |

---

## 6. Business Code Dictionary (业务错误码字典)

我们使用 5 位数字作为业务错误码。

* `0`: 成功
* `10xxx`: 通用/系统错误
* `20xxx`: 用户服务 (User Service) 错误
* `30xxx`: 评论服务 (Review Service) 错误

| Code | Message | Description |
| --- | --- | --- |
| **0** | success | 请求成功 |
| **10400** | Bad Request | 参数校验失败 (ValidationError) |
| **10401** | Unauthorized | 未登录或 Token 过期 |
| **10403** | Forbidden | 权限不足 (无权访问该资源) |
| **10500** | Internal Server Error | 服务器内部崩溃 |
| **20001** | User not found | 用户不存在 |
| **20002** | User already exists | 用户名已占用 |
| **30001** | Review not found | 评论不存在 |

---