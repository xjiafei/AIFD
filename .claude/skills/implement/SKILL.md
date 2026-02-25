---
name: implement
description: |
  为已批准技术设计的功能自动生成代码，包括后端、前端、测试和部署脚本。
  Use when the user wants to implement code, or mentions '实现', 'implement', '生成代码', '代码实现'.
---

# 代码实现技能

为已批准技术设计的功能启动自动化代码实现阶段，生成后端、前端、测试和部署代码。

## 参数

- 功能名称: {args} (如未提供，询问用户要实现哪个功能)

## 前置条件

⚠️ **必须先完成技术设计并通过审核**：
- 技术设计文档（架构、API、数据库、前端设计等）已完成
- 技术设计已通过审核

## 工作流程

### 步骤1：验证前置条件

#### 1.1 查找功能目录
- 找到与功能名称匹配的 `docs/feature{N}-specs/` 目录
- 如果未找到，通知用户并停止

#### 1.2 阅读技术设计文档
依次阅读以下文档：
- `tech/arch/architecture.md` - 架构设计
- `tech/backend_design/api-spec.md` - API设计
- `tech/backend_design/db-schema.sql` - 数据库设计
- `tech/backend_design/service-design.md` - 服务设计
- `tech/frontend_design/component-spec.md` - 组件设计
- `tech/frontend_design/routing.md` - 路由设计
- `tech/frontend_design/state-management.md` - 状态管理
- `tech/test_design/test-cases.md` - 测试用例
- `deploy/deployment-plan.md` - 部署计划

#### 1.3 验证审核状态
- 检查 `review_records/tech_review/` 中是否有审核记录
- 确认技术设计已批准
- 如果未批准，通知用户先完成技术审核

### 步骤2：生成忽略文件配置

#### 2.1 创建 .genignore 文件

在开始生成代码之前，创建 `.genignore` 文件来保护不应被覆盖的文件。

**目的**：防止代码生成工具覆盖重要文件（如已修改的配置、手动编写的代码等）

**.genignore 文件位置和内容**：

**后端 .genignore**（`backend/.genignore`）：
```gitignore
# 配置文件（可能包含敏感信息）
application.yml
application.properties
application-*.yml

# 已手动修改的核心文件
src/main/java/com/company/Application.java
src/main/java/com/company/config/SecurityConfig.java

# 测试配置
src/test/resources/application-test.yml

# 构建文件
pom.xml  # 如果已手动添加依赖

# 文档
README.md
```

**前端 .genignore**（`frontend/.genignore`）：
```gitignore
# 配置文件
.env
.env.local
.env.production

# 已手动修改的核心文件
src/main.ts
src/App.vue
src/router/index.ts  # 如果已有复杂路由逻辑

# 构建配置
vite.config.ts
package.json  # 如果已手动添加依赖

# 文档
README.md
```

**测试 .genignore**（`testing/.genignore`）：
```gitignore
# 配置文件
pytest.ini
conftest.py  # 如果已有复杂的fixture设置

# 测试数据
data/production_data.json  # 真实生产数据

# 文档
README.md
```

**重要提示**：
- ✅ .genignore 保护已手动修改或包含敏感信息的文件
- ✅ 代码生成前先检查 .genignore，跳过被忽略的文件
- ✅ 如果需要重新生成被忽略的文件，先从 .genignore 中移除

### 步骤3：后端实现

#### 3.1 准备工作
- 导航到 `backend/`（注意：项目结构已更新，不再使用 `repos/` 前缀）
- 创建 `backend/.genignore` 文件
- 阅读现有后端代码以了解项目结构和代码风格
- 从 `docs/knowledges/standards/` 阅读后端编码规范

#### 2.2 生成后端代码

**数据库迁移脚本**（Flyway/Liquibase）
```sql
-- repos/backend/infrastructure/src/main/resources/db/migration/V1.0__create_users_table.sql
CREATE TABLE `users` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL,
  ...
);
```

**实体类**（JPA Entity）
```java
// repos/backend/domain/src/main/java/com/company/domain/entity/User.java
@Entity
@Table(name = "users")
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    // ... 其他字段
}
```

