# Python 编码规范

> 本规范适用于测试代码（pytest）和后端脚本开发

## 基本规范

### 1. 代码风格

遵循 **PEP 8** 规范，使用工具自动检查：
- **black**：代码格式化
- **flake8**：代码检查
- **isort**：import排序

```bash
# 安装工具
pip install black flake8 isort

# 格式化代码
black .

# 检查代码
flake8 .

# 排序import
isort .
```

### 2. 命名规范

#### 2.1 模块和包名
- 使用小写字母
- 单词之间用下划线分隔
- 简短且有意义

```python
# 好的示例
user_service.py
api_client.py
test_user_api.py

# 不好的示例
UserService.py
apiClient.py
TestUserAPI.py
```

#### 2.2 类名
- 使用大驼峰命名（PascalCase）
- 名词或名词短语
- 测试类以 `Test` 开头

```python
# 好的示例
class UserService:
    pass

class OrderProcessor:
    pass

class TestUserAPI:
    pass

# 不好的示例
class user_service:
    pass

class orderprocessor:
    pass
```

#### 2.3 函数和方法名
- 使用小写字母
- 单词之间用下划线分隔
- 动词或动词短语
- 测试函数以 `test_` 开头

```python
# 好的示例
def create_user(data):
    pass

def test_create_user_success():
    pass

def test_create_user_with_duplicate_email():
    pass

# 不好的示例
def CreateUser(data):
    pass

def testCreateUser():
    pass
```

#### 2.4 变量名
- 使用小写字母
- 单词之间用下划线分隔
- 有意义的名称

```python
# 好的示例
user_name = "John"
total_count = 100
is_active = True

# 不好的示例
userName = "John"
tc = 100
flag = True
```

#### 2.5 常量名
- 全部大写字母
- 单词之间用下划线分隔

```python
# 好的示例
MAX_RETRY_TIMES = 3
DEFAULT_TIMEOUT = 30
API_BASE_URL = "http://localhost:8080"

# 不好的示例
max_retry_times = 3
defaultTimeout = 30
```

### 3. Import 规范

#### 3.1 Import 顺序
1. 标准库
2. 第三方库
3. 本地模块

各组之间用空行分隔

```python
# 好的示例
import os
import sys
from datetime import datetime

import pytest
import requests
from playwright.sync_api import Page

from utils.config import Config
from data.user_data import VALID_USER
```

#### 3.2 Import 格式
- 避免使用 `from module import *`
- 每行一个 import（多个可以合并）
- 使用绝对导入

```python
# 好的示例
from typing import Dict, List, Optional

# 可接受
from utils import config, logger

# 不好的示例
from utils import *
```

### 4. 代码布局

#### 4.1 缩进
- 使用 **4个空格**
- 不使用Tab

```python
# 好的示例
def create_user(data):
    if data:
        return process_data(data)
    return None

# 不好的示例（Tab或2空格）
def create_user(data):
  if data:
    return process_data(data)
  return None
```

#### 4.2 行长度
- 每行最多 **88个字符**（black默认）
- 超长行使用括号换行

```python
# 好的示例
user_data = {
    "username": "testuser",
    "email": "test@example.com",
    "password": "Test1234",
}

result = some_function(
    arg1="value1",
    arg2="value2",
    arg3="value3",
)

# 字符串换行
message = (
    "This is a very long message "
    "that spans multiple lines "
    "for better readability."
)
```

#### 4.3 空行
- 顶级函数和类定义之间：2个空行
- 类内方法之间：1个空行
- 函数内逻辑块之间：1个空行

```python
# 好的示例
class UserService:
    def __init__(self):
        self.users = []

    def create_user(self, data):
        # 验证数据
        self.validate_data(data)

        # 创建用户
        user = User(data)
        self.users.append(user)

        return user
```

### 5. 注释规范

#### 5.1 文档字符串（Docstring）
使用三引号，遵循 **Google风格**

```python
def create_user(username: str, email: str, password: str) -> dict:
    """创建新用户

    Args:
        username: 用户名，4-20个字符
        email: 邮箱地址
        password: 密码，至少8位

    Returns:
        用户对象字典，包含id、username、email等字段

    Raises:
        ValueError: 当参数不合法时
        DuplicateError: 当用户名或邮箱已存在时

    Examples:
        >>> create_user("john", "john@example.com", "Pass1234")
        {'id': 1, 'username': 'john', 'email': 'john@example.com'}
    """
    pass
```

#### 5.2 单行注释
- 使用 `#` 开头
- 与代码保持相同缩进
- `#` 后面有一个空格

```python
# 好的示例
def process_data(data):
    # 验证数据格式
    if not data:
        return None

    # 处理业务逻辑
    result = transform_data(data)

    return result

# 不好的示例
def process_data(data):
    #验证数据格式
    if not data:
        return None
```

