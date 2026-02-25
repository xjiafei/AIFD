---
name: tech-design
description: |
  为已批准的功能生成技术设计规格，包括架构设计、API设计、数据库设计、前端设计和测试策略。
  Use when the user wants to create technical design, or mentions '技术设计', 'tech design', 'API设计', '数据库设计'.
---

# 技术设计技能

为已完成产品设计的功能启动技术设计阶段，生成完整的技术实现方案。

## 参数

- 功能名称: {args} (如未提供，询问用户要设计哪个功能)

## 前置条件

⚠️ **必须先完成产品设计**：
- 产品设计文档（PRD、用户故事等）已完成
- 产品设计已通过审核

## 分阶段设计方法

技术设计采用三阶段渐进式方法，确保设计的完整性和可执行性：

```
Phase 0: Research（技术调研）
    ↓ 调研技术方案、分析可行性、记录决策

Phase 1: Design（技术设计）
    ↓ 架构设计、API设计、数据库设计、前端设计

Phase 2: Tasks（任务分解）
    ↓ 将设计分解为可执行的任务清单
```

### Phase 0: Research（技术调研，可选）

**何时需要**：
- ✅ 功能涉及新技术栈或未使用过的框架
- ✅ 有多个可选的技术方案需要对比
- ✅ 需要集成第三方服务或API
- ✅ 性能要求高，需要方案验证

**输出**：`docs/specs/feature{N}-specs/tech/research.md`

**内容**：
- 调研问题列表
- 技术方案对比（方案A vs 方案B）
- 原型验证结果（如需要）
- 技术决策和理由

**示例**：
```markdown
## 技术调研：用户认证方案

### 调研问题
1. 采用JWT还是Session认证？
2. 如何实现Token刷新机制？
3. 如何防止Token被盗用？

### 方案对比

| 方案 | 优点 | 缺点 | 是否采用 |
|------|------|------|----------|
| JWT Token | 无状态，适合分布式 | 无法主动失效 | ✅ 采用 |
| Session | 可主动失效，安全性高 | 需要共享Session，不适合分布式 | ❌ 不采用 |

### 技术决策
采用JWT Token + Redis黑名单机制：
- JWT保证无状态
- Redis黑名单实现主动失效
- 兼顾分布式和安全性
```

### Phase 1: Design（技术设计，必须）

**输出**：完整的技术设计文档
- `architecture.md` - 架构设计
- `api-spec.yaml` - API规格（OpenAPI 3.0）
- `database.md` - 数据库设计
- `frontend.md` - 前端设计

这是本skill的核心阶段，详见下方"工作流程"。

### Phase 2: Tasks（任务分解，推荐）

**何时执行**：Phase 1完成并审核通过后

**执行方式**：使用 `/task-breakdown` skill

**输出**：`docs/specs/feature{N}-specs/tasks.md`

**内容**：按用户故事组织的可执行任务清单

**为什么需要**：
- ✅ 将设计转化为可执行的具体任务
- ✅ 明确任务依赖关系和执行顺序
- ✅ 识别并行执行机会，提高开发效率
- ✅ 支持增量交付和MVP优先策略

---

## 工作流程（Phase 1: Design）

### 步骤1：验证前置条件

#### 1.1 查找功能目录
- 找到与功能名称匹配的 `docs/feature{N}-specs/` 目录
- 如果未找到，通知用户并停止

#### 1.2 阅读产品设计文档
依次阅读以下文档：
- `requirements/requirements.md` - 需求分析
- `product/prd/prd.md` - 产品需求文档
- `product/user_story/stories.md` - 用户故事
- `product/prototype/wireframes.md` - 原型设计

#### 1.3 验证审核状态
- 检查产品设计是否已批准
- 如果未批准，通知用户先完成产品审核

### 步骤2：理解 Baseline（必须首先执行）

⚠️ **关键步骤：在设计新功能前，必须先理解现有系统**

#### 2.1 Baseline 和 Feature-Specs 概念

**Baseline（基线规格）**：
- 位置：`docs/specs/baseline/`
- 定义：项目的全量规格文档，包含所有已实现功能的最新设计
- 作用：理解系统全貌的唯一真实来源

**Feature-Specs（功能规格）**：
- 位置：`docs/specs/feature{N}-specs/`
- 定义：单个特性的增量规格文档
- 作用：隔离单个特性的设计

**工作方式**：
1. 先从 baseline 理解现有系统
2. 在 feature{N}-specs 中设计**仅与此功能相关的增量部分**
3. 交付后，将 feature{N}-specs 的设计合并回 baseline