**Repository接口**
```java
// repos/backend/infrastructure/src/main/java/com/company/infrastructure/repository/UserRepository.java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
}
```

**Service接口和实现**
```java
// repos/backend/service/src/main/java/com/company/service/UserService.java
public interface UserService {
    UserVO createUser(CreateUserRequest request);
    // ... 其他方法
}

// repos/backend/service/src/main/java/com/company/service/impl/UserServiceImpl.java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    @Override
    public UserVO createUser(CreateUserRequest request) {
        // 业务逻辑实现
    }
}
```

**Controller**
```java
// repos/backend/api/src/main/java/com/company/api/controller/UserController.java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @PostMapping
    public ResponseEntity<Result<UserVO>> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserVO user = userService.createUser(request);
        return ResponseEntity.ok(Result.success(user));
    }
}
```

**DTO（数据传输对象）**
```java
// repos/backend/domain/src/main/java/com/company/domain/dto/CreateUserRequest.java
@Data
public class CreateUserRequest {
    @NotBlank(message = "用户名不能为空")
    @Size(min = 4, max = 20)
    private String username;

    @NotBlank(message = "邮箱不能为空")
    @Email
    private String email;

    // ... 其他字段和验证注解
}

// repos/backend/domain/src/main/java/com/company/domain/vo/UserVO.java
@Data
public class UserVO {
    private Long id;
    private String username;
    private String email;
    // ... 其他字段
}
```

**异常处理器**
```java
// repos/backend/api/src/main/java/com/company/api/exception/GlobalExceptionHandler.java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Result<?>> handleResourceNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(Result.error(404, ex.getMessage()));
    }
}
```

**单元测试**
```java
// repos/backend/service/src/test/java/com/company/service/UserServiceTest.java
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;

    @Test
    void testCreateUser_Success() {
        // 测试代码
    }
}
```

#### 2.3 代码质量要求
- 遵循现有代码模式和项目结构
- 确保所有代码遵守编码规范
- 添加必要的日志记录
- 实现适当的错误处理
- 添加参数验证

### 步骤3：前端实现

#### 3.1 准备工作
- 导航到 `repos/frontend/`
- 阅读现有前端代码以了解项目结构
- 从 `docs/knowledges/standards/` 阅读前端编码规范

#### 3.2 生成前端代码

**类型定义**（TypeScript）
```typescript
// repos/frontend/src/types/user.ts
export interface User {
  id: number
  username: string
  email: string
  createdAt: string
}

export interface CreateUserRequest {
  username: string
  email: string
  password: string
}
```

**API客户端**
```typescript
// repos/frontend/src/api/user.ts
import request from '@/utils/request'
import type { User, CreateUserRequest } from '@/types/user'

export const userApi = {
  // 创建用户
  createUser(data: CreateUserRequest): Promise<User> {
    return request.post('/api/users', data)
  },

  // 获取用户详情
  getUser(id: number): Promise<User> {
    return request.get(`/api/users/${id}`)
  }
}
```

**Pinia Store**
```typescript
// repos/frontend/src/stores/user.ts
import { defineStore } from 'pinia'
import type { User } from '@/types/user'
import { userApi } from '@/api/user'

export const useUserStore = defineStore('user', {
  state: () => ({
    users: [] as User[],
    loading: false
  }),

  actions: {
    async createUser(data: CreateUserRequest) {
      const user = await userApi.createUser(data)
      this.users.unshift(user)
      return user
    }
  }
})
```

**Vue组件**
```vue
<!-- repos/frontend/src/components/user/UserForm.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import type { CreateUserRequest } from '@/types/user'

interface Props {
  mode: 'create' | 'edit'
  initialData?: Partial<CreateUserRequest>
}

const props = defineProps<Props>()
const emit = defineEmits<{
  submit: [data: CreateUserRequest]
  cancel: []
}>()

const formData = ref<CreateUserRequest>({
  username: '',
  email: '',
  password: ''
})

const handleSubmit = () => {
  emit('submit', formData.value)
}
</script>

<template>
  <el-form :model="formData">
    <el-form-item label="用户名">
      <el-input v-model="formData.username" />
    </el-form-item>
    <!-- ... 其他表单项 -->
    <el-form-item>
      <el-button type="primary" @click="handleSubmit">提交</el-button>
      <el-button @click="emit('cancel')">取消</el-button>
    </el-form-item>
  </el-form>
</template>
```

