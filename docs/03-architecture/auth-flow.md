# Authentication & Authorization Flow
### 认证与鉴权架构

> **Architecture**: JWT (JSON Web Token) + Gateway Pattern.
> **Objective**: 实现无状态 (Stateless) 认证，确保微服务之间的信任传递安全高效。

---

## 1. Overview (概览)

RevieU 采用 **双 Token 机制 (Access Token + Refresh Token)** 结合 **API Gateway 集中鉴权** 的模式。

* **Frontend**: 负责 Token 的存储、携带与自动刷新。
* **API Gateway**: 负责 Token 的**验签 (Signature Verification)** 和黑名单检查。
* **Microservices**: 不处理复杂的登录逻辑，只信任网关透传的 User ID。

---

## 2. Token Strategy (令牌策略)

我们使用两种不同用途的 Token，以平衡安全性与用户体验。

| Token Type | Storage (Frontend) | Expiry | Purpose |
| :--- | :--- | :--- | :--- |
| **Access Token** | Memory (Zustand/Context) | **Short** (15 min) | 访问业务接口。放在 HTTP Header `Authorization: Bearer` 中。 |
| **Refresh Token** | **HttpOnly Cookie** | **Long** (7 Days) | 仅用于换取新的 Access Token。**严禁** JS 读取，防止 XSS 攻击窃取。 |



---

## 3. Detailed Workflows (详细流程)

### 3.1 Login Flow (登录流程)
1.  用户输入账号密码，前端调用 `POST /api/v1/auth/login`。
2.  **Auth Service** 验证密码成功。
3.  **Auth Service** 生成两个 Token：
    * `access_token` (JWT): 包含 `user_id`, `role`, `exp`。
    * `refresh_token` (Opaque/JWT): 存入数据库/Redis (用于允许强制下线)。
4.  **Response**:
    * Body: 返回 `access_token` 给前端 JS。
    * Header: Set-Cookie `refresh_token` (HttpOnly, Secure, SameSite=Strict)。

### 3.2 Request Guard (请求鉴权 - 核心)
*这是微服务鉴权最关键的一步：网关充当守门员。*

1.  前端发起请求 `GET /api/v1/reviews`，Header 携带 `Authorization: Bearer <access_token>`。
2.  **API Gateway** (Nginx/Kong/APISIX) 拦截请求：
    * **验证签名**: 使用公钥检查 JWT 是否被篡改。
    * **检查过期**: 检查 `exp` 字段是否过期。
3.  **Token Valid (合法)**:
    * 网关解析 JWT，提取 `sub` (User ID) 和 `role`。
    * 网关将 ID 放入新的 Header：`X-User-ID: 1001` 和 `X-User-Role: user`。
    * **转发**请求给下游的 **Review Service**。
4.  **Token Invalid (非法/过期)**:
    * 网关直接返回 `401 Unauthorized`，请求**不会**到达 Review Service。

> **对于后端开发的意义**:
> 在 `review-service` (Python/Go) 的代码中，**你不需要解析 JWT，也不需要去查 User 表**。你只需要读取 Header 里的 `X-User-ID`。
> * 如果有 `X-User-ID`，说明网关已经验过了，此人可信。

### 3.3 Silent Refresh (静默刷新)
前端 `axios` 拦截器负责处理 Token 过期。

1.  前端发起请求，收到 `401 Unauthorized`。
2.  前端拦截器挂起当前请求。
3.  调用 `POST /api/v1/auth/refresh` (浏览器会自动带上 HttpOnly Cookie 里的 Refresh Token)。
4.  **Auth Service** 验证 Refresh Token 合法且未被撤销。
5.  返回新的 `access_token`。
6.  前端重试刚才失败的请求。
7.  **用户全程无感知**。

---

## 4. Microservices Security (微服务内部安全)

### 4.1 Trust Boundary (信任边界)
* **External Requests (外部请求)**: 必须经过 Gateway，必须有 Token。
* **Internal Requests (内部调用)**: 服务 A 调用 服务 B (gRPC/HTTP)。
    * 原则上内网是可信的，但为了纵深防御，建议服务间调用也携带 `X-Internal-Secret` 或 mTLS 证书。
    * 在 RevieU v1 阶段，我们假设 Kubernetes Cluster IP 网络是安全的，直接透传 `X-User-ID` Header。

### 4.2 Permission Control (RBAC)
* **Gateway 层**: 负责 coarse-grained (粗粒度) 鉴权。
    * 例如：`/admin/*` 路径只允许 `X-User-Role: admin` 访问。
* **Service 层**: 负责 fine-grained (细粒度) 鉴权。
    * 例如：删除评论接口 `DELETE /reviews/:id`，Review Service 必须检查：`current_user_id == review.author_id`。

---

## 5. Implementation Guide (开发指南)

### Frontend (React)
使用我们在 `revieu-web` 中封装好的 `api-client.ts`，它已经内置了拦截器：

```typescript
// 伪代码示例
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      const newToken = await refreshToken(); // 自动调刷新接口
      setAccessToken(newToken);
      // 修改原请求头，重试
      originalRequest.headers.Authorization = `Bearer ${newToken}`;
      return api(originalRequest);
    }
    return Promise.reject(error);
  }
);

```

### Backend (Go/Python)

不要在业务代码里写 JWT 解析逻辑！请编写一个简单的 Middleware 读取 Header：

```go
// Go Middleware 示例
func UserIdentity(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        userID := r.Header.Get("X-User-ID")
        if userID == "" {
            // 防御性编程：如果网关配置错了，这里要拦截
            http.Error(w, "Missing Identity", http.StatusUnauthorized)
            return
        }
        // 将 userID 注入 context 传给 Controller
        ctx := context.WithValue(r.Context(), "userID", userID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

```

---

## 6. Security Checklist (安全红线)

1. **HTTPS Only**: 生产环境严禁使用 HTTP 传输 Token。
2. **No Sensitive Data**: Access Token 的 Payload 中**禁止**放入用户的手机号、邮箱等隐私信息（只放 ID 和 Role），因为 Base64 是可以被任何人解码的。
3. **Strong Secret**: JWT 的签名密钥 (Secret Key) 必须是 32 位以上的随机字符串，且必须存储在 K8s Secret 中，**严禁**硬编码在代码里。

```

---

### 💡 架构师视角的解读

1.  **为什么 Access Token 放在内存，Refresh Token 放在 Cookie？**
    * **防 XSS**: 如果 Access Token 存在 localStorage，黑客注入一段 JS 就能偷走你的 Token。放在内存里（变量），刷新页面就没了，黑客很难偷。
    * **防 CSRF**: Refresh Token 放在 Cookie 里容易被 CSRF，所以我们设置 `SameSite=Strict`，并且该接口只能做“刷新”这一件事，不能做业务操作，风险可控。

2.  **网关透传 Header (`X-User-ID`)**：
    * 这是大厂微服务最核心的设计。**业务服务应当是“傻”的**，它不需要知道什么是 JWT，什么是 RSA 加密。它只知道：“Header 里有 ID，说明网关大哥已经查过身份证了，我直接干活。”
    * 这大大降低了多语言开发的成本（你不用给 Go、Python、Node 每个语言都写一套复杂的 JWT 校验库）。

**到这里，你的 Handbook 核心架构已经非常稳固了。**
有了这份文档，你的后端开发无论用什么语言，都知道怎么拿 UserID；前端也知道怎么处理 401 自动刷新。协作效率直接起飞。

```