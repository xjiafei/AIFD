# 后端开发最佳实践

> AI全流程研发知识库 - Java/Spring Boot开发最佳实践
>
> **版本**: v1.0
> **更新日期**: 2026-02-13
> **适用范围**: Java后端开发

## 目录

- [1. 项目结构](#1-项目结构)
- [2. 分层架构](#2-分层架构)
- [3. 依赖注入](#3-依赖注入)
- [4. 异常处理](#4-异常处理)
- [5. 事务管理](#5-事务管理)
- [6. 数据库操作](#6-数据库操作)
- [7. 缓存策略](#7-缓存策略)
- [8. 异步处理](#8-异步处理)
- [9. 日志规范](#9-日志规范)
- [10. 配置管理](#10-配置管理)

---

## 1. 项目结构

### 1.1 Maven多模块结构

```
backend/
├── pom.xml                          # 父POM
├── common/                          # 公共模块
│   ├── src/main/java/.../common/
│   │   ├── constant/                # 常量定义
│   │   ├── util/                    # 工具类
│   │   ├── exception/               # 自定义异常
│   │   └── response/                # 统一响应
│   └── pom.xml
├── domain/                          # 领域模块
│   ├── src/main/java/.../domain/
│   │   ├── entity/                  # 实体类
│   │   ├── dto/                     # 数据传输对象
│   │   ├── vo/                      # 视图对象
│   │   └── enums/                   # 枚举
│   └── pom.xml
├── service/                         # 业务逻辑模块
│   ├── src/main/java/.../service/
│   │   ├── UserService.java        # 服务接口
│   │   └── impl/
│   │       └── UserServiceImpl.java # 服务实现
│   └── pom.xml
├── api/                             # API接口模块
│   ├── src/main/java/.../api/
│   │   ├── controller/              # 控制器
│   │   ├── filter/                  # 过滤器
│   │   └── interceptor/             # 拦截器
│   └── pom.xml
├── infrastructure/                  # 基础设施模块
│   ├── src/main/java/.../infrastructure/
│   │   ├── repository/              # 数据访问
│   │   ├── cache/                   # 缓存
│   │   └── mq/                      # 消息队列
│   └── pom.xml
└── application/                     # 启动模块
    ├── src/main/java/.../Application.java
    ├── src/main/resources/
    │   ├── application.yml
    │   ├── application-dev.yml
    │   └── application-prod.yml
    └── pom.xml
```

### 1.2 包结构规范

```java
com.company.project/
├── common/                  # 公共包
│   ├── constant/            # 常量
│   │   ├── ErrorCode.java
│   │   └── CommonConstants.java
│   ├── util/                # 工具类
│   │   ├── DateUtil.java
│   │   └── JsonUtil.java
│   ├── exception/           # 异常
│   │   ├── BusinessException.java
│   │   └── ResourceNotFoundException.java
│   └── response/            # 响应
│       ├── Result.java
│       └── PageResult.java
├── domain/                  # 领域包
│   ├── entity/              # 实体
│   │   └── User.java
│   ├── dto/                 # DTO
│   │   ├── CreateUserRequest.java
│   │   └── UpdateUserRequest.java
│   ├── vo/                  # VO
│   │   └── UserVO.java
│   └── enums/               # 枚举
│       └── UserStatus.java
├── service/                 # 服务包
│   ├── UserService.java
│   └── impl/
│       └── UserServiceImpl.java
├── api/                     # API包
│   └── controller/
│       └── UserController.java
└── infrastructure/          # 基础设施包
    └── repository/
        └── UserRepository.java
```

---

## 2. 分层架构

### 2.1 标准分层

```
┌──────────────────────────────────────┐
│        Controller层（API层）           │  接收请求、参数验证、调用Service
├──────────────────────────────────────┤
│          Service层（业务层）          │  业务逻辑、事务控制
├──────────────────────────────────────┤
│      Repository层（数据访问层）        │  数据库操作
├──────────────────────────────────────┤
│         Domain层（领域模型层）         │  实体、DTO、VO
└──────────────────────────────────────┘
```

### 2.2 各层职责

#### Controller层
```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    /**
     * 创建用户
     * 职责：
     * 1. 参数验证（@Valid）
     * 2. 调用Service
     * 3. 返回响应
     */
    @PostMapping
    public Result<UserVO> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.createUser(request);
        return Result.success(UserVO.from(user));
    }
}
```

**Controller层规则**：
```
✅ 应该做：
- 参数验证（使用@Valid）
- 调用Service方法
- 返回统一响应格式

❌ 不应该做：
- 包含业务逻辑
- 直接操作数据库
- 处理事务
- 包含复杂计算
```

#### Service层
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;

    /**
     * 创建用户
     * 职责：
     * 1. 业务逻辑处理
     * 2. 事务管理
     * 3. 调用其他Service
     * 4. 调用Repository
     */
    @Override
    @Transactional
    public User createUser(CreateUserRequest request) {
        // 1. 业务验证
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new BusinessException("用户名已存在");
        }

        // 2. 创建实体
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));

        // 3. 保存数据
        user = userRepository.save(user);

        // 4. 发送欢迎邮件
        emailService.sendWelcomeEmail(user.getEmail());

        return user;
    }
}
```

**Service层规则**：
```
✅ 应该做：
- 实现业务逻辑
- 管理事务
- 调用Repository
- 调用其他Service
- 抛出业务异常

❌ 不应该做：
- 直接操作数据库连接
- 包含SQL语句
- 处理HTTP请求/响应
```

#### Repository层
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    /**
     * 数据访问层
     * 职责：
     * 1. 定义数据访问接口
     * 2. 简单查询使用方法名
     * 3. 复杂查询使用@Query
     */

    // 方法名查询
    User findByUsername(String username);

    boolean existsByUsername(String username);

    List<User> findByStatus(UserStatus status);

    // JPQL查询
    @Query("SELECT u FROM User u WHERE u.email = :email AND u.status = :status")
    Optional<User> findByEmailAndStatus(@Param("email") String email,
                                       @Param("status") UserStatus status);

    // 原生SQL查询
    @Query(value = "SELECT * FROM users WHERE created_at > :date", nativeQuery = true)
    List<User> findRecentUsers(@Param("date") LocalDateTime date);
}
```

**Repository层规则**：
```
✅ 应该做：
- 定义数据访问方法
- 简单查询用方法名
- 复杂查询用@Query

❌ 不应该做：
- 包含业务逻辑
- 调用其他Service
- 处理事务
```

---

## 3. 依赖注入

### 3.1 构造器注入（推荐）

```java
✅ 好的实践（构造器注入 + @RequiredArgsConstructor）：

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;
    private final PasswordEncoder passwordEncoder;

    // Lombok自动生成构造器
}

优点：
- 依赖关系清晰
- 便于单元测试
- 保证依赖不可变（final）
- 避免循环依赖

❌ 不好的实践（字段注入）：

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailService emailService;

    // 难以测试，容易循环依赖
}
```

### 3.2 避免循环依赖

```java
❌ 循环依赖示例：

@Service
public class UserService {
    @Autowired
    private OrderService orderService;  // UserService依赖OrderService
}

@Service
public class OrderService {
    @Autowired
    private UserService userService;    // OrderService依赖UserService
}

✅ 解决方案1：重新设计，提取公共Service

@Service
public class UserService {
    @Autowired
    private CommonService commonService;
}

@Service
public class OrderService {
    @Autowired
    private CommonService commonService;
}

@Service
public class CommonService {
    // 公共逻辑
}

✅ 解决方案2：使用事件驱动

@Service
public class UserService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;

    public void deleteUser(Long userId) {
        // 删除用户
        userRepository.deleteById(userId);

        // 发布事件
        eventPublisher.publishEvent(new UserDeletedEvent(userId));
    }
}

@Service
public class OrderService {

    @EventListener
    public void handleUserDeleted(UserDeletedEvent event) {
        // 删除用户的订单
        orderRepository.deleteByUserId(event.getUserId());
    }
}
```

---

## 4. 异常处理

### 4.1 自定义业务异常

```java
/**
 * 业务异常基类
 */
public class BusinessException extends RuntimeException {

    private final String code;
    private final Object[] args;

    public BusinessException(String code, String message, Object... args) {
        super(message);
        this.code = code;
        this.args = args;
    }

    public BusinessException(ErrorCode errorCode, Object... args) {
        super(errorCode.getMessage());
        this.code = errorCode.getCode();
        this.args = args;
    }
}

/**
 * 资源未找到异常
 */
public class ResourceNotFoundException extends BusinessException {

    public ResourceNotFoundException(String resourceName, Object id) {
        super("RESOURCE_NOT_FOUND",
            String.format("%s with id %s not found", resourceName, id));
    }
}

/**
 * 未授权异常
 */
public class UnauthorizedException extends BusinessException {

    public UnauthorizedException(String message) {
        super("UNAUTHORIZED", message);
    }
}
```

### 4.2 全局异常处理

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    /**
     * 业务异常
     */
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<Result<?>> handleBusinessException(BusinessException e) {
        log.warn("业务异常: code={}, message={}", e.getCode(), e.getMessage());
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(Result.error(e.getCode(), e.getMessage()));
    }

    /**
     * 资源未找到异常
     */
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Result<?>> handleResourceNotFound(ResourceNotFoundException e) {
        log.warn("资源未找到: {}", e.getMessage());
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(Result.error("RESOURCE_NOT_FOUND", e.getMessage()));
    }

    /**
     * 参数验证异常
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Result<?>> handleValidationException(
            MethodArgumentNotValidException e) {

        List<String> errors = e.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());

        log.warn("参数验证失败: {}", errors);

        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(Result.error("VALIDATION_ERROR", "参数验证失败", errors));
    }

    /**
     * 系统异常
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<Result<?>> handleException(Exception e) {
        log.error("系统异常", e);
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(Result.error("SYSTEM_ERROR", "系统错误，请稍后重试"));
    }
}
```

### 4.3 异常使用规范

```
✅ 好的实践：
- 业务异常继承RuntimeException
- 异常信息要明确、友好
- 使用全局异常处理器
- 记录异常日志
- 不要吞掉异常

❌ 不好的实践：
- 捕获Exception但不处理
- 异常信息模糊："出错了"
- 抛出受检异常（强制调用者处理）
- 在Controller层处理异常
- 吞掉异常（catch后什么都不做）
```

---

## 5. 事务管理

### 5.1 声明式事务

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)  // 默认只读事务
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final AccountRepository accountRepository;

    /**
     * 只读操作，使用默认的只读事务
     */
    @Override
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }

    /**
     * 写操作，覆盖默认配置，使用读写事务
     */
    @Override
    @Transactional
    public User createUser(CreateUserRequest request) {
        // 创建用户
        User user = new User();
        user.setUsername(request.getUsername());
        user = userRepository.save(user);

        // 创建账户（同一事务）
        Account account = new Account();
        account.setUserId(user.getId());
        accountRepository.save(account);

        return user;
    }

    /**
     * 需要新事务的场景
     * 例如：记录日志，即使主业务失败也要记录
     */
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logUserAction(Long userId, String action) {
        // 独立事务，不受外部事务影响
        actionLogRepository.save(new ActionLog(userId, action));
    }
}
```

### 5.2 事务传播行为

```java
// REQUIRED（默认）：如果当前有事务，加入；否则创建新事务
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    // 业务逻辑
}

// REQUIRES_NEW：总是创建新事务，挂起当前事务
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() {
    // 独立事务
}

// NESTED：嵌套事务，可以回滚到savepoint
@Transactional(propagation = Propagation.NESTED)
public void methodC() {
    // 嵌套事务
}

// NOT_SUPPORTED：以非事务方式运行，挂起当前事务
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void methodD() {
    // 非事务
}
```

### 5.3 事务最佳实践

```
✅ 好的实践：
- 类级别使用@Transactional(readOnly = true)
- 写操作方法覆盖为@Transactional
- 事务方法尽量短小
- 避免在事务内执行RPC调用
- 避免在事务内执行长时间操作
- 异常要抛出（不要catch后不处理）

❌ 不好的实践：
- 事务方法过长，包含大量逻辑
- 在事务内执行HTTP请求
- 在事务内执行文件IO
- catch异常后不抛出
- 滥用REQUIRES_NEW
```

---

## 6. 数据库操作

### 6.1 批量操作

```java
✅ 好的实践（批量操作）：

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    /**
     * 批量插入
     */
    @Transactional
    public void batchCreate(List<CreateUserRequest> requests) {
        List<User> users = requests.stream()
            .map(this::toUser)
            .collect(Collectors.toList());

        // 批量保存（JPA会优化为批量插入）
        userRepository.saveAll(users);
    }

    /**
     * 批量更新（使用@Modifying）
     */
    @Transactional
    public int batchUpdateStatus(List<Long> userIds, UserStatus status) {
        return userRepository.batchUpdateStatus(userIds, status);
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id IN :ids")
    int batchUpdateStatus(@Param("ids") List<Long> ids,
                         @Param("status") UserStatus status);
}

❌ 不好的实践（循环单条操作）：

@Transactional
public void batchCreate(List<CreateUserRequest> requests) {
    for (CreateUserRequest request : requests) {
        User user = toUser(request);
        userRepository.save(user);  // N次数据库操作
    }
}
```

### 6.2 分页查询

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    /**
     * 分页查询
     */
    public PageResult<UserVO> searchUsers(UserSearchRequest request) {
        // 构建查询条件
        Specification<User> spec = (root, query, cb) -> {
            List<Predicate> predicates = new ArrayList<>();

            if (StringUtils.hasText(request.getUsername())) {
                predicates.add(cb.like(
                    root.get("username"),
                    "%" + request.getUsername() + "%"
                ));
            }

            if (request.getStatus() != null) {
                predicates.add(cb.equal(root.get("status"), request.getStatus()));
            }

            return cb.and(predicates.toArray(new Predicate[0]));
        };

        // 构建分页参数
        Sort sort = Sort.by(Sort.Direction.DESC, "createdAt");
        Pageable pageable = PageRequest.of(
            request.getPage() - 1,  // 页码从0开始
            request.getSize(),
            sort
        );

        // 执行查询
        Page<User> page = userRepository.findAll(spec, pageable);

        // 转换为VO
        List<UserVO> vos = page.getContent().stream()
            .map(UserVO::from)
            .collect(Collectors.toList());

        return PageResult.of(
            vos,
            page.getTotalElements(),
            page.getTotalPages(),
            request.getPage()
        );
    }
}
```

### 6.3 N+1查询问题

```java
❌ N+1问题示例：

@Entity
public class User {
    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}

// 查询用户列表
List<User> users = userRepository.findAll();  // 1次查询

// 遍历用户，获取订单
for (User user : users) {
    List<Order> orders = user.getOrders();  // N次查询
}

✅ 解决方案1：使用@EntityGraph

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @EntityGraph(attributePaths = {"orders"})
    List<User> findAll();  // 使用JOIN FETCH，1次查询
}

✅ 解决方案2：使用JPQL JOIN FETCH

@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();

✅ 解决方案3：分批查询（大数据量时）

// 1. 查询用户列表
List<User> users = userRepository.findAll();

// 2. 提取用户ID
List<Long> userIds = users.stream()
    .map(User::getId)
    .collect(Collectors.toList());

// 3. 批量查询订单
List<Order> orders = orderRepository.findByUserIdIn(userIds);

// 4. 在内存中组装
Map<Long, List<Order>> orderMap = orders.stream()
    .collect(Collectors.groupingBy(Order::getUserId));

users.forEach(user -> user.setOrders(orderMap.get(user.getId())));
```

---

## 7. 缓存策略

### 7.1 Spring Cache

```java
@Service
@RequiredArgsConstructor
@CacheConfig(cacheNames = "users")
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    /**
     * 缓存查询结果
     */
    @Override
    @Cacheable(key = "#id")
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }

    /**
     * 更新缓存
     */
    @Override
    @CachePut(key = "#user.id")
    @Transactional
    public User updateUser(User user) {
        return userRepository.save(user);
    }

    /**
     * 清除缓存
     */
    @Override
    @CacheEvict(key = "#id")
    @Transactional
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    /**
     * 清除所有缓存
     */
    @CacheEvict(allEntries = true)
    public void clearCache() {
        // 清除所有users缓存
    }
}
```

### 7.2 缓存最佳实践

```
✅ 好的实践：
- 缓存读多写少的数据
- 设置合理的过期时间
- 使用统一的缓存key规则
- 缓存空值防止缓存穿透
- 使用缓存更新策略

❌ 不好的实践：
- 缓存频繁变化的数据
- 缓存过期时间过长
- 缓存key不规范
- 不处理缓存穿透
- 不处理缓存雪崩
```

#### 缓存配置
```yaml
# application.yml
spring:
  cache:
    type: redis
    redis:
      time-to-live: 3600000  # 1小时
      cache-null-values: true # 缓存空值
  data:
    redis:
      host: localhost
      port: 6379
      password: ${REDIS_PASSWORD}
      database: 0
      timeout: 3000
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
```

---

## 8. 异步处理

### 8.1 异步方法

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {

    /**
     * 异步发送邮件
     */
    @Async
    public CompletableFuture<Boolean> sendEmail(String to, String subject, String content) {
        try {
            // 发送邮件逻辑
            log.info("发送邮件给: {}", to);
            Thread.sleep(3000);  // 模拟耗时操作
            return CompletableFuture.completedFuture(true);
        } catch (Exception e) {
            log.error("发送邮件失败", e);
            return CompletableFuture.completedFuture(false);
        }
    }
}

@Service
public class UserService {

    @Autowired
    private EmailService emailService;

    @Transactional
    public User createUser(CreateUserRequest request) {
        // 创建用户（同步）
        User user = userRepository.save(toUser(request));

        // 发送欢迎邮件（异步，不影响主流程）
        emailService.sendEmail(
            user.getEmail(),
            "欢迎注册",
            "欢迎加入我们！"
        );

        return user;
    }
}
```

### 8.2 消息队列

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final RabbitTemplate rabbitTemplate;

    /**
     * 创建订单后发送消息
     */
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // 1. 创建订单
        Order order = orderRepository.save(toOrder(request));

        // 2. 发送消息通知
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            order.getTotalAmount()
        );

        rabbitTemplate.convertAndSend("order.exchange", "order.created", event);

        return order;
    }
}

@Component
@Slf4j
public class OrderEventListener {

    /**
     * 监听订单创建事件
     */
    @RabbitListener(queues = "order.notification.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("订单创建通知: orderId={}", event.getOrderId());

        // 发送通知（邮件、短信等）
        notificationService.sendOrderNotification(event);
    }
}
```

---

## 9. 日志规范

### 9.1 日志级别

```java
@Service
@Slf4j
public class UserService {

    /**
     * TRACE: 最详细的信息，生产环境不开启
     */
    public void methodA() {
        log.trace("进入方法methodA，参数: {}", params);
    }

    /**
     * DEBUG: 调试信息，开发环境使用
     */
    public void methodB() {
        log.debug("查询用户: userId={}", userId);
    }

    /**
     * INFO: 重要业务流程
     */
    public User createUser(CreateUserRequest request) {
        log.info("创建用户: username={}", request.getUsername());
        // 业务逻辑
        log.info("用户创建成功: userId={}", user.getId());
        return user;
    }

    /**
     * WARN: 潜在问题，需要注意
     */
    public void methodD() {
        if (cache == null) {
            log.warn("缓存未初始化，使用数据库查询");
        }
    }

    /**
     * ERROR: 错误，需要处理
     */
    public void methodE() {
        try {
            // 业务逻辑
        } catch (Exception e) {
            log.error("业务处理失败: userId={}", userId, e);
            throw new BusinessException("处理失败");
        }
    }
}
```

### 9.2 日志规范

```
✅ 好的实践：
- 使用SLF4J + Logback
- 使用占位符而非字符串拼接
- 记录关键业务节点
- 异常日志包含堆栈信息
- 敏感信息脱敏

❌ 不好的实践：
- 使用System.out.println
- 字符串拼接（log.info("用户" + user)）
- 日志过多或过少
- 不记录异常堆栈
- 记录敏感信息（密码、身份证）
```

#### Logback配置
```xml
<!-- logback-spring.xml -->
<configuration>
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 文件输出 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 错误日志单独文件 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <file>logs/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/error-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 开发环境 -->
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>

    <!-- 生产环境 -->
    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="FILE" />
            <appender-ref ref="ERROR_FILE" />
        </root>
    </springProfile>
</configuration>
```

---

## 10. 配置管理

### 10.1 配置文件分离

```yaml
# application.yml（公共配置）
spring:
  application:
    name: backend-service
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}

---
# application-dev.yml（开发环境）
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb?useSSL=false
    username: root
    password: root
  data:
    redis:
      host: localhost
      port: 6379

logging:
  level:
    root: DEBUG

---
# application-prod.yml（生产环境）
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  data:
    redis:
      host: ${REDIS_HOST}
      port: ${REDIS_PORT}
      password: ${REDIS_PASSWORD}

logging:
  level:
    root: INFO
```

### 10.2 配置类

```java
@Configuration
@ConfigurationProperties(prefix = "app")
@Data
public class AppProperties {

    private Jwt jwt = new Jwt();
    private Upload upload = new Upload();

    @Data
    public static class Jwt {
        private String secret;
        private long expirationTime;
    }

    @Data
    public static class Upload {
        private String path;
        private long maxSize;
        private List<String> allowedExtensions;
    }
}
```

```yaml
# application.yml
app:
  jwt:
    secret: ${JWT_SECRET}
    expiration-time: 86400000
  upload:
    path: /data/uploads
    max-size: 10485760
    allowed-extensions:
      - jpg
      - png
      - pdf
```

---

## 总结

### 开发检查清单

```yaml
项目结构:
  - [ ] 使用Maven多模块
  - [ ] 包结构清晰合理
  - [ ] 依赖关系正确

分层架构:
  - [ ] Controller只处理HTTP
  - [ ] Service包含业务逻辑
  - [ ] Repository只做数据访问
  - [ ] 职责划分清晰

依赖注入:
  - [ ] 使用构造器注入
  - [ ] 避免循环依赖
  - [ ] 依赖声明为final

异常处理:
  - [ ] 自定义业务异常
  - [ ] 全局异常处理器
  - [ ] 异常信息明确

事务管理:
  - [ ] 使用声明式事务
  - [ ] 事务方法简短
  - [ ] 正确使用传播行为

数据库操作:
  - [ ] 批量操作避免循环
  - [ ] 分页查询使用Pageable
  - [ ] 解决N+1查询问题

缓存策略:
  - [ ] 合理使用缓存
  - [ ] 设置过期时间
  - [ ] 处理缓存穿透

异步处理:
  - [ ] 使用@Async异步方法
  - [ ] 配置线程池
  - [ ] 使用消息队列

日志规范:
  - [ ] 使用SLF4J
  - [ ] 日志级别正确
  - [ ] 敏感信息脱敏

配置管理:
  - [ ] 配置文件分环境
  - [ ] 敏感配置使用环境变量
  - [ ] 使用@ConfigurationProperties
```

---

**记住**：好的后端代码是结构清晰、职责明确、易于测试和维护的！
