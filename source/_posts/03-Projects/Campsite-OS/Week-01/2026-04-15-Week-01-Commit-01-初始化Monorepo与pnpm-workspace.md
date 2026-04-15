---
title: "Week 01 - Commit 01: 初始化Monorepo与pnpm workspace"
date: 2026-04-15T17:50:00+11:00
tags: [Monorepo, pnpm, workspace, 工程化, 项目架构, 依赖管理]
categories: 03-Projects/Campsite-OS/Week-01
---

# Week 01 - Commit 01: 初始化Monorepo与pnpm workspace

> 本文记录 Campsite OS 项目 Week 1 的第一个 Commit：初始化 Monorepo 与 pnpm workspace。这是项目工程化地基的第一步。

## Commit 信息
- **Commit Hash**: `[待补充]`
- **提交信息**: `chore: initialize monorepo with pnpm workspace`
- **时间**: 2026年4月15日
- **相关文件**: `package.json`, `pnpm-workspace.yaml`, `tsconfig.json`, `.gitignore`

## 核心知识点整理

### 1. Monorepo 是什么？

**Monorepo（单体仓库）** 类似于一个**超级仓库**。

#### 传统方式 vs Monorepo
| 传统方式 | Monorepo方式 |
|---------|-------------|
| 前端一个文件夹，后端一个文件夹 | 统一管理在一个仓库中 |
| 每个项目独立 `node_modules` | 共享依赖，只安装一次 |
| 重复的依赖存储 | pnpm 把依赖存在全局一个地方 |
| 项目间代码共享困难 | 项目里只是个"快捷方式" |

#### 关键理解
```bash
# 传统：每个项目独立安装
project1/node_modules/  # 500MB
project2/node_modules/  # 500MB（重复内容）
# 总计：1GB

# pnpm workspace：依赖共享
.pnpm-store/            # 500MB（全局存储）
project1/node_modules/  # 快捷方式
project2/node_modules/  # 快捷方式
# 总计：500MB（节省50%空间）
```

### 2. "幽灵依赖"杀手

**幽灵依赖（Phantom Dependencies）**：项目使用了没有在 `package.json` 中声明的依赖。

#### 问题示例
```javascript
// 问题：lodash 没有在 package.json 中声明，但代码能运行
import { debounce } from 'lodash';  // 从哪里来的？

// 原因：npm/yarn 的扁平化 node_modules 结构
// lodash 可能是其他依赖的依赖，被提升到了根目录
```

#### pnpm 解决方案
- **严格模式**：只能访问 `package.json` 中声明的依赖
- **符号链接**：每个包只能看到自己的依赖
- **hoisting control**：可控的提升策略，避免意外访问

### 3. Workspace 协议

在 `pnpm-workspace.yaml` 定义后，你可以用以下命令让 Web 端直接引用本地的共享包：

```bash
# Web端直接引用本地共享包，不需要发到 npm 官网上
pnpm add @campsite-os/shared --workspace
```

#### Workspace 协议的优势
```json
{
  "dependencies": {
    "@shared/utils": "workspace:*"  // 使用 workspace 协议
  }
}
```

- **本地开发**：指向本地包，即时生效
- **发布时自动替换**：构建时会自动替换为实际版本号
- **简化版本管理**：不需要手动协调版本号

## 架构设计理念

### 1. 开启"地块分封制"

告诉工具，我的项目不止在根目录，`apps` 和 `packages` 文件夹里的每一个子文件夹，都是一个独立的包。

#### 目录结构设计
```
campsite-os/
├── apps/                    # "诸侯国" - 可独立运行的应用程序
│   ├── web/                # Next.js 网页（前端王国）
│   └── api/                # NestJS 接口（后端王国）
├── packages/               # "中央库" - 共享的基础设施
│   ├── shared/             # 公共工具函数（电力系统）
│   ├── database/           # 数据库模型（水利系统）
│   └── ui/                 # UI 组件库（建筑材料）
└── 中央调度文件们
```

