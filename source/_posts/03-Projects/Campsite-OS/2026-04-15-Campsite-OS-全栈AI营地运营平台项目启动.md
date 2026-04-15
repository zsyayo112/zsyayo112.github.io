---
title: Campsite OS — 全栈 AI 营地运营平台项目启动（Week 1 完成）
date: 2026-04-15T17:45:00+11:00
tags: [全栈开发, Monorepo, TypeScript, NestJS, Next.js, AI应用, 项目重构, 求职作品]
categories: 03-Projects/Campsite-OS
---

# Campsite OS — 全栈 AI 营地运营平台项目启动（Week 1 完成）

> 本文记录 Campsite OS 项目的完整规划与 Week 1 工程化地基搭建。这是一个从零重构的真实营地运营管理系统，作为求职作品项目。

## 项目背景

我正在从零重构一个营地运营管理系统，作为求职作品。前身是 [campsite-manage](https://github.com/zsyayo112/campsite-manage)，一个 Express + CRA + JS 的 MVP，服务真实营地业务（长白山双溪森林营地）。v1 已上线运行，沉淀了 17 个 Prisma 模型、订单/预约/接送/套餐/SQL Server 双写等真实业务逻辑。

新仓库 [campsite-os](https://github.com/zsyayo112/campsite-os) 是**重生而非迭代** — 技术栈、架构、目录结构全部重做。v1 仅作为业务规格参考，代码不复用。

## 我的身份与目标

- **目标**：用这个项目找全栈/AI 应用方向的工作
- **当前水平**：能写 React/Express，但 TS、NestJS、Next.js App Router、AI 应用开发都是新学。需要在做项目的过程中同步学八股
- **时间预算**：8 周，平均 3-4 小时/天

## 项目叙事定位（面试一句话）

> 把一个真实运行的营地业务系统（原 17 模型、订单/接送/双写）从 JS+CRA 单体重构为 TS+NestJS+Next.js 全栈 Monorepo，精简至 13 个领域模型，接入 AI 助手与 RAG，带完整可观测与 CI/CD。

### 三个差异化卖点：
1. **真实业务背书**（非 todo demo）
2. **重构叙事**（讲得出 v1 痛点 和迁移决策）
3. **AI 不是套壳**（工具调用 + RAG + Text-to-SQL 三件套落到具体业务）

## 业务范围（已锁定）

四个 bounded context：

- **identity**：User（后台员工）、Customer（游客）
- **campsite**：Package、PackageItem、PickupLocation
- **booking**：Booking、Order、OrderItem、Payment
- **ai**：Conversation、Message、KnowledgeDoc、KnowledgeChunk

**已砍功能**：接送调度（Vehicle/Driver/Coach/ShuttleSchedule）、SQL Server 双写、行程排期、小红书内容管理、住宿房型库存。上车地点降级为 Order.pickupLocationId + 字典表。

## 真实业务需求（营地实际使用）

### 公开侧（游客）
- 营地介绍、活动、套餐展示
- 在线预约表单（移动端优先，微信内可用）
- 订单状态自助查询（手机号+预约码）

### 后台侧（运营）
- 预约管理：查看新预约、确认、记录定金、转订单
- 订单管理：状态流转（pending → confirmed → completed）、收款记录
- 客户管理：客户档案、消费历史、来源追踪
- 套餐/活动管理：内容编辑、图片/视频、公开展示配置
- 上车地点统计：按日期+地点汇总当日参与人数
- Dashboard：今日营收、订单数、客户数、营收趋势图

**v2.1 延后**：微信小程序、接送调度、行程排期。

## 技术栈（已锁定）

| 类别 | 技术栈 |
|------|--------|
| **Monorepo** | pnpm + Turborepo |
| **后端** | NestJS + Prisma + PostgreSQL + Redis + BullMQ + pgvector |
| **前端** | Next.js 14 App Router + TanStack Query + shadcn/ui + Tailwind + React Hook Form + Zod |
| **AI** | Vercel AI SDK + LangChain.js + OpenAI/Claude API |
| **可观测** | Sentry + OpenTelemetry + Prometheus + Grafana |
| **测试** | Jest + Supertest + Playwright |
| **部署** | Docker Compose + Nginx + GitHub Actions + GHCR |
| **规范** | ESLint flat config + Prettier + Husky + commitlint + conventional commits |

## 八周路线图

| 周次 | 主题 | 核心任务 |
|------|------|----------|
| **Week 1** | 工程化地基 | monorepo, CI, lint, ADR, README+架构图 ✅ **DONE** |
| **Week 2-3** | 后端 TS + NestJS 迁移 | Controller/Service/Repository 三层，全局 ExceptionFilter，class-validator DTO，Swagger，JWT + Refresh Token |
| **Week 4** | 前端 Next.js 14 | 后台 CSR+TanStack Query，公开页 SSR/ISR |
| **Week 5** | 缓存与可观测 | Redis 缓存 + BullMQ 队列 + OTel/Prom/Grafana + 限流 + 索引 review |
| **Week 6** | AI 三件套 | 自然语言预约助手(深) / RAG 智能客服(通) / Text-to-SQL 运营分析(原型)，时间分配 5:3:2 |
| **Week 7** | 容器化部署 | Docker 多阶段构建 + docker-compose + Nginx + GHCR 自动部署 |
| **Week 8** | 测试与文档 | 测试覆盖 + ADR + 架构图 + README 重写 |

### 砍优先级（时间不够时）
1. OTel全家桶 > Text-to-SQL > E2E

### 不能砍
- TS、NestJS 分层、Next.js SSR
- 至少一个 AI 功能
- Docker、README+架构图

## 部署节奏

1. **Week 1**：GitHub Actions CI（lint + build）✅
2. **Week 4 后**：Railway + Vercel 预览环境（给招聘方点链接）
3. **Week 7**：单台 VPS + Docker Compose

---

# Week 1 完成：工程化地基搭建

## Commit 1: chore: initialize monorepo with pnpm workspace

### 目录结构
```
campsite-os/
├── apps/                    # 可独立运行的应用程序
│   ├── web/                # Next.js 前端（待创建）
│   └── api/                # NestJS 后端（待创建）
├── packages/               # 共享代码包
│   ├── shared/             # 共享工具函数（待创建）
│   ├── database/           # 数据库模型（待创建）
│   └── ui/                 # UI 组件库（待创建）
├── package.json            # 根 package.json（中央调度中心）
├── pnpm-workspace.yaml     # workspace 配置
├── tsconfig.json           # 根 TypeScript 配置
├── turbo.json             # Turborepo 任务管理（待创建）
└── .gitignore             # Git 忽略配置
```

### 1. 根目录 package.json（中央调度中心）
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

**设计理念**：
- `engines`：锁定 Node 和 pnpm 版本，防止"我电脑能跑，你电脑不行"的玄学问题
- `packageManager`：强制使用 pnpm，避免团队协作时的包管理器不一致
- `scripts`：通过 turbo 命令分发到各个子项目执行构建、开发、测试等任务

### 2. pnpm-workspace.yaml（工作区配置）
```yaml
packages:
  - "apps/*"
  - "packages/*"
```

**设计理念**：
- **"地块分封制"**：`apps/` 下的每个子目录都是可独立运行的应用程序
- **"中央库"**：`packages/` 下的每个子目录都是共享代码包
- **本地依赖**：通过 `pnpm add @shared/utils --workspace` 直接引用本地包，无需发布到 npm

### 3. 根目录 tsconfig.json（全局编译器配置）
```json
{
  "files": [],
  "references": []
}
```

**设计理念**：
- `"files": []`：根目录没有代码文件，只是一个容器
- `"references": []`：通过项目引用告诉 VS Code 去子目录找配置
- **优势**：极大提升大型项目的代码跳转和编译速度

### 4. .gitignore（垃圾回收站）
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

**设计理念**：
- **依赖排除**：`node_modules/` 和 `.pnpm-store/` 不需要上传，节省仓库空间
- **环境保护**：`.env` 文件包含敏感信息，绝对不能上传
- **构建产物**：所有构建输出都应该在本地生成，不需要版本控制

## Monorepo 架构核心理解

### 1. Monorepo 是什么？
**Monorepo（单体仓库）**：将多个相关项目（包、应用）放在同一个代码仓库中管理。

### 2. pnpm workspace 的优势

#### 空间效率
```bash
# 传统方式：每个项目独立 node_modules，重复存储
project1/node_modules/  # 500MB
project2/node_modules/  # 500MB（重复内容）
# 总计：1GB

# pnpm workspace：依赖共享，符号链接
.pnpm-store/            # 500MB（全局存储）
project1/node_modules/  # 快捷方式
project2/node_modules/  # 快捷方式
# 总计：500MB
```

#### 开发效率
```bash
# 传统方式：需要发布到 npm 才能共享
npm publish @shared/utils
cd apps/web
npm install @shared/utils@latest

# pnpm workspace：本地即时共享
cd apps/web
pnpm add @shared/utils --workspace  # 立即生效
```

#### 协作效率
- **版本一致**：所有项目使用相同依赖版本
- **原子提交**：相关更改可以一起提交
- **统一构建**：一次命令构建所有相关项目

### 3. "幽灵依赖"杀手
**幽灵依赖（Phantom Dependencies）**：项目使用了没有在 package.json 中声明的依赖。

```javascript
// 问题：lodash 没有在 package.json 中声明，但代码能运行
import { debounce } from 'lodash';  // 从哪里来的？

// 原因：npm/yarn 的扁平化 node_modules 结构
// lodash 可能是其他依赖的依赖，被提升到了根目录
```

**pnpm 解决方案**：
- **严格模式**：只能访问 package.json 中声明的依赖
- **符号链接**：每个包只能看到自己的依赖
- **hoisting control**：可控的提升策略，避免意外访问

### 4. Workspace 协议
```json
{
  "dependencies": {
    "@shared/utils": "workspace:*"  // 使用 workspace 协议
  }
}
```

**优势**：
- **本地开发**：指向本地包，即时生效
- **发布时自动替换**：构建时会自动替换为实际版本号
- **简化版本管理**：不需要手动协调版本号

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

### ADR 002: 采用 Turborepo 作为任务运行器
**状态**：已接受  
**背景**：需要高效管理跨包构建、测试、开发任务  
**决策**：使用 Turborepo  
**依据**：
1. **智能缓存**：基于文件哈希的缓存，极大提升重复构建速度
2. **任务管道**：定义任务依赖关系，自动优化执行顺序
3. **远程缓存**：支持团队共享构建缓存
4. **Vercel 生态**：与 Next.js 深度集成

### ADR 003: 项目引用式 TypeScript 配置
**状态**：已接受  
**背景**：传统 tsconfig 扩展在大型 Monorepo 中维护困难  
**决策**：使用 TypeScript 项目引用  
**依据**：
1. **编译性能**：只编译变更的文件
2. **IDE 支持**：VS Code 能正确跳转到定义
3. **隔离性**：每个包有自己的配置，互不影响
4. **增量构建**：充分利用 TypeScript 的增量编译

## 下一步计划（Week 2-3）

### 后端迁移核心任务
1. **NestJS 项目初始化**
   ```bash
   cd apps/api
   pnpm dlx @nestjs/cli new api --package-manager pnpm
   ```

2. **Prisma Schema 设计**
   ```prisma
   // 基于 v1 的 17 个模型，精简为 13 个
   model User {
     id        String   @id @default(cuid())
     email     String   @unique
     name      String?
     role      UserRole @default(STAFF)
     createdAt DateTime @default(now())
   }
   ```

3. **三层架构实现**
   - Controller：处理 HTTP 请求
   - Service：业务逻辑
   - Repository：数据访问

4. **全局异常处理**
   ```typescript
   @Catch()
   export class GlobalExceptionFilter implements ExceptionFilter {
     catch(exception: unknown, host: ArgumentsHost) {
       // 统一错误响应格式
     }
   }
   ```

5. **DTO 与验证**
   ```typescript
   export class CreateBookingDto {
     @IsString()
     @IsNotEmpty()
     customerName: string;
     
     @IsDate()
     @Type(() => Date)
     bookingDate: Date;
   }
   ```

## 项目价值与学习目标

### 技术成长维度
1. **架构设计**：Monorepo、DDD、Clean Architecture
2. **全栈开发**：NestJS + Next.js 完整工作流
3. **AI 集成**：工具调用、RAG、Text-to-SQL 实战
4. **工程化**：CI/CD、容器化、可观测性
5. **团队协作**：代码规范、提交约定、文档驱动

### 求职价值
1. **真实业务背书**：不是又一个 Todo App
2. **技术深度展示**：覆盖现代全栈技术栈
3. **问题解决能力**：重构决策、技术选型、权衡取舍
4. **完整项目周期**：从设计到部署的全流程

## 总结

Week 1 成功搭建了 Campsite OS 的工程化地基：

✅ **Monorepo 架构确立**：pnpm workspace + Turborepo  
✅ **开发规范制定**：TypeScript 项目引用、Git 忽略策略  
✅ **团队协作基础**：引擎锁定、包管理器统一  
✅ **技术决策文档**：ADR 记录关键决策依据  

**核心收获**：
- 理解了 Monorepo 从"小作坊"到"工业化流水线"的质变
- 掌握了 pnpm workspace 解决幽灵依赖和空间效率的原理
- 建立了 TypeScript 项目引用的最佳实践
- 为后续 7 周开发奠定了坚实的工程基础

**下一步**：Week 2-3 开始后端 NestJS 迁移，将 v1 的业务逻辑用现代 TypeScript 全栈技术重新实现。

---

*本文记录 Campsite OS 项目 Week 1 的完整工作。通过这个项目，不仅是在构建一个营地管理系统，更是在实践中掌握现代全栈开发的工程化方法和架构设计思想。*