#### 2.2 阅读 Baseline 架构

**必须阅读的文档**：

```bash
# 1. 项目整体架构
cat docs/specs/baseline/tech/architecture.md
```
理解：
- 项目的技术栈选型（Java 8+, Spring Boot, Vue 3等）
- 整体架构模式（前后端分离、微服务、单体等）
- 已实现的核心模块
- 代码组织结构（Maven多模块、前端目录结构）
- 技术约束和限制

```bash
# 2. 完整 API 规格
cat docs/specs/baseline/tech/api-spec.yaml
```
理解：
- 已实现的所有 API 端点
- API 版本管理策略
- 统一的请求/响应格式
- 统一的错误处理机制
- 认证和授权方式

```bash
# 3. 完整数据库设计
cat docs/specs/baseline/tech/database.md
```
理解：
- 已创建的所有数据表
- 表之间的关系（外键、关联）
- 命名规范和约定
- 索引策略
- 公共字段（created_at, updated_at等）

```bash
# 4. 完整前端设计
cat docs/specs/baseline/tech/frontend.md
```
理解：
- 已实现的页面和组件
- 路由结构
- 状态管理方案（Pinia stores）
- 公共组件库
- API 调用方式

#### 2.3 识别复用和依赖

**确定以下问题**：

1. **可复用的模块**：
   - 新功能可以复用哪些现有的后端服务？
   - 新功能可以复用哪些现有的前端组件？
   - 新功能可以复用哪些现有的数据表？

2. **需要的新模块**：
   - 新功能需要新增哪些后端服务？
   - 新功能需要新增哪些前端组件？
   - 新功能需要新增哪些数据表？

3. **依赖关系**：
   - 新功能依赖哪些现有功能？
   - 新功能是否会影响现有功能？
   - 是否需要修改现有的API或数据模型？

4. **兼容性**：
   - 新设计与现有架构是否兼容？
   - 新API与现有API风格是否一致？
   - 新数据库设计与现有设计是否一致？

#### 2.4 阅读开发规范

```bash
# 后端规范
cat docs/knowledges/standards/java-standards.md
cat docs/knowledges/standards/api-design-standards.md

# 前端规范
cat docs/knowledges/standards/frontend-standards.md

# 数据库规范
cat docs/knowledges/standards/database-design-standards.md
```

#### 2.5 确定增量设计范围

基于对 baseline 的理解，明确：

✅ **需要在 feature{N}-specs 中设计的**：
- 新增的 API 端点
- 新增的数据表和字段
- 新增的前端页面和组件
- 与现有模块的集成方式

❌ **不需要在 feature{N}-specs 中重复的**：
- 已在 baseline 中定义的技术栈
- 已在 baseline 中定义的架构模式
- 已在 baseline 中定义的公共组件
- 已在 baseline 中定义的统一规范

**引用而非重复**：
如果需要引用 baseline 的内容，使用：
```markdown
本功能复用 baseline 中的用户认证机制（见 baseline/tech/api-spec.yaml#authentication）
```

### 步骤3：系统架构设计

#### 3.1 设计功能架构

基于对 baseline 的理解，设计此功能如何融入整体系统架构：

#### 2.2 设计功能架构

设计此功能如何融入整体系统架构：

**组件设计**
- 后端组件：Controller、Service、Repository、Entity
- 前端组件：Pages、Components、Stores、API clients
- 中间件组件：如需要

**数据流图**
```
用户操作 → 前端组件 → API调用 → 后端Controller
                                    ↓
                               Service层（业务逻辑）
                                    ↓
                               Repository层（数据访问）
                                    ↓
                                  数据库
```

**集成点**
- 与现有模块的集成
- 第三方服务集成（如有）
- 外部API调用（如有）

**依赖关系**
- 依赖的其他功能模块
- 依赖的第三方库
- 依赖的基础设施服务

**输出文档**：`docs/feature{N}-specs/tech/arch/architecture.md`

### 步骤3：后端设计

#### 3.1 阅读标准和规范
- 阅读 `docs/knowledges/standards/` 中的后端编码规范
- 阅读 `docs/knowledges/domain/` 中的领域模型
- 阅读 `docs/knowledges/templates/backend-patterns/` 中的设计模式

#### 3.2 API设计

设计RESTful API端点：

**API端点清单**
```
POST   /api/users          创建用户
GET    /api/users/{id}     获取用户详情
PUT    /api/users/{id}     更新用户信息
DELETE /api/users/{id}     删除用户
GET    /api/users          查询用户列表
```