#### pnpm-workspace.yaml 配置
```yaml
packages:
  - "apps/*"      # 所有 apps 下的子目录都是独立封地
  - "packages/*"  # 所有 packages 下的子目录都是中央资源
```

#### 工作流程
```bash
# 在 Web 里需要用到共享工具
cd apps/web
pnpm add @shared/utils --workspace  # pnpm 在本地自动帮你"接通电线"
```

### 2. 建立"中央调度中心" (package.json)

根目录下的这个文件不再负责写业务代码，而是负责管理全局：

#### 根 package.json 配置
```json
{
  "name": "campsite-os",
  "version": "0.0.0",
  "private": true,
  "description": "Full-stack AI-powered camp operations platform",
  
  "engines": {
    "node": ">=20.11.0",
    "pnpm": ">=9.0.0"
  },
  
  "packageManager": "pnpm@9.12.0",
  
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "typecheck": "turbo run typecheck",
    "clean": "turbo run clean && rm -rf node_modules"
  },
  
  "devDependencies": {
    "turbo": "^2.1.3",
    "typescript": "^5.6.3"
  }
}
```

#### 设计理念解析

1. **依赖管理**：
   - 定义了整个仓库共用的工具（比如 turbo 任务管理、typescript 编译器）
   - 所有子项目共享这些开发依赖

2. **脚本中转**：
   - 你在根目录运行 `pnpm build`，它会通过 `turbo` 命令分发到各个子项目去执行构建
   - 统一入口，简化操作

3. **引擎锁定**：
   - 代码里写了 `"engines": { "node": ">=20.11.0" }`
   - 这是为了防止协作时因为 Node 版本不同导致的"我电脑上能跑，你电脑上不行"的玄学问题

### 3. 配置"全局编译器" (tsconfig.json)

根目录的 `tsconfig.json` 设置为：

```json
{
  "files": [],
  "references": []
}
```

#### 设计理念解析

这是一种 **Project References（项目引用）** 的高级写法。

它告诉 VS Code：
> "根目录本身没有代码，不要尝试在这里编译，去具体的子文件夹（apps/packages）里看它们各自的配置。"

**优势**：
- 极大提升大型项目的代码跳转和编译速度
- 每个包有自己的 TypeScript 配置，互不影响
- 支持增量编译，只编译变更的文件

#### 实际工作方式
```json
{
  "files": [],  // 根目录没有代码文件
  "references": [  // 项目引用，告诉IDE去子目录找配置
    { "path": "apps/web" },
    { "path": "apps/api" },
    { "path": "packages/shared" }
  ]
}
```

### 4. 设立"垃圾回收站" (.gitignore)

它定义了哪些文件不应该上传到 GitHub。

#### .gitignore 配置
```
# Dependencies
node_modules/
.pnpm-store/

# Build outputs
dist/
build/
.next/
.turbo/
*.tsbuildinfo

# Env
.env
.env.local
.env.*.local
!.env.example

# Logs
*.log
npm-debug.log*
pnpm-debug.log*

# IDE
.vscode/
.idea/
*.swp
.DS_Store

# Testing
coverage/
.nyc_output/

# Misc
*.pem
.cache/
```

#### 设计理念解析

1. **依赖排除**：
   - `node_modules/`：那是几百 MB 的依赖包，不需要传，大家自己本地安装即可
   - `.pnpm-store/`：pnpm 的全局存储，也不需要上传

2. **环境保护**：
   - `.env`：里面存的是数据库密码、API 密钥，传上去就相当于把家里钥匙挂在门口
   - 使用 `.env.example` 作为模板，不包含真实密钥

3. **构建产物**：
   - 所有构建输出（`dist/`, `build/`, `.next/`）都应该在本地生成，不需要版本控制

## 为什么这一步是"精通"的起点？

### 从"小作坊"到"工业化流水线"