#### 5.3 TODO注释
使用 `# TODO:` 标记待办事项

```python
# TODO: 添加邮箱验证
# TODO(zhangsan): 优化查询性能
# FIXME: 修复并发问题
```

### 6. 类型注解

使用 **类型提示（Type Hints）**

```python
from typing import Dict, List, Optional, Union

def get_user(user_id: int) -> Optional[Dict[str, any]]:
    """获取用户信息"""
    pass

def create_users(users: List[Dict[str, str]]) -> List[int]:
    """批量创建用户"""
    pass

class UserService:
    def __init__(self, config: Dict[str, str]) -> None:
        self.config = config

    def find_by_id(self, user_id: int) -> Optional[dict]:
        pass
```

### 7. 异常处理

#### 7.1 捕获具体异常
```python
# 好的示例
try:
    result = risky_operation()
except ValueError as e:
    logger.error(f"Invalid value: {e}")
except ConnectionError as e:
    logger.error(f"Connection failed: {e}")

# 不好的示例
try:
    result = risky_operation()
except Exception:  # 太宽泛
    pass
```

#### 7.2 使用自定义异常
```python
class UserNotFoundError(Exception):
    """用户不存在异常"""
    pass

class DuplicateUserError(Exception):
    """用户重复异常"""
    pass

def get_user(user_id: int) -> dict:
    user = find_user(user_id)
    if not user:
        raise UserNotFoundError(f"User {user_id} not found")
    return user
```

#### 7.3 异常链
```python
try:
    result = process_data(data)
except ValueError as e:
    raise ProcessError("Failed to process data") from e
```

## 测试代码规范

### 1. 测试文件组织

```
testing/
├── unit/                   # 单元测试
│   ├── test_user_service.py
│   └── test_order_service.py
├── integration/            # 集成测试
│   ├── test_user_api.py
│   └── test_order_api.py
├── e2e/                    # E2E测试
│   ├── test_user_flow.py
│   └── test_order_flow.py
├── data/                   # 测试数据
│   ├── user_data.py
│   └── order_data.py
├── fixtures/               # 测试fixture
│   └── common_fixtures.py
└── conftest.py            # pytest配置
```

### 2. 测试命名规范

#### 2.1 测试文件名
- 以 `test_` 开头
- 与被测试模块对应

```python
# 被测试文件：user_service.py
# 测试文件：test_user_service.py

# 被测试文件：api/user.py
# 测试文件：test_user_api.py
```

#### 2.2 测试类名
- 以 `Test` 开头
- 描述测试的功能模块

```python
class TestUserAPI:
    """用户API测试"""
    pass

class TestUserService:
    """用户服务测试"""
    pass
```

#### 2.3 测试函数名
格式：`test_<被测函数>_<场景>_<预期结果>`

```python
class TestUserAPI:
    def test_create_user_with_valid_data_returns_201(self):
        """测试创建用户成功"""
        pass

    def test_create_user_with_duplicate_email_returns_400(self):
        """测试邮箱重复返回400"""
        pass

    def test_create_user_without_username_returns_400(self):
        """测试缺少用户名返回400"""
        pass
```

### 3. 测试结构（AAA模式）

**Arrange-Act-Assert** 模式

```python
def test_create_user_success():
    """测试创建用户成功"""
    # Arrange - 准备测试数据
    user_data = {
        "username": "testuser",
        "email": "test@example.com",
        "password": "Test1234"
    }

    # Act - 执行被测试的操作
    response = requests.post(f"{BASE_URL}/api/users", json=user_data)

    # Assert - 验证结果
    assert response.status_code == 201
    assert response.json()["username"] == "testuser"
    assert "id" in response.json()
```

### 4. Fixture使用

```python
# conftest.py
import pytest

@pytest.fixture(scope="session")
def base_url():
    """API基础URL"""
    return "http://localhost:8080"

@pytest.fixture(scope="function")
def clean_database(db_connection):
    """清理数据库"""
    yield
    db_connection.execute("DELETE FROM users")

@pytest.fixture
def valid_user_data():
    """有效的用户数据"""
    return {
        "username": "testuser",
        "email": "test@example.com",
        "password": "Test1234"
    }

# test_user_api.py
def test_create_user(base_url, valid_user_data, clean_database):
    """测试创建用户"""
    response = requests.post(f"{base_url}/api/users", json=valid_user_data)
    assert response.status_code == 201
```

### 5. 参数化测试

```python
import pytest

@pytest.mark.parametrize("username,email,expected_status", [
    ("testuser", "test@example.com", 201),  # 成功
    ("a", "test@example.com", 400),         # 用户名太短
    ("testuser", "invalid", 400),            # 邮箱格式错误
    ("", "test@example.com", 400),          # 用户名为空
])
def test_create_user_validation(username, email, expected_status):
    """测试创建用户的验证"""
    response = requests.post(f"{BASE_URL}/api/users", json={
        "username": username,
        "email": email,
        "password": "Test1234"
    })
    assert response.status_code == expected_status
```

