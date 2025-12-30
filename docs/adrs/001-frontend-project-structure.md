# ADR-001: Frontend Project Directory Structure

* **Status**: Accepted
* **Date**: 2025-12-30
* **Author**: @Antigravity
* **Deciders**: @RevieU-Web-Team

## Context (背景)

为了支持 RevieU Web 的快速迭代并行开发，我们需要一个清晰、可扩展且模块化的前端项目结构。目前项目采用了 **Feature-based (基于功能)** 的架构，以减少模块间的耦合。

## Decision (决策)

我们决定采用以下目录结构规范：

```text
revieu-web/
├── 📁 .git/                          # Git版本控制
├── 📁 .github/                       # GitHub配置
│   └── 📁 workflows/
│       └── ci.yml
├── 📁 .kiro/                         # Kiro配置
│   └── 📁 specs/
│       └── 📁 discover-page-refactor/
├── 📁 .vscode/                       # VS Code配置
│   └── settings.json
├── 📁 dist/                          # 构建输出目录
│   ├── 📁 assets/
│   │   ├── index-B7cNKECA.js
│   │   └── index-CvrvVug9.css
│   └── index.html
├── 📁 node_modules/                  # 依赖包
├── 📁 src/                           # 🎯 主要源代码目录
│   ├── 📁 app/                       # 应用入口
│   │   ├── App.tsx                   # 主应用组件
│   │   └── index.tsx                 # 应用入口点
│   ├── 📁 assets/                    # 静态资源
│   │   ├── 📁 icons/
│   │   ├── 📁 images/
│   │   └── 📁 styles/
│   │       └── globals.css           # 全局样式
│   ├── 📁 components/                # 🔧 共享组件
│   │   ├── 📁 common/                # 通用组件
│   │   │   ├── 📁 ImageWithFallback/
│   │   │   ├── 📁 RangeSelector/
│   │   │   └── index.ts
│   │   ├── 📁 layout/                # 布局组件
│   │   │   ├── 📁 BottomNav/
│   │   │   ├── 📁 FAB/
│   │   │   └── index.ts
│   │   ├── 📁 ui/                    # UI基础组件
│   │   │   ├── 📁 Button/
│   │   │   ├── 📁 Card/
│   │   │   ├── 📁 Dialog/
│   │   │   ├── 📁 Input/
│   │   │   ├── index.ts
│   │   │   └── utils.ts
│   │   └── index.ts
│   ├── 📁 config/                    # 配置文件
│   │   └── index.ts                  # 应用配置
│   ├── 📁 contexts/                  # React上下文
│   │   └── AuthContext.tsx           # 认证上下文
│   ├── 📁 features/                  # 🚀 功能模块 (Feature-based架构)
│   │   ├── 📁 auth/                  # 认证功能
│   │   ├── 📁 discover/              # 发现页功能 (已重构)
│   │   ├── 📁 home/                  # 首页功能 (已迁移)
│   │   ├── 📁 profile/               # 个人资料功能 (已迁移)
│   │   └── 📁 reviews/               # 评论功能 (已迁移)
│   └── 📁 shared/                    # 共享资源
│       ├── 📁 api/
│       ├── 📁 constants/
│       ├── 📁 hooks/
│       ├── 📁 types/
│       └── 📁 utils/
├── .gitignore                        # Git忽略文件
├── index.html                        # HTML入口文件
├── metadata.json                     # 项目元数据
```

### 核心设计原则

1.  **Feature-based**: `src/features/` 下的每个文件夹代表一个独立的业务模块（内聚所有相关的 components, hooks, pages, types, utils）。
2.  **Strict Sharing**: 只有真正全局通用的逻辑才放入 `src/components/` 或 `src/shared/`。
3.  **Encapsulation**: 每个 feature 通过其根目录下的 `index.ts` 暴露公有 API，外部不得直接拉取 feature 内部的子文件。

## Consequences (后果)

### ✅ Positive (收益)
*   **高内聚**: 修改某个功能时，代码都在同一个文件夹下。
*   **低耦合**: 功能模块之间逻辑隔离。
*   **易于重构**: 可以轻松移动或替换整个 feature 模块。

### ❌ Negative (成本/风险)
*   **重复代码风险**: 开发者可能在不同 feature 中实现类似的辅助函数（需要定期 code review 提取到 shared）。
*   **路径层级稍深**: 开发时目录跳转较多。

## References (参考)
*   [Feature-based architecture overview]