**API详细设计**（每个端点）：

```markdown
### POST /api/users

**描述**：创建新用户

**请求**：
- Method: POST
- Path: /api/users
- Headers:
  - Content-Type: application/json
  - Authorization: Bearer {token}

- Body:
```json
{
  "username": "string",
  "email": "string",
  "password": "string"
}
```

**响应**：
- 成功（200）：
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "username": "zhangsan",
    "email": "zhangsan@example.com",
    "createdAt": "2026-02-13T10:00:00Z"
  }
}
```

- 失败（400）：
```json
{
  "code": 400,
  "message": "用户名已存在"
}
```

**业务规则**：
- 用户名长度4-20个字符
- 邮箱格式验证
- 密码至少8位，包含字母和数字

**权限**：需要admin角色
```

**输出文档**：`docs/feature{N}-specs/tech/backend_design/api-spec.md`

#### 3.3 数据库设计

设计数据模型和表结构：

**ER图描述**
```
User (用户表)
  ├─ has many → Order (订单表)
  └─ has many → Address (地址表)
```

**DDL脚本**：
```sql
-- 用户表
CREATE TABLE `users` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `email` varchar(100) NOT NULL COMMENT '邮箱',
  `password` varchar(255) NOT NULL COMMENT '密码（加密）',
  `status` tinyint NOT NULL DEFAULT 1 COMMENT '状态：1-正常，0-禁用',
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_username` (`username`),
  UNIQUE KEY `uk_email` (`email`),
  KEY `idx_status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

**输出文档**：`docs/feature{N}-specs/tech/backend_design/db-schema.sql`

#### 3.4 服务层设计

设计业务逻辑和验证规则：

**Service接口**：
```java
public interface UserService {
    /**
     * 创建用户
     */
    UserVO createUser(CreateUserRequest request);

    /**
     * 获取用户详情
     */
    UserVO getUserById(Long id);

    /**
     * 更新用户信息
     */
    UserVO updateUser(Long id, UpdateUserRequest request);

    /**
     * 删除用户
     */
    void deleteUser(Long id);

    /**
     * 查询用户列表
     */
    PageResult<UserVO> listUsers(UserQueryRequest request);
}
```

**业务规则**：
- 创建用户时检查用户名和邮箱唯一性
- 密码需要加密存储（BCrypt）
- 删除用户需要检查是否有关联数据
- 更新用户信息需要权限验证

**输出文档**：`docs/feature{N}-specs/tech/backend_design/service-design.md`

### 步骤4：前端设计

#### 4.1 阅读标准和模式
- 阅读 `docs/knowledges/standards/` 中的前端编码规范
- 阅读 `docs/knowledges/templates/frontend-patterns/` 中的前端模式
- 阅读 `docs/knowledges/ui-guidelines/` 中的UI设计规范（如有）

#### 4.2 页面和路由设计

**路由结构**：
```javascript
// router/index.ts
const routes = [
  {
    path: '/users',
    name: 'UserList',
    component: () => import('@/views/user/UserList.vue'),
    meta: { title: '用户列表', requiresAuth: true }
  },
  {
    path: '/users/create',
    name: 'UserCreate',
    component: () => import('@/views/user/UserCreate.vue'),
    meta: { title: '创建用户', requiresAuth: true }
  },
  {
    path: '/users/:id',
    name: 'UserDetail',
    component: () => import('@/views/user/UserDetail.vue'),
    meta: { title: '用户详情', requiresAuth: true }
  }
]
```

**输出文档**：`docs/feature{N}-specs/tech/frontend_design/routing.md`

#### 4.3 组件设计

**组件层次结构**：
```
views/user/
├── UserList.vue          # 用户列表页面
├── UserCreate.vue        # 创建用户页面
└── UserDetail.vue        # 用户详情页面

components/user/
├── UserForm.vue          # 用户表单组件（创建/编辑共用）
├── UserCard.vue          # 用户卡片组件
└── UserTable.vue         # 用户表格组件
```

**组件规格**（每个组件）：

```markdown
### UserForm.vue

**描述**：用户表单组件，支持创建和编辑

**Props**：
- `mode`: 'create' | 'edit' - 表单模式
- `initialData`: UserFormData | undefined - 初始数据（编辑时）

**Emits**：
- `submit`: (data: UserFormData) => void - 提交表单
- `cancel`: () => void - 取消操作

**数据结构**：
```typescript
interface UserFormData {
  username: string
  email: string
  password?: string
}
```

**验证规则**：
- username: 必填，4-20字符
- email: 必填，邮箱格式
- password: 创建时必填，至少8位
```

