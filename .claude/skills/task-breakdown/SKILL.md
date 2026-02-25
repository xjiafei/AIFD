---
name: task-breakdown
description: |
  根据功能设计文档生成可执行的任务分解清单，支持增量交付和独立测试。
  Use when the user wants to break down feature into tasks, or mentions '任务分解', 'task breakdown', '生成任务清单', '任务列表'.
---

# 任务分解技能

你现在是一位经验丰富的项目管理专家，擅长将功能设计分解为可执行的、依赖关系清晰的任务清单。

## 核心理念

### 1. 按用户故事组织任务

**核心原则**：任务必须按用户故事分组，确保每个故事可以：
- ✅ 独立实现
- ✅ 独立测试
- ✅ 独立部署
- ✅ 作为MVP增量交付

**为什么这样做？**
```
传统方式（按技术层组织）：
├── 所有模型
├── 所有服务
├── 所有API
├── 所有前端
└── 难以独立测试和交付

推荐方式（按用户故事组织）：
├── 用户故事1（P1）→ 完整的端到端功能
├── 用户故事2（P2）→ 完整的端到端功能
├── 用户故事3（P3）→ 完整的端到端功能
└── 每个故事都可独立测试和部署
```

### 2. 严格的任务格式

**所有任务必须遵循此格式**：

```
- [ ] [TaskID] [P?] [Story?] 描述 + 文件路径
```

**格式说明**：
1. **Checkbox**：`- [ ]`（Markdown复选框）
2. **Task ID**：顺序编号（T001, T002, T003...）
3. **[P] 标记**：可并行任务（不同文件，无依赖关系）
4. **[Story] 标签**：用户故事标签（[US1], [US2], [US3]...）
   - Setup阶段：无故事标签
   - Foundational阶段：无故事标签
   - User Story阶段：必须有故事标签
   - Polish阶段：无故事标签
5. **描述**：清晰的操作说明 + 确切的文件路径

**示例**：
```markdown
✅ 正确：
- [ ] T001 创建项目结构
- [ ] T005 [P] 实现认证中间件 backend/api/src/main/java/com/company/api/middleware/AuthMiddleware.java
- [ ] T012 [P] [US1] 创建User实体 backend/domain/src/main/java/com/company/domain/entity/User.java
- [ ] T014 [US1] 实现UserService backend/service/src/main/java/com/company/service/UserService.java

❌ 错误：
- [ ] 创建User模型（缺少ID和Story标签）
- T001 [US1] 创建模型（缺少checkbox）
- [ ] [US1] 创建User模型（缺少Task ID）
- [ ] T001 [US1] 创建模型（缺少文件路径）
```

### 3. 阶段化组织

**标准阶段结构**：

```
阶段1: Setup（搭建）
    ↓ 项目初始化和基础结构

阶段2: Foundational（基础核心）
    ↓ 所有用户故事依赖的阻塞性前提

阶段3+: User Stories（按优先级）
    ├─ 阶段3: 用户故事1（P1）🎯 MVP
    ├─ 阶段4: 用户故事2（P2）
    ├─ 阶段5: 用户故事3（P3）
    └─ ...

最终阶段: Polish（优化）
    └ 跨故事的优化和完善
```

**关键点**：
- ⚠️ **Foundational阶段完成前，禁止开始任何用户故事工作**
- ✅ Foundational完成后，所有用户故事可并行开发
- ✅ 每个用户故事都是完整的端到端功能

## 工作流程

### 步骤1：读取设计文档

**必须读取的文档**：

```bash
# 1. 产品设计（用户故事来源）
cat docs/specs/feature{N}-specs/product/prd/prd.md
cat docs/specs/feature{N}-specs/product/user-stories/user-stories.md

# 2. 技术设计
cat docs/specs/feature{N}-specs/tech/architecture.md
cat docs/specs/feature{N}-specs/tech/api-spec.yaml
cat docs/specs/feature{N}-specs/tech/database.md
cat docs/specs/feature{N}-specs/tech/frontend.md
```

**提取关键信息**：
- 用户故事列表及其优先级（P1, P2, P3...）
- 技术架构和技术栈
- API端点及其与用户故事的映射
- 数据库实体及其与用户故事的映射
- 前端页面/组件及其与用户故事的映射

### 步骤2：生成任务清单

#### 2.1 阶段1: Setup（搭建）

**目的**：项目初始化

**典型任务**：
```markdown
## 阶段1: Setup（项目搭建）

**目的**：初始化项目结构和基础配置

- [ ] T001 创建项目目录结构
- [ ] T002 初始化后端项目 backend/pom.xml
- [ ] T003 初始化前端项目 frontend/package.json
- [ ] T004 [P] 配置代码检查工具 backend/checkstyle.xml
- [ ] T005 [P] 配置前端代码规范 frontend/.eslintrc.js
```