| 小作坊方式 | 工业化流水线 |
|-----------|-------------|
| 用 `npm init` 搞个文件夹 | 用 pnpm workspace 构建智能容器 |
| 手动复制粘贴代码 | 代码自动共享，即时生效 |
| 依赖重复安装 | 依赖全局共享，节省空间 |
| 版本管理混乱 | 统一版本，避免冲突 |
| 扩展困难 | 清晰结构，轻松扩展 |

### pnpm workspace 带来的三大优势

#### 1. 省空间
- 所有子项目共用的包只会在硬盘存一份
- 通过符号链接实现依赖共享
- 相比传统方式节省 50%+ 磁盘空间

#### 2. 保一致
- 全仓库的 TypeScript 版本、代码格式规则高度统一
- 统一的依赖版本，避免"依赖地狱"
- 一致的开发环境和构建流程

#### 3. 防混乱
- 清晰的目录结构让你在项目变大时（比如以后增加移动端 App、增加后台管理端），只需要在 `apps/` 下新建文件夹即可，地基完全不用动
- 明确的职责划分：`apps/` 放应用，`packages/` 放共享代码
- 避免幽灵依赖和版本冲突

## 一句话总结

**这一步是把一个"空文件夹"变成了一个"支持多项目协作的智能容器"。**

## 技术决策记录（ADR）

### ADR 001: 选择 pnpm 作为包管理器
**状态**：已接受  
**背景**：需要管理多个相关包和应用，传统 npm/yarn 在多包管理上效率低下  
**决策**：使用 pnpm workspace  
**依据**：
1. **磁盘效率**：依赖全局存储，节省 50%+ 磁盘空间
2. **安装速度**：比 npm/yarn 快 2-3 倍
3. **严格性**：解决幽灵依赖问题
4. **workspace 支持**：优秀的 Monorepo 支持

**后果**：
- ✅ 依赖安装更快，磁盘占用更少
- ✅ 避免幽灵依赖导致的构建问题
- ✅ 本地包引用简单直接
- ⚠️ 需要团队统一使用 pnpm
- ⚠️ 某些旧工具可能不完全兼容

## 实际代码示例

### 完整的根目录配置

#### package.json
```json
{
  "name": "campsite-os",
  "version": "0.0.0",
  "private": true,
  "description": "Full-stack AI-powered camp operations platform",
  "engines": {
    "node": ">=20.11.0",
    "pnpm": ">=9.0.0"
  },
  "packageManager": "pnpm@9.12.0",
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "typecheck": "turbo run typecheck",
    "clean": "turbo run clean && rm -rf node_modules"
  },
  "devDependencies": {
    "turbo": "^2.1.3",
    "typescript": "^5.6.3"
  }
}
```

#### pnpm-workspace.yaml
```yaml
packages:
  - "apps/*"
  - "packages/*"
```

#### tsconfig.json
```json
{
  "files": [],
  "references": []
}
```

## 下一步行动

### 立即验证
```bash
# 1. 安装依赖
pnpm install

# 2. 验证 workspace 配置
pnpm list -r

# 3. 创建第一个子项目
cd apps
pnpm dlx @nestjs/cli new api --package-manager pnpm
```

### 后续 Commit 规划
1. **Commit 02**: 初始化 NestJS 后端项目
2. **Commit 03**: 初始化 Next.js 前端项目  
3. **Commit 04**: 创建共享工具包
4. **Commit 05**: 配置 Turborepo 任务管理

## 学习收获

通过这个 Commit，我们掌握了：

1. **Monorepo 架构思想**：从多个独立仓库到统一智能容器
2. **pnpm workspace 核心原理**：依赖共享、幽灵依赖解决、本地包引用
3. **现代工程化配置**：TypeScript 项目引用、Git 策略、引擎锁定
4. **架构设计能力**："地块分封制"、"中央调度中心"等设计模式

**这不仅是技术配置，更是工程思维的提升。** 我们不再只是写代码的程序员，而是设计系统的工程师。

---

*本文记录 Campsite OS 项目 Week 1 的第一个 Commit。通过这个简单的初始化，我们为整个项目奠定了工程化基础，建立了从"小作坊"到"工业化流水线"的架构思维。*