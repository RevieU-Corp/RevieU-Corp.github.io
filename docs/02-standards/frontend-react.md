# Frontend Guidelines (React)
### 前端开发与架构规范

> **Stack**: React 18+, TypeScript, Tailwind CSS, TanStack Query, Zustand.
> **Objective**: 构建可维护、类型安全、高性能的单页应用 (SPA)。

---

## 1. Project Structure (目录架构)

我们采用 **Feature-based** (按业务功能划分) 的目录结构，而不是单纯按技术文件类型划分。这能确保当我们要修改“评论功能”时，不需要在 components, api, types 文件夹之间来回跳跃。

```text
src/
├── assets/                # 静态资源 (Images, Icons)
├── components/            # 全局通用 UI 组件 (非业务相关)
│   ├── ui/                # 基础原子组件 (Button, Input, Card)
│   └── layout/            # 布局组件 (Header, Sidebar)
├── features/              # 🟢 核心业务模块 (按领域划分)
│   ├── auth/              # 登录注册模块
│   │   ├── components/    # 该业务独有的组件 (LoginForm)
│   │   ├── hooks/         # 该业务独有的 Hooks (useAuth)
│   │   ├── api/           # 该业务的 API 请求定义
│   │   └── types/         # 该业务的 TS 类型定义
│   └── reviews/           # 评论模块
├── hooks/                 # 全局通用 Hooks (useDebounce, useToggle)
├── lib/                   # 第三方库配置 (Axios instance, QueryClient)
├── router/                # 路由配置
├── stores/                # 全局状态管理 (Zustand)
└── utils/                 # 纯函数工具类 (formatDate, validators)

```

---

## 2. Naming Conventions (命名规范)

### 2.1 Files & Folders

* **Components**: 使用 `PascalCase`。
* ✅ `UserProfile.tsx`, `PrimaryButton.tsx`


* **Non-Components**: 使用 `camelCase` (hooks, utils) 或 `kebab-case` (configs)。
* ✅ `useAuth.ts`, `api-client.ts`



### 2.2 Component Names

* 组件名必须与文件名一致。
* **严禁**使用 `index.tsx` 作为组件文件名（调试时会全是 `index`，非常痛苦）。
* ✅ `UserProfile/UserProfile.tsx`
* ❌ `UserProfile/index.tsx`



---

## 3. Component Patterns (组件规范)

### 3.1 Functional Components Only

严禁使用 Class Component。所有新代码必须使用 **Functional Component + Hooks**。

### 3.2 Named Exports

使用 **Named Export** 而不是 Default Export。这样有利于 IDE 自动补全和重构。

```tsx
// ✅ Correct
export const UserCard = () => { ... }

// ❌ Avoid
const UserCard = () => { ... }
export default UserCard;

```

### 3.3 Props Interface

必须显式定义 Props 接口，不要用 `any`。

```tsx
interface UserCardProps {
  username: string;
  avatarUrl?: string; // Optional
  onFollow: (id: string) => void;
}

export const UserCard = ({ username, avatarUrl, onFollow }: UserCardProps) => {
  // ...
};

```

---

## 4. State Management (状态管理策略)

不要把所有数据都塞进全局 Store。请遵循以下优先级：

1. **URL State**: 搜索参数、分页页码、筛选条件。**优先同步到 URL**，以便用户刷新页面后状态保留。
* *Tool*: `react-router-dom` (`useSearchParams`)


2. **Server State**: 后端接口返回的数据。**必须使用 React Query** 管理缓存和加载状态。
* *Tool*: `@tanstack/react-query`


3. **Local State**: 仅在当前组件内使用的 UI 状态（如弹窗开关、表单输入）。
* *Tool*: `useState`, `useReducer`


4. **Global Client State**: 跨组件共享的非业务数据（如主题模式、用户信息、全局 Toast）。
* *Tool*: `Zustand`



> **❌ Anti-Pattern**: 不要手动在 `useEffect` 里调用 API 然后存进 Redux/Zustand。让 React Query 去处理缓存、重试和竞态问题。

---

## 5. Styling with Tailwind (样式规范)

### 5.1 Utility First

尽量直接在 className 中写 Tailwind 类名。只有极其复杂的、需要复用的样式才抽取到 CSS 文件或 `@layer components` 中。

### 5.2 Conditional Classes

使用 `clsx` 或 `tailwind-merge` 处理条件样式。**严禁**使用字符串拼接。

```tsx
import { cn } from '@/lib/utils'; // 假设封装了 clsx + twMerge

// ✅ Correct
<button className={cn("bg-blue-500", isDisabled && "opacity-50 cursor-not-allowed")}>
  Submit
</button>

// ❌ Avoid
<button className={`bg-blue-500 ${isDisabled ? 'opacity-50' : ''}`}>

```

### 5.3 Color Palette

严禁使用硬编码的颜色值（Hex/RGB）。必须使用 Tailwind 配置文件中定义的语义化颜色，以支持暗黑模式切换。

* ✅ `text-primary`, `bg-background`
* ❌ `text-[#333333]`, `bg-white`

---

## 6. Data Fetching & API (数据交互)

前端必须严格遵守 [Global API Standards](https://www.google.com/search?q=./api-specs.md)。

### 6.1 Axios Instance

禁止直接使用 `fetch` 或 `axios.get`。必须使用我们在 `src/lib/api-client.ts` 中封装好的实例。
该实例会自动处理：

1. **Base URL**: 自动拼接 `/api/v1`。
2. **Auth Token**: 自动在 Header 注入 `Authorization: Bearer ...`。
3. **Response Interceptor**: 自动解包后端返回的 `Envelope` 结构，直接返回 `data` 字段；如果 `code !== 0`，自动抛出异常。
4. **Trace ID**: 记录后端的 `trace_id` 用于报错排查。

### 6.2 Service Layer

API 调用必须封装在 `features/<feature>/api` 目录下的函数中。

```tsx
// src/features/auth/api/login.ts
import { api } from '@/lib/api-client';
import { User } from '../types';

export const loginWithEmail = async (data: LoginDTO): Promise<User> => {
  // 这里不需要处理 code !== 0 的情况，拦截器处理了
  return api.post('/auth/login', data); 
};

```

---

## 7. TypeScript Rules (类型规范)

* **Strict Mode**: `tsconfig.json` 中必须开启 `strict: true`。
* **No Any**: **严禁**使用 `any` 类型。如果类型太复杂，使用 `unknown` 并配合类型守卫 (Type Guard)。
* **API Types**: 后端接口返回的数据必须定义 Interface。
* *Tip*: 如果后端 Swagger 文档完善，可以使用工具自动生成 TS 类型，避免手写。



---

## 8. Performance Best Practices (性能红线)

1. **Memoization**: 对于复杂的计算逻辑，使用 `useMemo`；对于传给子组件的回调函数，使用 `useCallback`，防止子组件无意义重渲染。
2. **Lazy Loading**: 路由组件 (Pages) 必须使用 `React.lazy` 和 `Suspense` 进行代码分割 (Code Splitting)，减小首屏体积。
3. **Image Optimization**: 图片必须指定 `width` 和 `height`，防止由于图片加载导致的布局抖动 (CLS)。

```