#### 2.2 阶段2: Foundational（基础核心）

**目的**：所有用户故事的阻塞性前提

**⚠️ 关键**：此阶段完成前，禁止开始任何用户故事工作

**典型任务**：
```markdown
## 阶段2: Foundational（基础核心）

**目的**：所有用户故事依赖的核心基础设施

**⚠️ 关键**：本阶段完成前，不得开始任何用户故事工作

- [ ] T006 搭建数据库Schema和迁移框架
- [ ] T007 [P] 实现认证授权框架 backend/api/src/main/java/com/company/api/filter/JwtAuthFilter.java
- [ ] T008 [P] 搭建API路由结构 backend/api/src/main/java/com/company/api/config/WebConfig.java
- [ ] T009 [P] 创建公共基础实体 backend/domain/src/main/java/com/company/domain/entity/BaseEntity.java
- [ ] T010 [P] 配置错误处理和日志 backend/common/src/main/java/com/company/common/exception/GlobalExceptionHandler.java
- [ ] T011 [P] 搭建前端API客户端 frontend/src/utils/request.ts
- [ ] T012 [P] 搭建前端路由框架 frontend/src/router/index.ts

**检查点**：基础就绪 - 用户故事可并行开始
```

#### 2.3 阶段3+: User Stories（按优先级）

**每个用户故事的标准结构**：

```markdown
## 阶段3: 用户故事1 - [标题]（优先级: P1）🎯 MVP

**目标**：[该故事交付的核心价值]

**独立测试方式**：[如何验证该故事独立运行]

### 用户故事1的实现

- [ ] T013 [P] [US1] 创建User实体 backend/domain/src/main/java/com/company/domain/entity/User.java
- [ ] T014 [P] [US1] 创建UserRepository backend/infrastructure/src/main/java/com/company/infrastructure/repository/UserRepository.java
- [ ] T015 [US1] 实现UserService backend/service/src/main/java/com/company/service/impl/UserServiceImpl.java（依赖T013, T014）
- [ ] T016 [P] [US1] 创建UserDTO backend/domain/src/main/java/com/company/domain/dto/CreateUserRequest.java
- [ ] T017 [US1] 实现UserController backend/api/src/main/java/com/company/api/controller/UserController.java
- [ ] T018 [P] [US1] 创建用户列表页面 frontend/src/views/user/UserList.vue
- [ ] T019 [P] [US1] 创建用户API调用函数 frontend/src/api/user.ts
- [ ] T020 [US1] 集成前后端，验证完整流程

**检查点**：用户故事1完全可用且可独立测试
```

**关键要点**：
- 每个用户故事包含完整的端到端功能（后端+前端）
- 明确标注 `[US1]`, `[US2]`, `[US3]` 等故事标签
- 提供独立测试方式
- 设置检查点验证故事完整性

#### 2.4 最终阶段: Polish（优化）

**目的**：跨故事的优化和完善

```markdown
## 阶段N: Polish（优化与完善）

**目的**：影响多个用户故事的改进工作

- [ ] TXXX [P] 更新文档 docs/README.md
- [ ] TXXX 代码清理与重构
- [ ] TXXX [P] 性能优化
- [ ] TXXX [P] 安全加固
- [ ] TXXX 执行完整联调测试
```

### 步骤3：生成依赖关系图

```markdown
## 依赖关系与执行顺序

### 阶段依赖

- **Setup阶段（阶段1）**：无依赖 - 可立即开始
- **Foundational阶段（阶段2）**：依赖Setup完成 - 阻塞所有用户故事
- **User Stories阶段（阶段3+）**：均依赖Foundational完成
  - 用户故事之间可并行开发（如有资源）
  - 或按优先级顺序串行（P1 → P2 → P3）
- **Polish阶段（最终）**：依赖所有目标用户故事完成

### 用户故事依赖

- **用户故事1（P1）**：Foundational完成后可开始 - 无其他故事依赖
- **用户故事2（P2）**：Foundational完成后可开始 - 可能与US1集成，但需可独立测试
- **用户故事3（P3）**：Foundational完成后可开始 - 可能与US1/US2集成，但需可独立测试

### 单个用户故事内部

- 先实体，后仓储
- 先仓储，后服务
- 先服务，后控制器
- 先后端API，后前端调用
- 先核心实现，后集成验证
```

### 步骤4：标识并行执行机会

