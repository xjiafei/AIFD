# 测试最佳实践

> 本文档总结了软件测试的行业最佳实践，涵盖单元测试、集成测试、E2E测试等全方位测试策略。

## 📋 目录

- [测试金字塔](#测试金字塔)
- [单元测试](#单元测试)
- [集成测试](#集成测试)
- [E2E测试](#e2e测试)
- [Mock和Stub](#mock和stub)
- [测试覆盖率](#测试覆盖率)
- [TDD实践](#tdd实践)
- [测试数据管理](#测试数据管理)
- [检查清单](#检查清单)

---

## 测试金字塔

### 测试金字塔模型

```
        /\
       /  \
      / E2E \       (少量：10-20%)
     /--------\
    /          \
   / Integration \  (中量：20-30%)
  /--------------\
 /                \
/ Unit Tests       \ (大量：50-70%)
--------------------
```

**分层说明**：

| 测试类型 | 占比 | 速度 | 成本 | 覆盖范围 | 稳定性 |
|---------|------|------|------|---------|--------|
| 单元测试 | 50-70% | 快 | 低 | 单个函数/方法 | 高 |
| 集成测试 | 20-30% | 中 | 中 | 多个模块/组件 | 中 |
| E2E测试 | 10-20% | 慢 | 高 | 完整业务流程 | 低 |

### ✅ 好的实践

```yaml
测试策略:
  单元测试:
    - 覆盖所有业务逻辑
    - 覆盖边界条件和异常情况
    - 快速执行（每个测试<100ms）

  集成测试:
    - 覆盖关键API接口
    - 测试数据库交互
    - 测试外部服务集成

  E2E测试:
    - 覆盖核心业务流程
    - 模拟真实用户操作
    - 定期执行（不是每次提交）
```

---

## 单元测试

### 1. JUnit 5（Java后端）

### ✅ 好的实践

```java
// UserServiceTest.java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @InjectMocks
    private UserServiceImpl userService;

    /**
     * 测试创建用户 - 成功场景
     */
    @Test
    @DisplayName("创建用户 - 成功")
    void testCreateUser_Success() {
        // Given（准备测试数据）
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername("testuser");
        request.setEmail("test@example.com");
        request.setPassword("password123");

        User savedUser = new User();
        savedUser.setId(1L);
        savedUser.setUsername("testuser");
        savedUser.setEmail("test@example.com");

        when(userRepository.existsByUsername("testuser")).thenReturn(false);
        when(userRepository.existsByEmail("test@example.com")).thenReturn(false);
        when(passwordEncoder.encode("password123")).thenReturn("encoded_password");
        when(userRepository.save(any(User.class))).thenReturn(savedUser);

        // When（执行测试方法）
        UserVO result = userService.createUser(request);

        // Then（验证结果）
        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("testuser", result.getUsername());
        assertEquals("test@example.com", result.getEmail());

        // 验证方法调用
        verify(userRepository).existsByUsername("testuser");
        verify(userRepository).existsByEmail("test@example.com");
        verify(passwordEncoder).encode("password123");
        verify(userRepository).save(any(User.class));
    }

    /**
     * 测试创建用户 - 用户名已存在
     */
    @Test
    @DisplayName("创建用户 - 用户名已存在")
    void testCreateUser_UsernameExists() {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername("existinguser");
        request.setEmail("test@example.com");
        request.setPassword("password123");

        when(userRepository.existsByUsername("existinguser")).thenReturn(true);

        // When & Then
        BusinessException exception = assertThrows(
            BusinessException.class,
            () -> userService.createUser(request)
        );

        assertEquals("用户名已存在", exception.getMessage());
        verify(userRepository).existsByUsername("existinguser");
        verify(userRepository, never()).save(any(User.class));
    }

    /**
     * 测试获取用户 - 不存在
     */
    @Test
    @DisplayName("获取用户 - 不存在")
    void testGetUser_NotFound() {
        // Given
        Long userId = 999L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());

        // When & Then
        assertThrows(
            ResourceNotFoundException.class,
            () -> userService.getUser(userId)
        );

        verify(userRepository).findById(userId);
    }

    /**
     * 参数化测试 - 测试多组数据
     */
    @ParameterizedTest
    @DisplayName("验证邮箱格式")
    @CsvSource({
        "test@example.com, true",
        "invalid-email, false",
        "@example.com, false",
        "test@, false"
    })
    void testValidateEmail(String email, boolean expected) {
        boolean result = userService.isValidEmail(email);
        assertEquals(expected, result);
    }

    /**
     * 测试边界条件
     */
    @Nested
    @DisplayName("边界条件测试")
    class BoundaryTests {

        @Test
        @DisplayName("用户名长度 - 最小值")
        void testUsername_MinLength() {
            CreateUserRequest request = new CreateUserRequest();
            request.setUsername("ab");  // 长度为2，不满足最小3的要求

            assertThrows(ValidationException.class, () -> userService.createUser(request));
        }

        @Test
        @DisplayName("用户名长度 - 最大值")
        void testUsername_MaxLength() {
            CreateUserRequest request = new CreateUserRequest();
            request.setUsername("a".repeat(21));  // 长度为21，超过最大20的要求

            assertThrows(ValidationException.class, () -> userService.createUser(request));
        }

        @Test
        @DisplayName("用户名长度 - 边界值")
        void testUsername_BoundaryValues() {
            // 测试最小合法值
            CreateUserRequest request1 = new CreateUserRequest();
            request1.setUsername("abc");  // 长度为3
            assertDoesNotThrow(() -> validateUsername(request1.getUsername()));

            // 测试最大合法值
            CreateUserRequest request2 = new CreateUserRequest();
            request2.setUsername("a".repeat(20));  // 长度为20
            assertDoesNotThrow(() -> validateUsername(request2.getUsername()));
        }
    }
}
```

**单元测试命名规范**：
- 方法名：`test{方法名}_{场景}_{预期结果}`
- 示例：`testCreateUser_UsernameExists_ThrowsException`

**单元测试原则**：
- **F.I.R.S.T原则**：
  - **F**ast：快速执行
  - **I**ndependent：独立运行
  - **R**epeatable：可重复执行
  - **S**elf-Validating：自动验证
  - **T**imely：及时编写

### 2. Vitest（前端）

### ✅ 好的实践

```typescript
// userService.spec.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { userApi } from '@/api/user'
import { userService } from '@/services/userService'
import type { User } from '@/types/user'

// Mock API
vi.mock('@/api/user')

describe('UserService', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  describe('getUser', () => {
    it('should return user when API call succeeds', async () => {
      // Given
      const mockUser: User = {
        id: 1,
        username: 'testuser',
        email: 'test@example.com'
      }

      vi.mocked(userApi.getUser).mockResolvedValue(mockUser)

      // When
      const result = await userService.getUser(1)

      // Then
      expect(result).toEqual(mockUser)
      expect(userApi.getUser).toHaveBeenCalledWith(1)
      expect(userApi.getUser).toHaveBeenCalledTimes(1)
    })

    it('should throw error when user not found', async () => {
      // Given
      vi.mocked(userApi.getUser).mockRejectedValue(
        new Error('User not found')
      )

      // When & Then
      await expect(userService.getUser(999)).rejects.toThrow('User not found')
      expect(userApi.getUser).toHaveBeenCalledWith(999)
    })
  })

  describe('createUser', () => {
    it('should create user successfully', async () => {
      // Given
      const newUser = {
        username: 'newuser',
        email: 'new@example.com',
        password: 'password123'
      }

      const createdUser: User = {
        id: 1,
        ...newUser
      }

      vi.mocked(userApi.createUser).mockResolvedValue(createdUser)

      // When
      const result = await userService.createUser(newUser)

      // Then
      expect(result).toEqual(createdUser)
      expect(userApi.createUser).toHaveBeenCalledWith(newUser)
    })

    it('should handle validation errors', async () => {
      // Given
      const invalidUser = {
        username: 'ab',  // 太短
        email: 'invalid-email',
        password: '123'  // 太短
      }

      vi.mocked(userApi.createUser).mockRejectedValue(
        new Error('Validation failed')
      )

      // When & Then
      await expect(userService.createUser(invalidUser))
        .rejects.toThrow('Validation failed')
    })
  })

  describe('validateEmail', () => {
    it.each([
      ['test@example.com', true],
      ['invalid-email', false],
      ['@example.com', false],
      ['test@', false]
    ])('should validate email: %s -> %s', (email, expected) => {
      const result = userService.validateEmail(email)
      expect(result).toBe(expected)
    })
  })
})
```

### 3. Vue组件测试

### ✅ 好的实践

```typescript
// UserLogin.spec.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import { createPinia } from 'pinia'
import UserLogin from '@/views/auth/UserLogin.vue'
import { authApi } from '@/api/auth'

// Mock API
vi.mock('@/api/auth')

describe('UserLogin', () => {
  it('should render login form', () => {
    // Given
    const wrapper = mount(UserLogin, {
      global: {
        plugins: [createPinia()]
      }
    })

    // Then
    expect(wrapper.find('input[name="username"]').exists()).toBe(true)
    expect(wrapper.find('input[name="password"]').exists()).toBe(true)
    expect(wrapper.find('button[type="submit"]').exists()).toBe(true)
  })

  it('should login successfully', async () => {
    // Given
    const wrapper = mount(UserLogin, {
      global: {
        plugins: [createPinia()]
      }
    })

    vi.mocked(authApi.login).mockResolvedValue({
      token: 'fake-token',
      expiresIn: 86400
    })

    // When
    await wrapper.find('input[name="username"]').setValue('testuser')
    await wrapper.find('input[name="password"]').setValue('password123')
    await wrapper.find('form').trigger('submit')
    await wrapper.vm.$nextTick()

    // Then
    expect(authApi.login).toHaveBeenCalledWith({
      username: 'testuser',
      password: 'password123'
    })
  })

  it('should show error message on login failure', async () => {
    // Given
    const wrapper = mount(UserLogin, {
      global: {
        plugins: [createPinia()]
      }
    })

    vi.mocked(authApi.login).mockRejectedValue(
      new Error('Invalid credentials')
    )

    // When
    await wrapper.find('input[name="username"]').setValue('wronguser')
    await wrapper.find('input[name="password"]').setValue('wrongpass')
    await wrapper.find('form').trigger('submit')
    await wrapper.vm.$nextTick()

    // Then
    expect(wrapper.find('.error-message').text()).toContain('Invalid credentials')
  })

  it('should disable submit button when loading', async () => {
    // Given
    const wrapper = mount(UserLogin, {
      global: {
        plugins: [createPinia()]
      }
    })

    vi.mocked(authApi.login).mockImplementation(
      () => new Promise(resolve => setTimeout(resolve, 1000))
    )

    // When
    await wrapper.find('form').trigger('submit')
    await wrapper.vm.$nextTick()

    // Then
    expect(wrapper.find('button[type="submit"]').attributes('disabled')).toBeDefined()
  })
})
```

---

## 集成测试

### 1. API集成测试（Python + pytest）

### ✅ 好的实践

```python
# test_user_api.py
import pytest
import requests
from typing import Dict, Any

# 测试配置
BASE_URL = "http://localhost:8080"
API_PREFIX = "/api"

@pytest.fixture(scope="module")
def api_client():
    """API客户端fixture"""
    class APIClient:
        def __init__(self, base_url: str):
            self.base_url = base_url
            self.token = None

        def login(self, username: str, password: str):
            """登录并获取token"""
            response = requests.post(
                f"{self.base_url}{API_PREFIX}/auth/login",
                json={"username": username, "password": password}
            )
            response.raise_for_status()
            self.token = response.json()["data"]["token"]
            return self.token

        def get_headers(self) -> Dict[str, str]:
            """获取请求头（包含token）"""
            headers = {"Content-Type": "application/json"}
            if self.token:
                headers["Authorization"] = f"Bearer {self.token}"
            return headers

        def get(self, path: str, **kwargs):
            """GET请求"""
            return requests.get(
                f"{self.base_url}{API_PREFIX}{path}",
                headers=self.get_headers(),
                **kwargs
            )

        def post(self, path: str, **kwargs):
            """POST请求"""
            return requests.post(
                f"{self.base_url}{API_PREFIX}{path}",
                headers=self.get_headers(),
                **kwargs
            )

        def put(self, path: str, **kwargs):
            """PUT请求"""
            return requests.put(
                f"{self.base_url}{API_PREFIX}{path}",
                headers=self.get_headers(),
                **kwargs
            )

        def delete(self, path: str, **kwargs):
            """DELETE请求"""
            return requests.delete(
                f"{self.base_url}{API_PREFIX}{path}",
                headers=self.get_headers(),
                **kwargs
            )

    client = APIClient(BASE_URL)
    yield client

@pytest.fixture(scope="function")
def test_user(api_client):
    """创建测试用户（每个测试用例独立）"""
    # Setup：创建测试用户
    user_data = {
        "username": f"testuser_{pytest.timestamp()}",
        "email": f"test_{pytest.timestamp()}@example.com",
        "password": "TestPassword123!"
    }

    response = api_client.post("/users", json=user_data)
    assert response.status_code == 201
    user = response.json()["data"]

    yield user

    # Teardown：删除测试用户
    api_client.delete(f"/users/{user['id']}")

class TestUserAPI:
    """用户API测试"""

    def test_create_user_success(self, api_client):
        """测试创建用户 - 成功"""
        # Given
        user_data = {
            "username": f"newuser_{pytest.timestamp()}",
            "email": f"new_{pytest.timestamp()}@example.com",
            "password": "Password123!"
        }

        # When
        response = api_client.post("/users", json=user_data)

        # Then
        assert response.status_code == 201
        data = response.json()
        assert data["code"] == 200
        assert data["data"]["username"] == user_data["username"]
        assert data["data"]["email"] == user_data["email"]
        assert "password" not in data["data"]  # 不应该返回密码

        # Cleanup
        api_client.delete(f"/users/{data['data']['id']}")

    def test_create_user_duplicate_username(self, api_client, test_user):
        """测试创建用户 - 用户名重复"""
        # Given
        user_data = {
            "username": test_user["username"],  # 重复的用户名
            "email": "another@example.com",
            "password": "Password123!"
        }

        # When
        response = api_client.post("/users", json=user_data)

        # Then
        assert response.status_code == 400
        data = response.json()
        assert data["code"] == 40001
        assert "用户名已存在" in data["message"]

    def test_create_user_invalid_email(self, api_client):
        """测试创建用户 - 邮箱格式错误"""
        # Given
        user_data = {
            "username": "testuser",
            "email": "invalid-email",  # 错误的邮箱格式
            "password": "Password123!"
        }

        # When
        response = api_client.post("/users", json=user_data)

        # Then
        assert response.status_code == 400
        data = response.json()
        assert "邮箱格式不正确" in data["message"]

    def test_get_user_success(self, api_client, test_user):
        """测试获取用户 - 成功"""
        # When
        response = api_client.get(f"/users/{test_user['id']}")

        # Then
        assert response.status_code == 200
        data = response.json()
        assert data["data"]["id"] == test_user["id"]
        assert data["data"]["username"] == test_user["username"]

    def test_get_user_not_found(self, api_client):
        """测试获取用户 - 不存在"""
        # When
        response = api_client.get("/users/999999")

        # Then
        assert response.status_code == 404
        data = response.json()
        assert data["code"] == 404
        assert "用户不存在" in data["message"]

    def test_update_user_success(self, api_client, test_user):
        """测试更新用户 - 成功"""
        # Given
        api_client.login(test_user["username"], "TestPassword123!")
        update_data = {
            "email": "updated@example.com"
        }

        # When
        response = api_client.put(f"/users/{test_user['id']}", json=update_data)

        # Then
        assert response.status_code == 200
        data = response.json()
        assert data["data"]["email"] == "updated@example.com"

    def test_delete_user_success(self, api_client, test_user):
        """测试删除用户 - 成功"""
        # Given
        api_client.login(test_user["username"], "TestPassword123!")

        # When
        response = api_client.delete(f"/users/{test_user['id']}")

        # Then
        assert response.status_code == 204

        # 验证用户已删除
        response = api_client.get(f"/users/{test_user['id']}")
        assert response.status_code == 404

    @pytest.mark.parametrize("page,size,expected_count", [
        (0, 10, 10),
        (0, 20, 20),
        (1, 10, 10)
    ])
    def test_list_users_pagination(self, api_client, page, size, expected_count):
        """测试用户列表 - 分页"""
        # When
        response = api_client.get("/users", params={"page": page, "size": size})

        # Then
        assert response.status_code == 200
        data = response.json()
        assert len(data["data"]["items"]) <= expected_count
        assert data["data"]["pagination"]["page"] == page
        assert data["data"]["pagination"]["size"] == size
```

### 2. 数据库集成测试

### ✅ 好的实践

```java
// UserRepositoryTest.java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(locations = "classpath:application-test.properties")
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @BeforeEach
    void setUp() {
        // 清理数据
        userRepository.deleteAll();
    }

    @Test
    @DisplayName("保存用户 - 成功")
    void testSave_Success() {
        // Given
        User user = new User();
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        user.setPassword("encoded_password");

        // When
        User savedUser = userRepository.save(user);
        entityManager.flush();

        // Then
        assertNotNull(savedUser.getId());
        assertEquals("testuser", savedUser.getUsername());
        assertEquals("test@example.com", savedUser.getEmail());
        assertNotNull(savedUser.getCreatedAt());
    }

    @Test
    @DisplayName("根据用户名查询 - 存在")
    void testFindByUsername_Exists() {
        // Given
        User user = new User();
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        user.setPassword("encoded_password");
        entityManager.persist(user);
        entityManager.flush();

        // When
        Optional<User> result = userRepository.findByUsername("testuser");

        // Then
        assertTrue(result.isPresent());
        assertEquals("testuser", result.get().getUsername());
    }

    @Test
    @DisplayName("根据用户名查询 - 不存在")
    void testFindByUsername_NotExists() {
        // When
        Optional<User> result = userRepository.findByUsername("nonexistent");

        // Then
        assertFalse(result.isPresent());
    }

    @Test
    @DisplayName("检查用户名是否存在")
    void testExistsByUsername() {
        // Given
        User user = new User();
        user.setUsername("existinguser");
        user.setEmail("existing@example.com");
        user.setPassword("encoded_password");
        entityManager.persist(user);
        entityManager.flush();

        // When & Then
        assertTrue(userRepository.existsByUsername("existinguser"));
        assertFalse(userRepository.existsByUsername("nonexistent"));
    }

    @Test
    @DisplayName("分页查询")
    void testFindAll_Pagination() {
        // Given：创建20个用户
        for (int i = 0; i < 20; i++) {
            User user = new User();
            user.setUsername("user" + i);
            user.setEmail("user" + i + "@example.com");
            user.setPassword("password");
            entityManager.persist(user);
        }
        entityManager.flush();

        // When
        Page<User> page1 = userRepository.findAll(PageRequest.of(0, 10));
        Page<User> page2 = userRepository.findAll(PageRequest.of(1, 10));

        // Then
        assertEquals(20, page1.getTotalElements());
        assertEquals(2, page1.getTotalPages());
        assertEquals(10, page1.getContent().size());
        assertEquals(10, page2.getContent().size());
    }
}
```

---

## E2E测试

### 1. Playwright测试

### ✅ 好的实践

```python
# test_user_flow.py
import pytest
from playwright.sync_api import Page, expect

@pytest.fixture(scope="session")
def browser_context_args(browser_context_args):
    """配置浏览器上下文"""
    return {
        **browser_context_args,
        "viewport": {"width": 1920, "height": 1080},
        "locale": "zh-CN"
    }

@pytest.fixture(scope="function")
def page(context):
    """每个测试用例创建新页面"""
    page = context.new_page()
    yield page
    page.close()

class TestUserLoginFlow:
    """用户登录流程测试"""

    def test_login_success(self, page: Page):
        """测试登录 - 成功"""
        # Given：访问登录页
        page.goto("http://localhost:3000/login")

        # When：输入用户名和密码
        page.fill('input[name="username"]', "testuser")
        page.fill('input[name="password"]', "password123")

        # 点击登录按钮
        page.click('button[type="submit"]')

        # Then：验证跳转到首页
        page.wait_for_url("http://localhost:3000/dashboard")
        expect(page).to_have_url("http://localhost:3000/dashboard")

        # 验证用户名显示
        expect(page.locator(".user-info")).to_contain_text("testuser")

    def test_login_invalid_credentials(self, page: Page):
        """测试登录 - 错误的用户名或密码"""
        # Given
        page.goto("http://localhost:3000/login")

        # When
        page.fill('input[name="username"]', "wronguser")
        page.fill('input[name="password"]', "wrongpass")
        page.click('button[type="submit"]')

        # Then：验证错误提示
        expect(page.locator(".error-message")).to_be_visible()
        expect(page.locator(".error-message")).to_contain_text("用户名或密码错误")

    def test_login_validation(self, page: Page):
        """测试登录 - 表单验证"""
        # Given
        page.goto("http://localhost:3000/login")

        # When：不输入任何内容，直接点击登录
        page.click('button[type="submit"]')

        # Then：验证验证提示
        expect(page.locator('input[name="username"]:invalid')).to_be_visible()
        expect(page.locator('input[name="password"]:invalid')).to_be_visible()

class TestUserRegistrationFlow:
    """用户注册流程测试"""

    def test_registration_success(self, page: Page):
        """测试注册 - 成功"""
        # Given
        page.goto("http://localhost:3000/register")

        # When
        timestamp = int(pytest.timestamp())
        page.fill('input[name="username"]', f"newuser{timestamp}")
        page.fill('input[name="email"]', f"newuser{timestamp}@example.com")
        page.fill('input[name="password"]', "Password123!")
        page.fill('input[name="confirmPassword"]', "Password123!")
        page.click('button[type="submit"]')

        # Then：验证跳转到登录页
        page.wait_for_url("http://localhost:3000/login")
        expect(page.locator(".success-message")).to_contain_text("注册成功")

    def test_registration_password_mismatch(self, page: Page):
        """测试注册 - 两次密码不一致"""
        # Given
        page.goto("http://localhost:3000/register")

        # When
        page.fill('input[name="username"]', "testuser")
        page.fill('input[name="email"]', "test@example.com")
        page.fill('input[name="password"]', "Password123!")
        page.fill('input[name="confirmPassword"]', "DifferentPass!")
        page.click('button[type="submit"]')

        # Then
        expect(page.locator(".error-message")).to_contain_text("两次密码不一致")

class TestUserManagementFlow:
    """用户管理流程测试"""

    @pytest.fixture(autouse=True)
    def login(self, page: Page):
        """自动登录（每个测试前执行）"""
        page.goto("http://localhost:3000/login")
        page.fill('input[name="username"]', "admin")
        page.fill('input[name="password"]', "admin123")
        page.click('button[type="submit"]')
        page.wait_for_url("http://localhost:3000/dashboard")

    def test_view_user_list(self, page: Page):
        """测试查看用户列表"""
        # When
        page.goto("http://localhost:3000/users")

        # Then：验证用户列表显示
        expect(page.locator("table")).to_be_visible()
        expect(page.locator("table tbody tr")).to_have_count_greater_than(0)

    def test_create_user(self, page: Page):
        """测试创建用户"""
        # Given
        page.goto("http://localhost:3000/users")

        # When：点击"新建用户"按钮
        page.click('button:has-text("新建用户")')

        # 填写表单
        timestamp = int(pytest.timestamp())
        page.fill('input[name="username"]', f"user{timestamp}")
        page.fill('input[name="email"]', f"user{timestamp}@example.com")
        page.fill('input[name="password"]', "Password123!")
        page.click('button:has-text("确定")')

        # Then：验证成功提示
        expect(page.locator(".success-message")).to_contain_text("创建成功")

        # 验证用户出现在列表中
        expect(page.locator("table")).to_contain_text(f"user{timestamp}")

    def test_edit_user(self, page: Page):
        """测试编辑用户"""
        # Given
        page.goto("http://localhost:3000/users")

        # When：点击第一行的"编辑"按钮
        page.locator("table tbody tr").first.locator('button:has-text("编辑")').click()

        # 修改邮箱
        page.fill('input[name="email"]', "updated@example.com")
        page.click('button:has-text("确定")')

        # Then：验证成功提示
        expect(page.locator(".success-message")).to_contain_text("更新成功")

    def test_delete_user(self, page: Page):
        """测试删除用户"""
        # Given
        page.goto("http://localhost:3000/users")
        initial_count = page.locator("table tbody tr").count()

        # When：点击第一行的"删除"按钮
        page.locator("table tbody tr").first.locator('button:has-text("删除")').click()

        # 确认删除
        page.click('button:has-text("确认")')

        # Then：验证成功提示
        expect(page.locator(".success-message")).to_contain_text("删除成功")

        # 验证用户数量减少
        expect(page.locator("table tbody tr")).to_have_count(initial_count - 1)

    def test_search_user(self, page: Page):
        """测试搜索用户"""
        # Given
        page.goto("http://localhost:3000/users")

        # When：输入搜索关键词
        page.fill('input[placeholder="搜索用户"]', "admin")
        page.press('input[placeholder="搜索用户"]', "Enter")

        # Then：验证搜索结果
        page.wait_for_timeout(500)  # 等待搜索完成
        expect(page.locator("table tbody tr")).to_have_count_greater_than(0)
        expect(page.locator("table")).to_contain_text("admin")
```

### 2. E2E测试最佳实践

### ✅ 好的实践

```python
# conftest.py
import pytest
import time

@pytest.fixture(scope="session")
def timestamp():
    """生成唯一时间戳"""
    return int(time.time() * 1000)

# 使用Page Object模式
# pages/login_page.py
from playwright.sync_api import Page

class LoginPage:
    def __init__(self, page: Page):
        self.page = page
        self.username_input = page.locator('input[name="username"]')
        self.password_input = page.locator('input[name="password"]')
        self.submit_button = page.locator('button[type="submit"]')
        self.error_message = page.locator('.error-message')

    def navigate(self):
        """访问登录页"""
        self.page.goto("http://localhost:3000/login")

    def login(self, username: str, password: str):
        """执行登录"""
        self.username_input.fill(username)
        self.password_input.fill(password)
        self.submit_button.click()

    def get_error_message(self) -> str:
        """获取错误提示"""
        return self.error_message.text_content()

# 使用Page Object
def test_login_with_page_object(page: Page):
    login_page = LoginPage(page)

    login_page.navigate()
    login_page.login("testuser", "password123")

    expect(page).to_have_url("http://localhost:3000/dashboard")
```

---

## Mock和Stub

### 1. Mockito（Java）

### ✅ 好的实践

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private ProductService productService;

    @Mock
    private UserService userService;

    @InjectMocks
    private OrderServiceImpl orderService;

    @Test
    @DisplayName("创建订单 - Mock多个依赖")
    void testCreateOrder_MockMultipleDependencies() {
        // Given
        Long userId = 1L;
        Long productId = 100L;

        User user = new User();
        user.setId(userId);
        user.setUsername("testuser");

        Product product = new Product();
        product.setId(productId);
        product.setPrice(new BigDecimal("99.99"));
        product.setStock(10);

        when(userService.getUser(userId)).thenReturn(user);
        when(productService.getProduct(productId)).thenReturn(product);
        when(orderRepository.save(any(Order.class))).thenAnswer(
            invocation -> {
                Order order = invocation.getArgument(0);
                order.setId(1L);
                return order;
            }
        );

        CreateOrderRequest request = new CreateOrderRequest();
        request.setUserId(userId);
        request.setProductId(productId);
        request.setQuantity(2);

        // When
        OrderVO result = orderService.createOrder(request);

        // Then
        assertNotNull(result);
        assertEquals(1L, result.getId());

        // 验证方法调用顺序
        InOrder inOrder = inOrder(userService, productService, orderRepository);
        inOrder.verify(userService).getUser(userId);
        inOrder.verify(productService).getProduct(productId);
        inOrder.verify(orderRepository).save(any(Order.class));
    }

    @Test
    @DisplayName("使用ArgumentCaptor捕获参数")
    void testCreateOrder_ArgumentCaptor() {
        // Given
        ArgumentCaptor<Order> orderCaptor = ArgumentCaptor.forClass(Order.class);

        when(userService.getUser(anyLong())).thenReturn(new User());
        when(productService.getProduct(anyLong())).thenReturn(new Product());
        when(orderRepository.save(any(Order.class))).thenReturn(new Order());

        CreateOrderRequest request = new CreateOrderRequest();
        request.setUserId(1L);
        request.setProductId(100L);
        request.setQuantity(2);

        // When
        orderService.createOrder(request);

        // Then：捕获并验证传入save方法的Order对象
        verify(orderRepository).save(orderCaptor.capture());
        Order capturedOrder = orderCaptor.getValue();

        assertEquals(1L, capturedOrder.getUserId());
        assertEquals(2, capturedOrder.getQuantity());
    }

    @Test
    @DisplayName("Mock void方法抛出异常")
    void testDeleteOrder_MockVoidMethod() {
        // Given
        Long orderId = 1L;

        doThrow(new RuntimeException("删除失败"))
            .when(orderRepository).deleteById(orderId);

        // When & Then
        assertThrows(RuntimeException.class, () -> {
            orderService.deleteOrder(orderId);
        });
    }
}
```

---

## 测试覆盖率

### 1. JaCoCo（Java）

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>PACKAGE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

```bash
# 运行测试并生成覆盖率报告
mvn clean test jacoco:report

# 查看报告
open target/site/jacoco/index.html
```

### 2. Vitest Coverage（前端）

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.spec.ts',
        '**/*.config.ts'
      ],
      lines: 80,
      functions: 80,
      branches: 80,
      statements: 80
    }
  }
})
```

```bash
# 运行测试并生成覆盖率报告
npm run test:coverage

# 查看报告
open coverage/index.html
```

---

## TDD实践

### Test-Driven Development（测试驱动开发）

**TDD流程**：

```
1. 编写失败的测试（Red）
   ↓
2. 编写最小代码使测试通过（Green）
   ↓
3. 重构代码（Refactor）
   ↓
回到步骤1
```

### ✅ 好的实践

```java
// 步骤1：编写失败的测试
@Test
void testCalculateDiscount_VipUser_ShouldGet20PercentOff() {
    // Given
    User vipUser = new User();
    vipUser.setMemberLevel("VIP");
    BigDecimal originalPrice = new BigDecimal("100.00");

    // When
    BigDecimal discountedPrice = orderService.calculateDiscount(vipUser, originalPrice);

    // Then
    assertEquals(new BigDecimal("80.00"), discountedPrice);
}

// 步骤2：编写最小代码使测试通过
public BigDecimal calculateDiscount(User user, BigDecimal originalPrice) {
    if ("VIP".equals(user.getMemberLevel())) {
        return originalPrice.multiply(new BigDecimal("0.8"));
    }
    return originalPrice;
}

// 步骤3：重构（添加更多测试，优化代码）
@ParameterizedTest
@CsvSource({
    "VIP, 100.00, 80.00",
    "REGULAR, 100.00, 95.00",
    "GUEST, 100.00, 100.00"
})
void testCalculateDiscount_DifferentMemberLevels(
    String memberLevel,
    BigDecimal originalPrice,
    BigDecimal expected
) {
    User user = new User();
    user.setMemberLevel(memberLevel);

    BigDecimal result = orderService.calculateDiscount(user, originalPrice);

    assertEquals(expected, result);
}

// 重构后的代码
public BigDecimal calculateDiscount(User user, BigDecimal originalPrice) {
    BigDecimal discountRate = switch (user.getMemberLevel()) {
        case "VIP" -> new BigDecimal("0.8");
        case "REGULAR" -> new BigDecimal("0.95");
        default -> BigDecimal.ONE;
    };
    return originalPrice.multiply(discountRate);
}
```

---

## 测试数据管理

### 1. 使用Fixture和Factory

```python
# conftest.py
import pytest
from faker import Faker

fake = Faker('zh_CN')

@pytest.fixture
def user_factory():
    """用户工厂"""
    def create_user(**kwargs):
        default_data = {
            "username": fake.user_name(),
            "email": fake.email(),
            "password": fake.password(),
            "first_name": fake.first_name(),
            "last_name": fake.last_name()
        }
        default_data.update(kwargs)
        return default_data
    return create_user

@pytest.fixture
def product_factory():
    """产品工厂"""
    def create_product(**kwargs):
        default_data = {
            "name": fake.word(),
            "price": fake.pydecimal(left_digits=3, right_digits=2, positive=True),
            "stock": fake.random_int(min=0, max=100)
        }
        default_data.update(kwargs)
        return default_data
    return create_product

# 使用Factory
def test_create_user(user_factory):
    user = user_factory(username="testuser")
    assert user["username"] == "testuser"
```

### 2. 数据库Seed

```java
// 测试数据初始化
@Component
public class TestDataSeeder {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private ProductRepository productRepository;

    public void seedUsers() {
        List<User> users = Arrays.asList(
            createUser("admin", "admin@example.com", "ADMIN"),
            createUser("user1", "user1@example.com", "USER"),
            createUser("user2", "user2@example.com", "USER")
        );
        userRepository.saveAll(users);
    }

    public void seedProducts() {
        List<Product> products = Arrays.asList(
            createProduct("Product 1", new BigDecimal("99.99")),
            createProduct("Product 2", new BigDecimal("199.99")),
            createProduct("Product 3", new BigDecimal("299.99"))
        );
        productRepository.saveAll(products);
    }

    private User createUser(String username, String email, String role) {
        User user = new User();
        user.setUsername(username);
        user.setEmail(email);
        user.setRole(role);
        return user;
    }

    private Product createProduct(String name, BigDecimal price) {
        Product product = new Product();
        product.setName(name);
        product.setPrice(price);
        product.setStock(100);
        return product;
    }
}
```

---

## 检查清单

```yaml
单元测试:
  - [ ] 覆盖所有业务逻辑
  - [ ] 测试正常流程、异常流程、边界条件
  - [ ] 测试命名清晰（test{方法名}_{场景}_{结果}）
  - [ ] 每个测试独立运行
  - [ ] 快速执行（<100ms）
  - [ ] 使用Mock隔离依赖
  - [ ] 覆盖率 >= 80%

集成测试:
  - [ ] 覆盖所有API接口
  - [ ] 测试数据库交互
  - [ ] 测试请求参数验证
  - [ ] 测试错误处理
  - [ ] 测试分页和过滤
  - [ ] 每个测试独立数据
  - [ ] 测试后清理数据

E2E测试:
  - [ ] 覆盖核心业务流程
  - [ ] 使用Page Object模式
  - [ ] 模拟真实用户操作
  - [ ] 失败时自动截图
  - [ ] 稳定性高（减少flaky tests）

通用:
  - [ ] 遵循AAA模式（Arrange-Act-Assert）
  - [ ] 一个测试只验证一个功能点
  - [ ] 测试代码和业务代码同步更新
  - [ ] 定期review测试代码质量
  - [ ] CI/CD集成自动化测试
```

---

## 总结

1. **测试金字塔**：单元测试为主，集成测试为辅，E2E测试覆盖核心流程
2. **测试原则**：F.I.R.S.T（Fast, Independent, Repeatable, Self-Validating, Timely）
3. **高质量测试**：清晰的命名、完善的断言、充分的覆盖
4. **Mock和Stub**：隔离依赖，提高测试速度和稳定性
5. **TDD**：测试驱动开发，先写测试再写代码
6. **数据管理**：使用Fixture和Factory，保持测试数据独立
7. **持续集成**：自动化测试集成到CI/CD流程
