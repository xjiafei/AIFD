# 数据库设计规范

## 命名

### 表命名

- 表前缀: `t_`
- 使用复数名词: `users`, `orders`, `order_items`
- 使用snake_case: `user_profiles`, `product_categories`
- 关联表: `user_roles`, `order_products`

### 字段命名

- 使用snake_case: `first_name`, `created_at`
- 主键: `id` (BIGINT AUTO_INCREMENT)
- 外键: `{表名单数}_id` (例: `user_id`, `order_id`)
- 布尔字段: `is_`, `has_`, `can_` 前缀 (例: `is_active`, `has_permission`)
- 时间字段:
  - `created_at` (创建时间)
  - `updated_at` (更新时间)
  - `deleted_at` (软删除时间)

### 索引命名

- 主键: `pk_{表名}` (例: `pk_users`)
- 唯一索引: `uk_{表名}_{字段名}` (例: `uk_users_email`)
- 普通索引: `idx_{表名}_{字段名}` (例: `idx_users_name`)
- 外键索引: `fk_{表名}_{字段名}` (例: `fk_orders_user_id`)

## 数据类型选择

- **ID**: BIGINT
- **短文本**: VARCHAR(n)
- **长文本**: TEXT
- **金额**: DECIMAL(precision, scale)
- **日期时间**: TIMESTAMP 或 DATETIME
- **布尔**: TINYINT(1) 或 BOOLEAN
- **枚举**: VARCHAR，而不是ENUM

## 必备字段
- `id` (主键)
- `remark` (备注)
- `deleted` (软删标识)
- `created_at` (创建时间)
- `updated_at` (更新时间)

## 注释说明
- 字段和表必须要有注释，如果是枚举类型的值，需要给出示例值和说明

## 示例表结构

```sql
CREATE TABLE t_users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    is_active TINYINT(1) DEFAULT 1,
    deleted TINYINT(1) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT uk_users_email UNIQUE (email),
    INDEX idx_users_is_active (is_active),
    INDEX idx_users_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