```markdown
## 并行执行机会

### Setup阶段
- 所有标记 [P] 的任务可并行执行

### Foundational阶段
- 所有标记 [P] 的任务可并行执行（阶段内）

### User Stories阶段
- Foundational完成后，所有用户故事可并行开始（如团队容量允许）
- 单个用户故事内，所有标记 [P] 的任务可并行执行
- 不同团队成员可并行处理不同用户故事

### 并行示例：用户故事1

```bash
# 同时创建用户故事1的所有实体：
任务: "创建User实体 backend/domain/src/main/java/com/company/domain/entity/User.java"
任务: "创建UserRepository backend/infrastructure/src/main/java/com/company/infrastructure/repository/UserRepository.java"
任务: "创建UserDTO backend/domain/src/main/java/com/company/domain/dto/CreateUserRequest.java"

# 同时创建用户故事1的所有前端组件：
任务: "创建用户列表页面 frontend/src/views/user/UserList.vue"
任务: "创建用户API调用函数 frontend/src/api/user.ts"
```
```

### 步骤5：定义实施策略

```markdown
## 实施策略

### MVP优先（仅用户故事1）

1. 完成阶段1: Setup
2. 完成阶段2: Foundational（关键 - 阻塞所有故事）
3. 完成阶段3: 用户故事1
4. **暂停并验证**：独立测试用户故事1
5. 就绪后部署/演示

### 增量交付

1. 完成Setup + Foundational → 基础就绪
2. 加入用户故事1 → 独立测试 → 部署/演示（MVP！）
3. 加入用户故事2 → 独立测试 → 部署/演示
4. 加入用户故事3 → 独立测试 → 部署/演示
5. 每个故事新增价值且不破坏已有故事

### 团队并行策略

多开发者协作时：

1. 团队共同完成Setup + Foundational阶段
2. Foundational完成后：
   - 开发者A: 负责用户故事1
   - 开发者B: 负责用户故事2
   - 开发者C: 负责用户故事3
3. 各故事独立完成并集成
```

### 步骤6：生成任务文档

**保存位置**：`docs/specs/feature{N}-specs/tasks.md`

**文档结构**：
```markdown
# 任务清单: [功能名称]

**输入**: 来自 docs/specs/feature{N}-specs/ 的设计文档
**前提条件**: PRD、技术设计文档已完成并审核通过

## 概述

[功能的简要说明，来自PRD]

## 任务组织说明

- **[P]**: 可并行执行的任务
- **[US1]**, **[US2]**: 任务所属的用户故事
- 描述中包含确切文件路径

## 阶段1: Setup（项目搭建）

[Setup任务列表]

## 阶段2: Foundational（基础核心）

[Foundational任务列表]

## 阶段3: 用户故事1 - [标题]（优先级: P1）🎯 MVP

[用户故事1任务列表]

## 阶段4: 用户故事2 - [标题]（优先级: P2）

[用户故事2任务列表]

## 阶段N: Polish（优化与完善）

[优化任务列表]

## 依赖关系与执行顺序

[依赖关系图]

## 并行执行机会

[并行执行说明]

## 实施策略

[MVP优先、增量交付、团队并行策略]

## 任务统计

- **总任务数**: {N}
- **Setup**: {N}个任务
- **Foundational**: {N}个任务
- **用户故事1**: {N}个任务
- **用户故事2**: {N}个任务
- **Polish**: {N}个任务
- **可并行任务**: {N}个（标记[P]）
```

## 任务映射规则

### 从PRD映射任务

**用户故事** → **完整的功能阶段**

示例：
```
PRD中的用户故事：
"作为系统管理员，我想要管理用户账户，以便控制系统访问权限"

映射到任务：
阶段3: 用户故事1 - 用户账户管理（P1）
├── 创建User实体
├── 创建UserRepository
├── 实现UserService（CRUD）
├── 创建UserDTO（Request/Response）
├── 实现UserController（API）
├── 创建用户管理页面
├── 创建用户API调用
└── 集成验证
```

### 从技术设计映射任务

**API端点** → **用户故事任务**

示例：
```
api-spec.yaml中的端点：
POST /api/users
GET /api/users/{id}
PUT /api/users/{id}
DELETE /api/users/{id}

映射到任务：
- [ ] T017 [US1] 实现UserController
  - 实现POST /api/users（创建用户）
  - 实现GET /api/users/{id}（查询用户）
  - 实现PUT /api/users/{id}（更新用户）
  - 实现DELETE /api/users/{id}（删除用户）
  文件：backend/api/src/main/java/com/company/api/controller/UserController.java
```

**数据库实体** → **用户故事任务**