**输出文档**：`docs/feature{N}-specs/tech/frontend_design/component-spec.md`

#### 4.4 状态管理设计

**Store设计**（使用Pinia）：

```typescript
// stores/user.ts
export const useUserStore = defineStore('user', {
  state: () => ({
    users: [] as User[],
    currentUser: null as User | null,
    loading: false,
    total: 0
  }),

  actions: {
    async fetchUsers(params: UserQueryParams) {
      this.loading = true
      try {
        const result = await userApi.listUsers(params)
        this.users = result.data
        this.total = result.total
      } finally {
        this.loading = false
      }
    },

    async createUser(data: CreateUserRequest) {
      const user = await userApi.createUser(data)
      this.users.unshift(user)
      return user
    }
  }
})
```

**输出文档**：`docs/feature{N}-specs/tech/frontend_design/state-management.md`

### 步骤5：测试设计

#### 5.1 阅读测试模式
- 阅读 `docs/knowledges/templates/testing-patterns/` 中的测试模板

#### 5.2 测试策略设计

**单元测试**（后端）：
```markdown
### UserServiceTest

**测试类**：`UserServiceTest.java`

**测试用例**：
1. testCreateUser_Success - 创建用户成功
2. testCreateUser_DuplicateUsername - 用户名重复
3. testCreateUser_DuplicateEmail - 邮箱重复
4. testGetUserById_Success - 获取用户成功
5. testGetUserById_NotFound - 用户不存在
6. testUpdateUser_Success - 更新用户成功
7. testDeleteUser_Success - 删除用户成功
```

**单元测试**（前端）：
```markdown
### UserForm.spec.ts

**测试用例**：
1. 渲染表单元素
2. 表单验证 - 用户名为空
3. 表单验证 - 邮箱格式错误
4. 提交表单 - 成功
5. 提交表单 - 失败
```

**集成测试**：
```markdown
### User API集成测试

**测试用例**：
1. POST /api/users - 创建用户成功
2. POST /api/users - 用户名已存在（400）
3. GET /api/users/{id} - 获取用户成功
4. GET /api/users/{id} - 用户不存在（404）
5. PUT /api/users/{id} - 更新用户成功
6. DELETE /api/users/{id} - 删除用户成功
```

**E2E测试**：
```markdown
### 用户管理E2E测试

**测试场景**：
1. 用户注册流程
   - 访问注册页面
   - 填写表单
   - 提交
   - 验证注册成功

2. 用户登录流程
   - 访问登录页面
   - 输入用户名密码
   - 提交
   - 验证跳转到首页
```

**输出文档**：`docs/feature{N}-specs/tech/test_design/test-cases.md`

#### 5.3 测试数据设计

```markdown
### 测试数据

**用户测试数据**：
```json
{
  "validUser": {
    "username": "testuser",
    "email": "test@example.com",
    "password": "Test1234"
  },
  "invalidUser": {
    "username": "a",  // 太短
    "email": "invalid",  // 格式错误
    "password": "123"  // 太短
  }
}
```
```

**输出文档**：`docs/feature{N}-specs/tech/test_design/test-data.md`

### 步骤6：部署计划

#### 6.1 数据库迁移

**迁移脚本**：
```sql
-- V1.0_create_users_table.sql
-- 创建用户表
CREATE TABLE `users` (...);

-- 插入初始数据（如需要）
INSERT INTO `users` (...) VALUES (...);
```

#### 6.2 配置更改

**环境变量**：
```properties
# 数据库配置
DB_HOST=localhost
DB_PORT=3306
DB_NAME=mydb
DB_USER=root
DB_PASSWORD=******

# JWT配置
JWT_SECRET=your-secret-key
JWT_EXPIRATION=86400
```

#### 6.3 发布计划

**发布步骤**：
1. 停止服务（如需要）
2. 备份数据库
3. 执行数据库迁移
4. 部署新版本代码
5. 更新配置
6. 重启服务
7. 健康检查
8. 验证核心功能

#### 6.4 回滚计划

**回滚步骤**：
1. 停止服务
2. 回滚代码到上一版本
3. 回滚数据库（执行回滚脚本）
4. 恢复配置
5. 重启服务
6. 验证

**输出文档**：`docs/feature{N}-specs/deploy/deployment-plan.md`

### 步骤7：请求审核

