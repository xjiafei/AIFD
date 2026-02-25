# 数据库设计最佳实践

> 本文档总结了数据库设计的行业最佳实践，涵盖设计原则、性能优化、事务处理等核心内容。

## 📋 目录

- [数据库设计原则](#数据库设计原则)
- [表设计规范](#表设计规范)
- [索引优化](#索引优化)
- [查询优化](#查询优化)
- [事务处理](#事务处理)
- [锁和并发控制](#锁和并发控制)
- [分库分表](#分库分表)
- [数据备份与恢复](#数据备份与恢复)
- [检查清单](#检查清单)

---

## 数据库设计原则

### 1. 三范式（3NF）

**第一范式（1NF）**：列不可再分

### ✅ 好的实践

```sql
-- 符合1NF：每个字段都是原子值
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100)
);
```

### ❌ 不好的实践

```sql
-- 违反1NF：name字段包含多个值
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    name VARCHAR(100),  -- "John Doe"
    email VARCHAR(100)
);
```

**第二范式（2NF）**：消除部分依赖

### ✅ 好的实践

```sql
-- 订单表
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    order_date DATETIME NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL
);

-- 订单明细表
CREATE TABLE order_items (
    id BIGINT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);
```

### ❌ 不好的实践

```sql
-- 违反2NF：product_name依赖于product_id，而非主键
CREATE TABLE order_items (
    order_id BIGINT,
    product_id BIGINT,
    product_name VARCHAR(100),  -- 部分依赖
    quantity INT,
    price DECIMAL(10, 2),
    PRIMARY KEY (order_id, product_id)
);
```

**第三范式（3NF）**：消除传递依赖

### ✅ 好的实践

```sql
-- 用户表
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    city_id BIGINT
);

-- 城市表
CREATE TABLE cities (
    id BIGINT PRIMARY KEY,
    name VARCHAR(50),
    province_id BIGINT
);

-- 省份表
CREATE TABLE provinces (
    id BIGINT PRIMARY KEY,
    name VARCHAR(50)
);
```

### ❌ 不好的实践

```sql
-- 违反3NF：province_name依赖于city_name
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    city_name VARCHAR(50),
    province_name VARCHAR(50)  -- 传递依赖
);
```

### 2. 反范式设计（性能优化）

在某些场景下，为了提高查询性能，可以适当违反范式。

### ✅ 好的实践（适度冗余）

```sql
-- 订单表（冗余用户名，避免频繁JOIN）
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    user_name VARCHAR(50),  -- 冗余字段，提高查询性能
    order_date DATETIME NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,

    INDEX idx_user_id (user_id)
);

-- 使用触发器或应用层保持数据一致性
```

**何时使用反范式**：
- 频繁查询，很少更新
- JOIN操作成为性能瓶颈
- 数据一致性要求不高

**注意事项**：
- 冗余数据需要维护一致性（触发器或应用层）
- 评估存储空间和维护成本

---

## 表设计规范

### 1. 命名规范

### ✅ 好的实践

```sql
-- 表名：小写字母，下划线分隔，使用复数
CREATE TABLE user_profiles (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    avatar_url VARCHAR(255),
    bio TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 索引命名
CREATE INDEX idx_user_profiles_user_id ON user_profiles(user_id);
CREATE UNIQUE INDEX uk_users_username ON users(username);
CREATE UNIQUE INDEX uk_users_email ON users(email);

-- 外键命名
ALTER TABLE user_profiles
ADD CONSTRAINT fk_user_profiles_user_id
FOREIGN KEY (user_id) REFERENCES users(id);
```

**命名规则**：
- 表名：`小写_复数`（如 `user_profiles`, `order_items`）
- 字段名：`小写_下划线`（如 `user_id`, `created_at`）
- 索引：`idx_表名_字段名`（如 `idx_users_email`）
- 唯一索引：`uk_表名_字段名`（如 `uk_users_username`）
- 外键：`fk_表名_字段名`（如 `fk_orders_user_id`）

### 2. 字段类型选择

### ✅ 好的实践

```sql
CREATE TABLE products (
    -- 主键：使用BIGINT（支持更大范围）
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,

    -- 字符串：根据实际长度选择VARCHAR
    name VARCHAR(100) NOT NULL,
    description TEXT,
    sku VARCHAR(50) NOT NULL,

    -- 数字：根据精度选择类型
    price DECIMAL(10, 2) NOT NULL,  -- 金额使用DECIMAL
    stock INT UNSIGNED NOT NULL DEFAULT 0,

    -- 布尔：使用TINYINT(1)
    is_active TINYINT(1) NOT NULL DEFAULT 1,

    -- 时间：使用DATETIME
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    -- 枚举：使用VARCHAR而非ENUM（更灵活）
    status VARCHAR(20) NOT NULL DEFAULT 'active',

    -- JSON：MySQL 5.7+支持
    attributes JSON,

    -- 索引
    INDEX idx_sku (sku),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**字段类型选择指南**：

| 数据类型 | 推荐使用 | 不推荐 |
|---------|---------|--------|
| 整数 | BIGINT, INT | TINYINT（除非确定范围小） |
| 小数 | DECIMAL | FLOAT, DOUBLE（精度问题） |
| 字符串 | VARCHAR, TEXT | CHAR（浪费空间） |
| 布尔 | TINYINT(1) | ENUM('Y','N') |
| 日期时间 | DATETIME | TIMESTAMP（2038年问题） |
| 枚举 | VARCHAR | ENUM（不灵活） |

### ❌ 不好的实践

```sql
CREATE TABLE products (
    -- 主键使用INT（范围不够）
    id INT PRIMARY KEY,

    -- 金额使用FLOAT（精度丢失）
    price FLOAT,

    -- 固定长度的CHAR（浪费空间）
    sku CHAR(100),

    -- ENUM（难以扩展）
    status ENUM('active', 'inactive'),

    -- TIMESTAMP（2038年问题）
    created_at TIMESTAMP
);
```

### 3. 必备字段

### ✅ 好的实践

```sql
CREATE TABLE users (
    -- 主键
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,

    -- 业务字段
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    password VARCHAR(255) NOT NULL,

    -- 通用字段（每张表都应该有）
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    created_by BIGINT COMMENT '创建人ID',
    updated_by BIGINT COMMENT '更新人ID',
    is_deleted TINYINT(1) NOT NULL DEFAULT 0 COMMENT '是否删除（逻辑删除）',
    version INT NOT NULL DEFAULT 0 COMMENT '版本号（乐观锁）',

    -- 唯一索引
    UNIQUE INDEX uk_username (username),
    UNIQUE INDEX uk_email (email),

    -- 普通索引
    INDEX idx_created_at (created_at),
    INDEX idx_is_deleted (is_deleted)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户表';
```

**必备字段说明**：
- `id`：主键，自增
- `created_at`：创建时间
- `updated_at`：更新时间
- `created_by`：创建人（可选）
- `updated_by`：更新人（可选）
- `is_deleted`：逻辑删除标记
- `version`：乐观锁版本号（高并发场景）

---

## 索引优化

### 1. 索引类型

**主键索引（PRIMARY KEY）**：

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
);
```

**唯一索引（UNIQUE INDEX）**：

```sql
-- 单列唯一索引
CREATE UNIQUE INDEX uk_users_username ON users(username);

-- 复合唯一索引
CREATE UNIQUE INDEX uk_user_profiles_user_platform
ON user_profiles(user_id, platform);
```

**普通索引（INDEX）**：

```sql
-- 单列索引
CREATE INDEX idx_users_email ON users(email);

-- 复合索引（注意字段顺序）
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
```

**全文索引（FULLTEXT）**：

```sql
-- 全文搜索索引（仅支持MyISAM和InnoDB 5.6+）
CREATE FULLTEXT INDEX ft_articles_title_content ON articles(title, content);

-- 使用全文索引
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST ('搜索关键词' IN NATURAL LANGUAGE MODE);
```

### 2. 索引设计原则

### ✅ 好的实践

```sql
-- 1. 在WHERE、ORDER BY、GROUP BY、JOIN字段上建索引
CREATE INDEX idx_orders_user_id ON orders(user_id);  -- WHERE user_id = ?
CREATE INDEX idx_orders_created_at ON orders(created_at);  -- ORDER BY created_at

-- 2. 复合索引遵循"最左前缀原则"
CREATE INDEX idx_users_city_age_name ON users(city_id, age, name);

-- 可以使用索引的查询：
-- WHERE city_id = ?
-- WHERE city_id = ? AND age = ?
-- WHERE city_id = ? AND age = ? AND name = ?

-- 不能使用索引的查询：
-- WHERE age = ?
-- WHERE name = ?

-- 3. 区分度高的字段放在前面
CREATE INDEX idx_orders_status_date ON orders(status, created_at);
-- status区分度高（只有几个值），created_at区分度低

-- 4. 覆盖索引（避免回表）
CREATE INDEX idx_users_email_name ON users(email, name);
-- 查询 SELECT name FROM users WHERE email = ? 只需要访问索引

-- 5. 前缀索引（字符串字段过长时）
CREATE INDEX idx_articles_url ON articles(url(50));  -- 只索引前50个字符
```

### ❌ 不好的实践

```sql
-- 1. 在低区分度字段上建索引（性别、状态等只有少量值）
CREATE INDEX idx_users_gender ON users(gender);  -- 只有男/女，区分度低

-- 2. 索引过多（影响写入性能）
CREATE INDEX idx1 ON users(name);
CREATE INDEX idx2 ON users(email);
CREATE INDEX idx3 ON users(phone);
CREATE INDEX idx4 ON users(city);
CREATE INDEX idx5 ON users(age);
-- ... 太多索引

-- 3. 在小表上建索引（全表扫描更快）
CREATE INDEX idx_small_table ON small_table(field);  -- 表只有100条数据

-- 4. 使用函数破坏索引
SELECT * FROM users WHERE YEAR(created_at) = 2026;  -- 无法使用索引
-- 应该改为：
SELECT * FROM users WHERE created_at >= '2026-01-01' AND created_at < '2027-01-01';

-- 5. 使用!=、<>、NOT IN破坏索引
SELECT * FROM users WHERE status != 'deleted';  -- 无法使用索引
-- 应该改为：
SELECT * FROM users WHERE status IN ('active', 'inactive');
```

### 3. 索引监控和优化

```sql
-- 查看表的索引
SHOW INDEX FROM users;

-- 查看索引使用情况
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 分析表（更新索引统计信息）
ANALYZE TABLE users;

-- 优化表（重建索引）
OPTIMIZE TABLE users;

-- 查看未使用的索引
SELECT
    t.table_schema,
    t.table_name,
    s.index_name,
    s.column_name
FROM information_schema.tables t
LEFT JOIN information_schema.statistics s ON t.table_schema = s.table_schema
    AND t.table_name = s.table_name
WHERE t.table_schema NOT IN ('mysql', 'information_schema', 'performance_schema')
    AND s.index_name NOT IN (
        SELECT DISTINCT index_name
        FROM mysql.innodb_index_stats
    );
```

---

## 查询优化

### 1. 使用EXPLAIN分析查询

### ✅ 好的实践

```sql
-- 分析查询执行计划
EXPLAIN SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.city_id = 1
ORDER BY u.created_at DESC
LIMIT 10;

-- 关注以下字段：
-- type: ALL（全表扫描，最差）< index < range < ref < eq_ref < const（最好）
-- key: 使用的索引
-- rows: 扫描的行数
-- Extra: Using filesort（需要排序，慢）, Using temporary（使用临时表，慢）
```

**EXPLAIN结果分析**：

| type类型 | 说明 | 性能 |
|---------|------|------|
| ALL | 全表扫描 | 最差 |
| index | 索引全扫描 | 差 |
| range | 索引范围扫描 | 中 |
| ref | 非唯一索引查找 | 好 |
| eq_ref | 唯一索引查找 | 很好 |
| const | 主键或唯一索引查找 | 最好 |

### 2. 避免SELECT *

### ✅ 好的实践

```sql
-- 只查询需要的字段
SELECT id, username, email FROM users WHERE id = 1;

-- 使用覆盖索引（查询字段都在索引中）
CREATE INDEX idx_users_email_name ON users(email, name);
SELECT email, name FROM users WHERE email = 'test@example.com';
```

### ❌ 不好的实践

```sql
-- 查询所有字段（包括大字段如TEXT、BLOB）
SELECT * FROM users WHERE id = 1;
```

### 3. 分页优化

### ✅ 好的实践

```sql
-- 方法1：使用子查询优化（适用于大offset）
SELECT u.*
FROM users u
JOIN (
    SELECT id FROM users ORDER BY created_at DESC LIMIT 1000, 20
) t ON u.id = t.id;

-- 方法2：使用游标分页（适用于无限滚动）
SELECT * FROM users
WHERE id > 12345  -- 上一页的最后一条记录的ID
ORDER BY id ASC
LIMIT 20;

-- 方法3：使用延迟关联（适用于复杂查询）
SELECT u.*
FROM users u
JOIN (
    SELECT id FROM users
    WHERE city_id = 1
    ORDER BY created_at DESC
    LIMIT 1000, 20
) t ON u.id = t.id;
```

### ❌ 不好的实践

```sql
-- 大offset导致性能问题
SELECT * FROM users ORDER BY created_at DESC LIMIT 1000000, 20;
-- MySQL需要扫描1000020条记录，然后丢弃前1000000条
```

### 4. JOIN优化

### ✅ 好的实践

```sql
-- 1. 小表驱动大表（LEFT JOIN时，左表是小表）
SELECT o.*
FROM orders o  -- 小表
LEFT JOIN users u ON o.user_id = u.id  -- 大表
WHERE o.status = 'pending';

-- 2. 确保JOIN字段有索引
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_id ON users(id);  -- 主键自动有索引

-- 3. 避免JOIN过多表（不超过3个）
SELECT u.username, o.order_no, p.name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
-- 如果需要JOIN更多表，考虑拆分查询或冗余数据

-- 4. 使用STRAIGHT_JOIN强制JOIN顺序（少用）
SELECT STRAIGHT_JOIN u.*, o.*
FROM small_table u
JOIN large_table o ON u.id = o.user_id;
```

### 5. 子查询优化

### ✅ 好的实践

```sql
-- 使用JOIN代替IN子查询（性能更好）
SELECT u.*
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'
GROUP BY u.id;

-- 而不是：
SELECT u.*
FROM users u
WHERE u.id IN (
    SELECT user_id FROM orders WHERE status = 'completed'
);

-- 使用EXISTS代替IN（大数据量时）
SELECT u.*
FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id AND o.status = 'completed'
);
```

### 6. 批量操作

### ✅ 好的实践

```sql
-- 批量插入（使用单条INSERT）
INSERT INTO users (username, email) VALUES
    ('user1', 'user1@example.com'),
    ('user2', 'user2@example.com'),
    ('user3', 'user3@example.com');

-- 批量更新（使用CASE WHEN）
UPDATE users SET status = CASE
    WHEN id = 1 THEN 'active'
    WHEN id = 2 THEN 'inactive'
    WHEN id = 3 THEN 'blocked'
END
WHERE id IN (1, 2, 3);
```

### ❌ 不好的实践

```sql
-- 逐条插入（性能差）
INSERT INTO users (username, email) VALUES ('user1', 'user1@example.com');
INSERT INTO users (username, email) VALUES ('user2', 'user2@example.com');
INSERT INTO users (username, email) VALUES ('user3', 'user3@example.com');
```

---

## 事务处理

### 1. ACID特性

- **原子性（Atomicity）**：事务中的所有操作要么全部成功，要么全部失败
- **一致性（Consistency）**：事务前后数据保持一致
- **隔离性（Isolation）**：事务之间互不干扰
- **持久性（Durability）**：事务提交后永久保存

### 2. 事务隔离级别

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 性能 |
|---------|-----|----------|------|------|
| READ UNCOMMITTED | ✓ | ✓ | ✓ | 最高 |
| READ COMMITTED | ✗ | ✓ | ✓ | 高 |
| REPEATABLE READ（默认） | ✗ | ✗ | ✓ | 中 |
| SERIALIZABLE | ✗ | ✗ | ✗ | 最低 |

### ✅ 好的实践

```java
// Spring事务管理
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private ProductRepository productRepository;

    /**
     * 创建订单（事务）
     */
    @Transactional(rollbackFor = Exception.class)
    public void createOrder(CreateOrderRequest request) {
        // 1. 创建订单
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setTotalAmount(request.getTotalAmount());
        orderRepository.save(order);

        // 2. 扣减库存
        for (OrderItem item : request.getItems()) {
            Product product = productRepository.findById(item.getProductId())
                .orElseThrow(() -> new ResourceNotFoundException("产品不存在"));

            if (product.getStock() < item.getQuantity()) {
                throw new BusinessException("库存不足");
            }

            product.setStock(product.getStock() - item.getQuantity());
            productRepository.save(product);
        }

        // 如果发生异常，整个事务回滚
    }

    /**
     * 只读事务（提高性能）
     */
    @Transactional(readOnly = true)
    public OrderVO getOrder(Long id) {
        Order order = orderRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("订单不存在"));
        return convertToVO(order);
    }

    /**
     * 指定隔离级别
     */
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void processOrder(Long id) {
        // 处理订单
    }

    /**
     * 指定传播行为
     */
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createLog(String message) {
        // 创建日志（新事务，不受外层事务影响）
    }
}
```

**事务传播行为**：

| 传播行为 | 说明 |
|---------|------|
| REQUIRED（默认） | 如果当前存在事务，则加入该事务；如果不存在，则创建新事务 |
| REQUIRES_NEW | 创建新事务，如果当前存在事务，则挂起当前事务 |
| SUPPORTS | 如果当前存在事务，则加入该事务；如果不存在，则以非事务方式执行 |
| NOT_SUPPORTED | 以非事务方式执行，如果当前存在事务，则挂起当前事务 |
| NEVER | 以非事务方式执行，如果当前存在事务，则抛出异常 |
| MANDATORY | 如果当前存在事务，则加入该事务；如果不存在，则抛出异常 |

### 3. 避免长事务

### ✅ 好的实践

```java
@Service
public class OrderService {

    /**
     * 拆分事务（将非核心操作移到事务外）
     */
    public void createOrder(CreateOrderRequest request) {
        // 事务外：发送MQ消息、调用外部API等
        String messageId = mqService.sendMessage("order_created", request);

        // 事务内：核心数据库操作
        doCreateOrder(request);

        // 事务外：发送邮件通知
        emailService.sendOrderConfirmation(request.getUserId());
    }

    @Transactional(rollbackFor = Exception.class)
    private void doCreateOrder(CreateOrderRequest request) {
        // 只包含核心数据库操作
        Order order = new Order();
        orderRepository.save(order);
    }
}
```

### ❌ 不好的实践

```java
@Service
public class OrderService {

    /**
     * 长事务（包含外部调用）
     */
    @Transactional(rollbackFor = Exception.class)
    public void createOrder(CreateOrderRequest request) {
        // 数据库操作
        Order order = new Order();
        orderRepository.save(order);

        // 外部API调用（可能很慢）
        paymentService.processPayment(order);  // 可能需要3-5秒

        // 发送邮件（可能失败）
        emailService.sendConfirmation(order);  // 可能需要2-3秒

        // 事务持续时间过长，占用数据库连接
    }
}
```

---

## 锁和并发控制

### 1. 乐观锁

### ✅ 好的实践

```sql
-- 使用版本号实现乐观锁
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    stock INT NOT NULL DEFAULT 0,
    version INT NOT NULL DEFAULT 0,  -- 版本号
    updated_at DATETIME
);
```

```java
// 后端实现
@Service
public class ProductService {

    /**
     * 使用乐观锁扣减库存
     */
    public boolean decreaseStock(Long productId, int quantity) {
        Product product = productRepository.findById(productId)
            .orElseThrow(() -> new ResourceNotFoundException("产品不存在"));

        // 检查库存
        if (product.getStock() < quantity) {
            throw new BusinessException("库存不足");
        }

        // 使用版本号更新（乐观锁）
        int updatedRows = productRepository.updateStockWithVersion(
            productId,
            product.getStock() - quantity,
            product.getVersion()
        );

        // 更新失败（版本号已变化，说明有并发修改）
        if (updatedRows == 0) {
            return false;  // 重试或抛出异常
        }

        return true;
    }
}

// MyBatis Mapper
@Update("UPDATE products SET stock = #{newStock}, version = version + 1 " +
        "WHERE id = #{id} AND version = #{version}")
int updateStockWithVersion(@Param("id") Long id,
                          @Param("newStock") int newStock,
                          @Param("version") int version);
```

### 2. 悲观锁

### ✅ 好的实践

```java
// 使用悲观锁（FOR UPDATE）
@Service
public class AccountService {

    @Transactional(rollbackFor = Exception.class)
    public void transfer(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        // 锁定账户（悲观锁）
        Account fromAccount = accountRepository.findByIdForUpdate(fromAccountId);
        Account toAccount = accountRepository.findByIdForUpdate(toAccountId);

        // 检查余额
        if (fromAccount.getBalance().compareTo(amount) < 0) {
            throw new BusinessException("余额不足");
        }

        // 转账
        fromAccount.setBalance(fromAccount.getBalance().subtract(amount));
        toAccount.setBalance(toAccount.getBalance().add(amount));

        accountRepository.save(fromAccount);
        accountRepository.save(toAccount);
    }
}

// MyBatis Mapper
@Select("SELECT * FROM accounts WHERE id = #{id} FOR UPDATE")
Account findByIdForUpdate(@Param("id") Long id);
```

**乐观锁 vs 悲观锁**：

| 锁类型 | 适用场景 | 优点 | 缺点 |
|-------|---------|------|------|
| 乐观锁 | 读多写少 | 性能好，无阻塞 | 更新失败需要重试 |
| 悲观锁 | 写多读少 | 数据一致性强 | 性能差，可能死锁 |

---

## 分库分表

### 1. 垂直拆分

**垂直分库**：按业务模块拆分

```
原来：
└── main_db
    ├── users
    ├── products
    ├── orders
    └── payments

拆分后：
├── user_db
│   └── users
├── product_db
│   └── products
├── order_db
│   └── orders
└── payment_db
    └── payments
```

**垂直分表**：按字段拆分

```sql
-- 原表（包含大字段）
CREATE TABLE articles (
    id BIGINT PRIMARY KEY,
    title VARCHAR(200),
    author VARCHAR(50),
    content TEXT,  -- 大字段
    created_at DATETIME
);

-- 拆分后
-- 主表（高频访问字段）
CREATE TABLE articles (
    id BIGINT PRIMARY KEY,
    title VARCHAR(200),
    author VARCHAR(50),
    created_at DATETIME
);

-- 扩展表（低频访问字段）
CREATE TABLE article_contents (
    article_id BIGINT PRIMARY KEY,
    content TEXT
);
```

### 2. 水平拆分

**水平分表**：按数据拆分

```sql
-- 按时间分表
CREATE TABLE orders_2026_01 (...);
CREATE TABLE orders_2026_02 (...);
CREATE TABLE orders_2026_03 (...);

-- 按Hash分表
CREATE TABLE users_0 (...);  -- user_id % 10 = 0
CREATE TABLE users_1 (...);  -- user_id % 10 = 1
...
CREATE TABLE users_9 (...);  -- user_id % 10 = 9
```

**分表策略**：

```java
// 使用ShardingSphere实现分表
@Configuration
public class ShardingConfig {

    @Bean
    public DataSource dataSource() {
        // 数据源配置
        Map<String, DataSource> dataSourceMap = new HashMap<>();
        dataSourceMap.put("ds0", createDataSource("jdbc:mysql://localhost:3306/db0"));
        dataSourceMap.put("ds1", createDataSource("jdbc:mysql://localhost:3306/db1"));

        // 分表规则
        TableRuleConfiguration orderTableRule = new TableRuleConfiguration(
            "orders",
            "ds${0..1}.orders_${0..9}"
        );

        // 分库策略（按user_id取模）
        orderTableRule.setDatabaseShardingStrategyConfig(
            new InlineShardingStrategyConfiguration("user_id", "ds${user_id % 2}")
        );

        // 分表策略（按order_id取模）
        orderTableRule.setTableShardingStrategyConfig(
            new InlineShardingStrategyConfiguration("order_id", "orders_${order_id % 10}")
        );

        // 创建ShardingDataSource
        ShardingRuleConfiguration shardingRuleConfig = new ShardingRuleConfiguration();
        shardingRuleConfig.getTableRuleConfigs().add(orderTableRule);

        return ShardingDataSourceFactory.createDataSource(dataSourceMap, shardingRuleConfig, new Properties());
    }
}
```

---

## 数据备份与恢复

### 1. 备份策略

### ✅ 好的实践

```bash
# 1. 全量备份（每天凌晨）
mysqldump -u root -p \
  --single-transaction \
  --master-data=2 \
  --flush-logs \
  --all-databases > /backup/full_backup_$(date +%Y%m%d).sql

# 2. 增量备份（每小时）
# 启用二进制日志
# my.cnf:
# log-bin=/var/log/mysql/mysql-bin
# expire_logs_days=7

# 备份二进制日志
mysqlbinlog /var/log/mysql/mysql-bin.000001 > /backup/binlog_$(date +%Y%m%d_%H).sql

# 3. 自动化备份脚本
#!/bin/bash
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d)
MYSQL_USER="root"
MYSQL_PASS="password"

# 全量备份
mysqldump -u$MYSQL_USER -p$MYSQL_PASS \
  --single-transaction \
  --master-data=2 \
  --flush-logs \
  --all-databases | gzip > $BACKUP_DIR/full_$DATE.sql.gz

# 删除7天前的备份
find $BACKUP_DIR -name "full_*.sql.gz" -mtime +7 -delete

# 4. 备份到远程服务器
rsync -avz $BACKUP_DIR root@backup-server:/data/mysql-backup/
```

### 2. 恢复策略

```bash
# 1. 全量恢复
gunzip < /backup/full_20260213.sql.gz | mysql -u root -p

# 2. 增量恢复（使用二进制日志）
# 恢复到指定时间点
mysqlbinlog --stop-datetime="2026-02-13 10:30:00" \
  /var/log/mysql/mysql-bin.000001 | mysql -u root -p

# 3. 恢复单个数据库
gunzip < /backup/full_20260213.sql.gz | \
  mysql -u root -p --one-database mydb

# 4. 验证备份（定期测试恢复）
# 恢复到测试环境
gunzip < /backup/full_20260213.sql.gz | \
  mysql -h test-server -u root -p
```

---

## 检查清单

### 数据库设计检查清单

```yaml
表设计:
  - [ ] 表名使用小写字母、下划线分隔、复数形式
  - [ ] 字段名使用小写字母、下划线分隔
  - [ ] 主键使用BIGINT UNSIGNED AUTO_INCREMENT
  - [ ] 包含created_at、updated_at字段
  - [ ] 包含is_deleted字段（逻辑删除）
  - [ ] 字段类型选择合理（金额用DECIMAL，时间用DATETIME）
  - [ ] 字符集使用utf8mb4
  - [ ] 每个字段都有COMMENT注释

索引设计:
  - [ ] 主键自动创建主键索引
  - [ ] 唯一字段创建唯一索引
  - [ ] WHERE、ORDER BY、GROUP BY字段创建索引
  - [ ] JOIN字段创建索引
  - [ ] 复合索引遵循"最左前缀原则"
  - [ ] 区分度高的字段放在索引前面
  - [ ] 索引数量控制在5个以内
  - [ ] 定期检查未使用的索引

查询优化:
  - [ ] 避免SELECT *
  - [ ] 使用EXPLAIN分析查询计划
  - [ ] 分页避免大offset
  - [ ] JOIN表数量不超过3个
  - [ ] 避免在WHERE中使用函数
  - [ ] 使用批量操作代替逐条操作

事务:
  - [ ] 事务操作添加@Transactional注解
  - [ ] 指定rollbackFor = Exception.class
  - [ ] 只读操作使用readOnly = true
  - [ ] 避免长事务
  - [ ] 事务内避免外部调用

锁:
  - [ ] 读多写少使用乐观锁
  - [ ] 写多读少使用悲观锁
  - [ ] 避免死锁（按顺序锁定资源）

备份:
  - [ ] 每天全量备份
  - [ ] 启用二进制日志（增量备份）
  - [ ] 备份文件异地存储
  - [ ] 定期测试恢复流程
```

---

## 总结

1. **规范设计**：遵循命名规范、字段类型规范、必备字段规范
2. **合理索引**：在高频查询字段上建索引，但不要过度索引
3. **优化查询**：使用EXPLAIN分析，避免全表扫描和大offset
4. **事务管理**：保证ACID特性，避免长事务
5. **并发控制**：根据场景选择乐观锁或悲观锁
6. **水平扩展**：数据量大时考虑分库分表
7. **数据安全**：定期备份，测试恢复