**页面组件**
```vue
<!-- repos/frontend/src/views/user/UserList.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

onMounted(() => {
  userStore.fetchUsers()
})
</script>

<template>
  <div class="user-list">
    <el-table :data="userStore.users" :loading="userStore.loading">
      <el-table-column prop="id" label="ID" />
      <el-table-column prop="username" label="用户名" />
      <el-table-column prop="email" label="邮箱" />
    </el-table>
  </div>
</template>
```

**路由配置**
```typescript
// repos/frontend/src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/users',
    name: 'UserList',
    component: () => import('@/views/user/UserList.vue'),
    meta: { title: '用户列表', requiresAuth: true }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

**单元测试**
```typescript
// repos/frontend/src/components/user/__tests__/UserForm.spec.ts
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import UserForm from '../UserForm.vue'

describe('UserForm', () => {
  it('renders form elements', () => {
    const wrapper = mount(UserForm, {
      props: { mode: 'create' }
    })
    expect(wrapper.find('input').exists()).toBe(true)
  })
})
```

#### 3.3 代码质量要求
- 遵循现有代码模式和项目结构
- 确保所有代码遵守编码规范
- 使用TypeScript类型定义
- 组件要有Props和Emits定义
- 样式使用scoped

### 步骤4：测试实现

#### 4.1 准备工作
- 导航到 `repos/testing/`
- 从 `tech/test_design/` 阅读测试设计

#### 4.2 生成测试代码

**集成测试**（API测试）
```python
# repos/testing/integration/test_user_api.py
import pytest
import requests

BASE_URL = "http://localhost:8080"

def test_create_user_success():
    """测试创建用户成功"""
    response = requests.post(f"{BASE_URL}/api/users", json={
        "username": "testuser",
        "email": "test@example.com",
        "password": "Test1234"
    })

    assert response.status_code == 200
    data = response.json()
    assert data["code"] == 200
    assert "id" in data["data"]

def test_create_user_duplicate_username():
    """测试用户名重复"""
    response = requests.post(f"{BASE_URL}/api/users", json={
        "username": "existing",
        "email": "new@example.com",
        "password": "Test1234"
    })

    assert response.status_code == 400
```

**E2E测试**
```python
# repos/testing/e2e/test_user_flow.py
import pytest
from playwright.sync_api import Page, expect

def test_user_registration_flow(page: Page):
    """测试用户注册流程"""
    # 访问注册页面
    page.goto("http://localhost:3000/register")

    # 填写表单
    page.fill('input[name="username"]', 'testuser')
    page.fill('input[name="email"]', 'test@example.com')
    page.fill('input[name="password"]', 'Test1234')

    # 提交
    page.click('button[type="submit"]')

    # 验证成功
    expect(page.locator('.success-message')).to_be_visible()
```

**测试配置**
```python
# repos/testing/conftest.py
import pytest

@pytest.fixture(scope="session")
def base_url():
    return "http://localhost:8080"

@pytest.fixture(scope="function")
def clean_database():
    """清理测试数据"""
    # 清理逻辑
    yield
    # 清理逻辑
```

**测试数据**
```python
# repos/testing/data/user_data.py
VALID_USER = {
    "username": "testuser",
    "email": "test@example.com",
    "password": "Test1234"
}