示例：
```
database.md中的表：
users（用户表）
user_roles（用户角色表）

映射到任务：
- [ ] T013 [P] [US1] 创建User实体
  backend/domain/src/main/java/com/company/domain/entity/User.java
- [ ] T014 [P] [US1] 创建UserRole实体
  backend/domain/src/main/java/com/company/domain/entity/UserRole.java
```

**前端页面** → **用户故事任务**

示例：
```
frontend.md中的页面：
用户列表页（UserList.vue）
用户详情页（UserDetail.vue）

映射到任务：
- [ ] T018 [P] [US1] 创建用户列表页面
  frontend/src/views/user/UserList.vue
- [ ] T019 [P] [US1] 创建用户详情页面
  frontend/src/views/user/UserDetail.vue
```

## 质量检查标准

生成任务清单后，必须验证：

### 格式检查
- ✅ 所有任务都有checkbox `- [ ]`
- ✅ 所有任务都有Task ID（T001, T002...）
- ✅ 用户故事阶段的任务都有Story标签（[US1], [US2]...）
- ✅ 所有任务描述都包含文件路径

### 完整性检查
- ✅ 每个用户故事都有完整的端到端任务（后端+前端）
- ✅ 每个用户故事都有"独立测试方式"说明
- ✅ 每个用户故事都有检查点
- ✅ Foundational阶段包含所有共享基础设施

### 可执行性检查
- ✅ 任务描述清晰，LLM可直接执行
- ✅ 文件路径准确，符合项目结构
- ✅ 依赖关系明确，执行顺序合理
- ✅ 并行机会已标识

### 独立性检查
- ✅ 每个用户故事可独立实现
- ✅ 每个用户故事可独立测试
- ✅ 用户故事1可作为MVP独立部署

## 输出报告

生成任务清单后，向用户报告：

```markdown
## 任务分解完成报告

✅ 任务清单已生成：docs/specs/feature{N}-specs/tasks.md

### 任务统计

- **总任务数**: {N}
- **Setup**: {N}个任务
- **Foundational**: {N}个任务（阻塞所有用户故事）
- **用户故事1（P1 - MVP）**: {N}个任务
- **用户故事2（P2）**: {N}个任务
- **用户故事3（P3）**: {N}个任务
- **Polish**: {N}个任务
- **可并行任务**: {N}个（标记[P]）

### 并行执行机会

- Setup阶段：{N}个任务可并行
- Foundational阶段：{N}个任务可并行
- 用户故事阶段：{N}个故事可并行开发（如有资源）

### MVP范围建议

建议MVP仅包含：
- 阶段1: Setup
- 阶段2: Foundational
- 阶段3: 用户故事1（P1）

预计任务数：{N}个

### 增量交付计划

1. MVP（用户故事1） - 预计{N}个任务
2. V1.1（+用户故事2） - 预计+{N}个任务
3. V1.2（+用户故事3） - 预计+{N}个任务

### 下一步

请审核任务清单，确认后可使用 `/implement` 开始实现。
```

## 注意事项

### ✅ 必须遵守

1. **严格任务格式** - 所有任务必须包含checkbox、ID、文件路径
2. **按用户故事组织** - 每个故事是完整的端到端功能
3. **独立可测** - 每个用户故事必须可独立测试
4. **MVP优先** - 用户故事1必须可作为MVP独立交付
5. **文件路径准确** - 符合项目结构（backend/frontend分离）

### ❌ 避免

1. 按技术层组织任务（所有模型、所有服务...）
2. 模糊的任务描述（"实现用户功能"）
3. 缺少文件路径
4. 用户故事之间强依赖（无法独立测试）
5. 忽略并行执行机会

## 执行步骤

当用户调用此skill时：

1. **读取设计文档**：PRD、技术设计、API规格、数据库设计、前端设计
2. **提取用户故事**：从PRD中提取用户故事及优先级
3. **生成Setup任务**：项目初始化任务
4. **生成Foundational任务**：识别所有用户故事的共享依赖
5. **生成User Story任务**：为每个用户故事生成完整的端到端任务
6. **生成Polish任务**：跨故事的优化任务
7. **标识并行机会**：标记所有可并行任务为[P]
8. **生成依赖关系图**：明确执行顺序
9. **定义实施策略**：MVP优先、增量交付、团队并行
10. **质量检查**：验证格式、完整性、可执行性、独立性
11. **保存文档**：保存到 docs/specs/feature{N}-specs/tasks.md
12. **生成报告**：向用户报告任务统计和建议

## 参考资源

- PRD模板：`docs/knowledges/templates/product-templates/prd-template.md`
- 技术设计模板：`docs/knowledges/templates/tech-templates/`（待创建）
- 任务清单示例：`docs/knowledges/templates/tech-templates/tasks-template.md`（待创建）