#### 7.1 创建技术设计摘要

```markdown
# Feature {N} 技术设计文档摘要

## 功能概述
{一段话描述功能}

## 技术方案
- 后端：Spring Boot + MySQL
- 前端：Vue 3 + Pinia
- 部署：Docker + Docker Compose

## 核心文档
- ✅ 架构设计：docs/feature{N}-specs/tech/arch/architecture.md
- ✅ API设计：docs/feature{N}-specs/tech/backend_design/api-spec.md
- ✅ 数据库设计：docs/feature{N}-specs/tech/backend_design/db-schema.sql
- ✅ 前端组件设计：docs/feature{N}-specs/tech/frontend_design/component-spec.md
- ✅ 测试策略：docs/feature{N}-specs/tech/test_design/test-cases.md
- ✅ 部署计划：docs/feature{N}-specs/deploy/deployment-plan.md

## 关键技术点
1. 技术点1
2. 技术点2
3. 技术点3

## 待确认事项
1. 事项1
2. 事项2

## 下一步
审核通过后，使用 `/implement {feature-name}` 开始代码实现。
```

#### 7.2 请求用户审核

明确告知用户：
- 技术设计文档已完成
- 请审核所有文档
- 指出任何需要修改的地方
- 确认是否批准进入代码实现阶段

#### 7.3 记录审核反馈

在 `docs/feature{N}-specs/review_records/tech_review/review-{timestamp}.md` 中记录：
- 审核时间
- 审核人
- 审核意见
- 修改建议
- 审核结论（通过/待修改/不通过）

#### 7.4 迭代或推进

- **如果需要修改**：根据反馈修改文档，重新请求审核
- **如果批准**：通知用户可以继续执行 `/implement {feature-name}`

## 重要说明

### 1. 一致性检查
- 技术设计必须与产品设计保持一致
- API设计要符合产品PRD的功能需求
- 数据库设计要满足业务规则
- 前端设计要符合原型设计

### 2. 遵循规范
- 所有设计必须遵循 `docs/knowledges/standards/` 中的编码规范
- 参考 `docs/knowledges/templates/` 中的设计模板
- 遵循 `docs/knowledges/best-practices/` 中的最佳实践

### 3. 详细程度
- 要足够具体和详细以便开发人员直接实现
- API设计要包含请求/响应示例
- 数据库设计要包含完整的DDL脚本
- 组件设计要包含Props、Emits、数据结构

### 4. 边界处理
- 包括错误处理策略
- 考虑边界情况和异常场景
- 设计降级方案（如需要）
- 设计限流和熔断（如需要）

### 5. 性能考虑
- 考虑可扩展性（用户量增长）
- 考虑性能优化（响应时间、并发量）
- 设计缓存策略（如需要）
- 设计索引优化

### 6. 文档质量
- 记录所有技术假设
- 说明技术选型的权衡（trade-off）
- 提供清晰的图表和示例
- 确保文档可维护性

## 输出示例

```
✅ 技术设计已完成！

📁 Feature编号：feature003

📄 已生成文档：
- ✅ 架构设计：docs/feature003-specs/tech/arch/architecture.md
- ✅ API设计：docs/feature003-specs/tech/backend_design/api-spec.md
- ✅ 数据库设计：docs/feature003-specs/tech/backend_design/db-schema.sql
- ✅ 服务设计：docs/feature003-specs/tech/backend_design/service-design.md
- ✅ 组件设计：docs/feature003-specs/tech/frontend_design/component-spec.md
- ✅ 路由设计：docs/feature003-specs/tech/frontend_design/routing.md
- ✅ 状态管理：docs/feature003-specs/tech/frontend_design/state-management.md
- ✅ 测试用例：docs/feature003-specs/tech/test_design/test-cases.md
- ✅ 部署计划：docs/feature003-specs/deploy/deployment-plan.md

🎯 技术方案：
- 后端：Spring Boot + MySQL + Redis
- 前端：Vue 3 + Pinia + Element Plus
- 部署：Docker + Docker Compose

📊 数据库变更：
- 新增表：users, user_sessions
- 新增索引：3个

🔌 API端点：
- POST /api/auth/login
- POST /api/auth/logout
- GET /api/auth/me

📋 待确认事项：
1. 是否需要Redis缓存用户会话？
2. JWT token过期时间设置为多久？

---

请审核以上技术设计文档，确认无误后回复"批准"或提出修改意见。

审核通过后，使用 `/implement 用户登录` 开始代码实现。
```