INVALID_USER = {
    "username": "a",  # 太短
    "email": "invalid",  # 格式错误
    "password": "123"  # 太短
}
```

#### 4.3 测试要求
- 遵循现有测试模式
- 测试用例要覆盖正常、异常、边界情况
- 使用测试fixture管理测试数据
- 每个测试用例独立，不依赖其他测试

### 步骤5：部署脚本

#### 5.1 准备工作
- 导航到 `repos/devops/`
- 从 `deploy/deployment-plan.md` 阅读部署计划

#### 5.2 生成部署脚本

**数据库迁移脚本**
```bash
# repos/devops/scripts/migrate-db.sh
#!/bin/bash
echo "Running database migrations..."
flyway -configFiles=flyway.conf migrate
echo "Migration completed!"
```

**Docker配置更新**（如需要）
```dockerfile
# repos/devops/docker/backend/Dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**部署文档**
```markdown
# repos/devops/docs/deployment.md
## 部署步骤

1. 备份数据库
2. 执行数据库迁移
3. 构建Docker镜像
4. 停止旧容器
5. 启动新容器
6. 健康检查
7. 验证功能
```

### 步骤6：运行测试

#### 6.1 后端测试
```bash
cd repos/backend
mvn clean test
mvn jacoco:report  # 生成覆盖率报告
```

#### 6.2 前端测试
```bash
cd repos/frontend
npm run test:unit
npm run type-check
npm run lint
```

#### 6.3 集成测试
```bash
cd repos/testing
python -m pytest integration/ -v
```

#### 6.4 E2E测试
```bash
cd repos/testing
python -m pytest e2e/ -v --headed
```

#### 6.5 自动修复机制

如果测试失败：
1. 分析测试失败原因
2. 定位问题代码
3. 修复问题
4. 重新运行测试
5. 最多重试3次
6. 如果仍失败，请求人工介入

⚠️ **重要**：在所有测试通过之前不要继续下一步！

### 步骤7：Git提交

#### 7.1 阅读提交规范
从 `docs/knowledges/standards/git-commit-standards.md` 阅读提交规范

#### 7.2 为每个仓库创建提交

**后端提交**：
```bash
cd repos/backend
git add .
git commit -m "feat(user): [feature-003] implement user management API

- Add User entity and repository
- Add UserService with CRUD operations
- Add UserController with REST endpoints
- Add DTO and validation
- Add unit tests

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

**前端提交**：
```bash
cd repos/frontend
git add .
git commit -m "feat(user): [feature-003] implement user management UI

- Add User types and API client
- Add UserStore with Pinia
- Add UserForm and UserList components
- Add user routes
- Add component tests

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

**测试提交**：
```bash
cd repos/testing
git add .
git commit -m "test(user): [feature-003] add user management tests

- Add integration tests for user API
- Add E2E tests for user flows
- Add test fixtures and data

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

**DevOps提交**：
```bash
cd repos/devops
git add .
git commit -m "chore(deploy): [feature-003] add deployment scripts

- Add database migration scripts
- Add deployment documentation
- Update Docker configurations

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

#### 7.3 提交要求
- 保持提交原子性（一个提交一个功能）
- 提交消息清晰描述变更内容
- 遵循约定式提交规范
- 包含feature编号便于追踪
- **不要推送到远程**（等待审核）

### 步骤8：代码审核请求

#### 8.1 生成实现摘要

```markdown
# Feature 003 代码实现摘要

## 实现概述
已完成用户管理功能的代码实现，包括后端API、前端UI、测试用例和部署脚本。

## 代码变更

### 后端（repos/backend/）
新增文件：
- domain/entity/User.java
- infrastructure/repository/UserRepository.java
- service/UserService.java
- service/impl/UserServiceImpl.java
- api/controller/UserController.java
- domain/dto/CreateUserRequest.java
- domain/vo/UserVO.java
- service/test/UserServiceTest.java

数据库迁移：
- V1.0__create_users_table.sql

### 前端（repos/frontend/）
新增文件：
- src/types/user.ts
- src/api/user.ts
- src/stores/user.ts
- src/components/user/UserForm.vue
- src/views/user/UserList.vue
- src/router/user-routes.ts
- src/components/user/__tests__/UserForm.spec.ts

### 测试（repos/testing/）
新增文件：
- integration/test_user_api.py
- e2e/test_user_flow.py
- data/user_data.py

### DevOps（repos/devops/）
新增文件：
- scripts/migrate-db.sh
- docs/deployment.md

## 测试结果

✅ 后端单元测试：15个用例全部通过，覆盖率85%
✅ 前端单元测试：8个用例全部通过
✅ 集成测试：10个用例全部通过
✅ E2E测试：3个场景全部通过

## Git提交

✅ 后端：commit 1a2b3c4
✅ 前端：commit 5d6e7f8
✅ 测试：commit 9g0h1i2
✅ DevOps：commit 3j4k5l6

## 待确认事项
1. 代码风格是否符合规范？
2. 是否需要补充注释？
3. 是否有安全隐患？

---

请审核代码，确认无误后回复"批准"。

批准后，我将把代码推送到远程仓库。
```