### 6. 测试标记

```python
import pytest

@pytest.mark.slow
def test_bulk_import():
    """慢速测试"""
    pass

@pytest.mark.integration
def test_api_integration():
    """集成测试"""
    pass

@pytest.mark.skip(reason="Feature not implemented yet")
def test_future_feature():
    """跳过测试"""
    pass

@pytest.mark.xfail
def test_known_bug():
    """预期失败"""
    pass

# 运行特定标记的测试
# pytest -m integration
# pytest -m "not slow"
```

## 代码质量工具

### 1. 配置文件

#### pyproject.toml
```toml
[tool.black]
line-length = 88
target-version = ['py310']
include = '\.pyi?$'

[tool.isort]
profile = "black"
line_length = 88

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = "test_*.py"
python_classes = "Test*"
python_functions = "test_*"
markers = [
    "slow: marks tests as slow",
    "integration: marks tests as integration tests",
]
```

#### .flake8
```ini
[flake8]
max-line-length = 88
extend-ignore = E203, W503
exclude = .git,__pycache__,venv,.venv
```

### 2. pre-commit配置

```.pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.1.0
    hooks:
      - id: black

  - repo: https://github.com/PyCQA/flake8
    rev: 6.0.0
    hooks:
      - id: flake8

  - repo: https://github.com/PyCQA/isort
    rev: 5.12.0
    hooks:
      - id: isort
```

## 最佳实践

### 1. 使用上下文管理器

```python
# 好的示例
with open("file.txt", "r") as f:
    content = f.read()

with database.connection() as conn:
    result = conn.execute(query)

# 不好的示例
f = open("file.txt", "r")
content = f.read()
f.close()
```

### 2. 使用列表推导式

```python
# 好的示例
squares = [x**2 for x in range(10)]
even_numbers = [x for x in numbers if x % 2 == 0]

# 不好的示例
squares = []
for x in range(10):
    squares.append(x**2)
```

### 3. 使用生成器

```python
# 好的示例（节省内存）
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()

# 使用
for line in read_large_file("large.txt"):
    process(line)
```

### 4. 使用f-string

```python
# 好的示例（Python 3.6+）
name = "John"
age = 30
message = f"Hello, {name}! You are {age} years old."

# 不好的示例
message = "Hello, %s! You are %d years old." % (name, age)
message = "Hello, {}! You are {} years old.".format(name, age)
```

### 5. 避免可变默认参数

```python
# 好的示例
def append_to_list(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# 不好的示例
def append_to_list(item, items=[]):  # 危险！
    items.append(item)
    return items
```

### 6. 使用 dataclass

```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    username: str
    email: str
    is_active: bool = True

    def to_dict(self) -> dict:
        return {
            "id": self.id,
            "username": self.username,
            "email": self.email,
            "is_active": self.is_active,
        }
```

## 性能优化

### 1. 使用内置函数
```python
# 好的示例
numbers = [1, 2, 3, 4, 5]
total = sum(numbers)
maximum = max(numbers)

# 不好的示例
total = 0
for num in numbers:
    total += num
```

### 2. 避免重复计算
```python
# 好的示例
def process_users(users):
    valid_users = [u for u in users if u.is_valid()]
    count = len(valid_users)

    print(f"Processing {count} users")
    return valid_users

# 不好的示例
def process_users(users):
    print(f"Processing {len([u for u in users if u.is_valid()])} users")
    return [u for u in users if u.is_valid()]
```

### 3. 使用集合（set）进行查找
```python
# 好的示例
valid_ids = {1, 2, 3, 4, 5}
if user_id in valid_ids:  # O(1)
    pass

# 不好的示例
valid_ids = [1, 2, 3, 4, 5]
if user_id in valid_ids:  # O(n)
    pass
```

## 安全性

### 1. 避免SQL注入
```python
# 好的示例（使用参数化查询）
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))

# 不好的示例（SQL注入风险）
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

### 2. 避免敏感信息泄露
```python
# 好的示例
logger.info(f"User {user_id} logged in")

# 不好的示例
logger.info(f"User {username} logged in with password {password}")
```

### 3. 使用环境变量
```python
import os

# 好的示例
API_KEY = os.getenv("API_KEY")
DATABASE_URL = os.getenv("DATABASE_URL")

# 不好的示例
API_KEY = "hardcoded_key_123"  # 危险！
```

## 总结

遵循本规范可以：
- ✅ 提高代码可读性
- ✅ 减少bug
- ✅ 提升团队协作效率
- ✅ 便于代码维护
- ✅ 提高测试质量

**记住**：好的代码是给人读的，机器只是顺便执行而已。