#### 8.2 记录审核反馈

在 `docs/feature{N}-specs/review_records/code_review/review-{timestamp}.md` 中记录：
- 审核时间
- 审核人
- 审核意见
- 发现的问题
- 修改建议
- 审核结论

#### 8.3 处理审核结果

**如果需要修改**：
1. 根据反馈修改代码
2. 重新运行测试
3. 更新Git提交（使用 `git commit --amend` 或新提交）
4. 重新请求审核

**如果批准**：
1. 询问用户是否推送到远程仓库
2. 如果用户同意，执行推送：
   ```bash
   cd repos/backend && git push origin feature/feature-003
   cd repos/frontend && git push origin feature/feature-003
   cd repos/testing && git push origin feature/feature-003
   cd repos/devops && git push origin feature/feature-003
   ```
3. 通知用户实现完成

## 重要说明

### 1. 实现一致性
- 实现必须完全匹配技术设计文档
- 如果发现设计问题，记录并咨询用户
- 不要擅自偏离设计方案

### 2. 代码质量
- 所有代码必须遵循编码规范
- 添加适当的错误处理和日志
- 实现参数验证
- 仅在逻辑不明显处添加注释

### 3. 测试优先
- 在请求审核之前所有测试必须通过
- 测试覆盖率要达标（后端≥80%，前端≥70%）
- 测试用例要覆盖正常、异常、边界场景

### 4. Git规范
- 提交消息遵循约定式提交规范
- 保持提交原子性
- 未经明确用户批准，不得推送到远程

### 5. 安全考虑
- 敏感信息不要硬编码
- 密码要加密存储
- API要有权限验证
- 防止SQL注入、XSS等安全漏洞

### 6. 性能考虑
- 数据库查询要有索引支持
- 避免N+1查询问题
- 前端组件要有loading状态
- 大列表要分页

## 输出示例

```
✅ 代码实现已完成！

📁 Feature编号：feature003

📊 代码统计：
- 后端：新增8个类，800行代码
- 前端：新增5个组件，600行代码
- 测试：新增18个测试用例
- 文档：新增3个部署文档

✅ 测试结果：
- 后端单元测试：15个用例全部通过 ✓
- 前端单元测试：8个用例全部通过 ✓
- 集成测试：10个用例全部通过 ✓
- E2E测试：3个场景全部通过 ✓
- 代码覆盖率：后端85%，前端72%

📝 Git提交：
- ✅ 后端：feat(user): [feature-003] implement user management API
- ✅ 前端：feat(user): [feature-003] implement user management UI
- ✅ 测试：test(user): [feature-003] add user management tests
- ✅ DevOps：chore(deploy): [feature-003] add deployment scripts

---

代码已准备就绪，请审核！

审核通过后，我将推送到远程仓库。
```

## 常见问题处理

### Q1: 测试失败怎么办？

**A**: 自动修复机制：
1. 分析失败原因
2. 修复代码
3. 重新测试
4. 最多重试3次
5. 失败后请求人工介入

### Q2: 发现设计问题怎么办？

**A**:
1. 停止实现
2. 记录问题
3. 咨询用户
4. 等待用户决策（修改设计 or 继续实现）

### Q3: 代码风格不一致怎么办？

**A**:
1. 运行代码格式化工具
2. 运行lint检查
3. 修复所有警告
4. 确保通过CI检查

### Q4: 性能不达标怎么办？

**A**:
1. 进行性能分析
2. 识别瓶颈
3. 优化代码（添加索引、缓存等）
4. 重新测试
5. 如果无法优化，咨询用户
