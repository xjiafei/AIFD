# API设计最佳实践

> 本文档总结了RESTful API设计的行业最佳实践，帮助构建高质量、易维护、易扩展的API。

## 📋 目录

- [RESTful API设计原则](#restful-api设计原则)
- [API版本管理](#api版本管理)
- [请求响应格式](#请求响应格式)
- [错误处理](#错误处理)
- [分页和过滤](#分页和过滤)
- [限流和缓存](#限流和缓存)
- [API文档](#api文档)
- [安全性](#安全性)
- [检查清单](#检查清单)

---

## RESTful API设计原则

### 1. 使用标准HTTP方法

**基本原则**：
- `GET` - 获取资源（幂等）
- `POST` - 创建资源
- `PUT` - 完整更新资源（幂等）
- `PATCH` - 部分更新资源
- `DELETE` - 删除资源（幂等）

### ✅ 好的实践

```http
# 获取用户列表
GET /api/users

# 获取单个用户
GET /api/users/123

# 创建用户
POST /api/users

# 完整更新用户
PUT /api/users/123

# 部分更新用户（只更新邮箱）
PATCH /api/users/123

# 删除用户
DELETE /api/users/123
```

### ❌ 不好的实践

```http
# 不要在URL中使用动作动词
GET /api/getUsers
POST /api/createUser
POST /api/updateUser
POST /api/deleteUser

# 不要所有操作都用POST
POST /api/users?action=get
POST /api/users?action=create
```

### 2. 使用复数名词表示资源

### ✅ 好的实践

```http
GET /api/users           # 获取用户列表
GET /api/orders          # 获取订单列表
GET /api/products/123/reviews  # 获取产品的评论
```

### ❌ 不好的实践

```http
GET /api/user            # 单数形式
GET /api/getOrder        # 包含动词
GET /api/Product         # 首字母大写
```

### 3. 使用层级结构表达关系

### ✅ 好的实践

```http
# 获取用户的订单
GET /api/users/123/orders

# 获取订单的商品
GET /api/orders/456/items

# 创建用户的地址
POST /api/users/123/addresses

# 获取产品的评论
GET /api/products/789/reviews
```

```java
// 后端实现
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{userId}/orders")
    public ResponseEntity<List<OrderVO>> getUserOrders(
        @PathVariable Long userId,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size
    ) {
        List<OrderVO> orders = orderService.getUserOrders(userId, page, size);
        return ResponseEntity.ok(orders);
    }
}
```

### ❌ 不好的实践

```http
# 扁平化，丢失关系信息
GET /api/orders?userId=123

# 过度嵌套（超过3层）
GET /api/users/123/orders/456/items/789/details
```

### 4. 使用查询参数进行过滤、排序、搜索

### ✅ 好的实践

```http
# 过滤
GET /api/users?status=active&role=admin

# 排序
GET /api/users?sort=createdAt:desc

# 搜索
GET /api/users?search=john

# 分页
GET /api/users?page=1&size=20

# 组合使用
GET /api/products?category=electronics&minPrice=100&maxPrice=1000&sort=price:asc&page=1&size=20
```

```java
// 后端实现
@GetMapping
public ResponseEntity<PageResult<ProductVO>> getProducts(
    @RequestParam(required = false) String category,
    @RequestParam(required = false) BigDecimal minPrice,
    @RequestParam(required = false) BigDecimal maxPrice,
    @RequestParam(defaultValue = "createdAt:desc") String sort,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size
) {
    ProductQuery query = ProductQuery.builder()
        .category(category)
        .minPrice(minPrice)
        .maxPrice(maxPrice)
        .sort(sort)
        .page(page)
        .size(size)
        .build();

    PageResult<ProductVO> result = productService.queryProducts(query);
    return ResponseEntity.ok(result);
}
```

---

## API版本管理

### 1. URL路径版本（推荐）

### ✅ 好的实践

```http
# V1版本
GET /api/v1/users
POST /api/v1/orders

# V2版本
GET /api/v2/users
POST /api/v2/orders
```

**优点**：
- 简单明了
- 容易理解和实现
- 支持多版本共存
- 缓存友好

```java
// 后端实现
@RestController
@RequestMapping("/api/v1/users")
public class UserV1Controller {

    @GetMapping("/{id}")
    public ResponseEntity<UserV1VO> getUser(@PathVariable Long id) {
        // V1版本实现
    }
}

@RestController
@RequestMapping("/api/v2/users")
public class UserV2Controller {

    @GetMapping("/{id}")
    public ResponseEntity<UserV2VO> getUser(@PathVariable Long id) {
        // V2版本实现（可能包含更多字段）
    }
}
```

### 2. 请求头版本（备选）

```http
GET /api/users
Accept: application/vnd.myapi.v1+json

GET /api/users
Accept: application/vnd.myapi.v2+json
```

**缺点**：
- 不直观
- 调试困难
- 缓存复杂

### 3. 版本升级策略

### ✅ 好的实践

```yaml
# 向后兼容的变更（不需要新版本）
- 添加新的可选字段
- 添加新的端点
- 修复bug

# 不兼容的变更（需要新版本）
- 删除字段
- 重命名字段
- 改变字段类型
- 改变响应结构
```

**版本弃用通知**：

```java
@RestController
@RequestMapping("/api/v1/users")
@Deprecated // 标记为弃用
public class UserV1Controller {

    @GetMapping("/{id}")
    public ResponseEntity<UserV1VO> getUser(@PathVariable Long id) {
        // 在响应头中添加弃用警告
        HttpHeaders headers = new HttpHeaders();
        headers.add("Warning", "299 - \"API v1 is deprecated. Please use v2. Support ends on 2026-12-31\"");
        headers.add("Sunset", "Wed, 31 Dec 2026 23:59:59 GMT");

        UserV1VO user = userService.getUserV1(id);
        return ResponseEntity.ok().headers(headers).body(user);
    }
}
```

---

## 请求响应格式

### 1. 统一响应格式

### ✅ 好的实践

```json
// 成功响应
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "timestamp": "2026-02-13T10:30:00Z"
}

// 列表响应
{
  "code": 200,
  "message": "success",
  "data": {
    "items": [
      {"id": 1, "name": "Item 1"},
      {"id": 2, "name": "Item 2"}
    ],
    "pagination": {
      "page": 1,
      "size": 20,
      "total": 100,
      "totalPages": 5
    }
  },
  "timestamp": "2026-02-13T10:30:00Z"
}
```

```java
// 后端实现
@Data
@Builder
public class ApiResponse<T> {
    private Integer code;
    private String message;
    private T data;
    private String timestamp;

    public static <T> ApiResponse<T> success(T data) {
        return ApiResponse.<T>builder()
            .code(200)
            .message("success")
            .data(data)
            .timestamp(Instant.now().toString())
            .build();
    }

    public static <T> ApiResponse<T> error(int code, String message) {
        return ApiResponse.<T>builder()
            .code(code)
            .message(message)
            .timestamp(Instant.now().toString())
            .build();
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserVO>> getUser(@PathVariable Long id) {
        UserVO user = userService.getUser(id);
        return ResponseEntity.ok(ApiResponse.success(user));
    }
}
```

### 2. 请求格式规范

### ✅ 好的实践

```json
// POST /api/users
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "profile": {
    "firstName": "John",
    "lastName": "Doe",
    "phone": "+1234567890"
  }
}
```

```java
// 后端DTO
@Data
@Validated
public class CreateUserRequest {

    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度必须在3-20之间")
    @Pattern(regexp = "^[a-zA-Z0-9_]+$", message = "用户名只能包含字母、数字和下划线")
    private String username;

    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;

    @NotBlank(message = "密码不能为空")
    @Size(min = 8, max = 20, message = "密码长度必须在8-20之间")
    private String password;

    @Valid
    private UserProfile profile;
}

@Data
public class UserProfile {
    @NotBlank(message = "名字不能为空")
    private String firstName;

    @NotBlank(message = "姓氏不能为空")
    private String lastName;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "手机号格式不正确")
    private String phone;
}
```

### 3. 字段命名规范

### ✅ 好的实践

```json
// 使用驼峰命名
{
  "userId": 123,
  "firstName": "John",
  "lastName": "Doe",
  "createdAt": "2026-02-13T10:30:00Z",
  "isActive": true,
  "orderCount": 10
}
```

### ❌ 不好的实践

```json
// 混合使用不同命名风格
{
  "user_id": 123,           // 下划线
  "FirstName": "John",      // 首字母大写
  "last-name": "Doe",       // 短横线
  "created_at": "...",      // 下划线
  "IsActive": true,         // 首字母大写
  "order-count": 10         // 短横线
}
```

---

## 错误处理

### 1. 使用标准HTTP状态码

**常用状态码**：

| 状态码 | 含义 | 使用场景 |
|-------|------|---------|
| 200 | OK | 请求成功 |
| 201 | Created | 创建成功 |
| 204 | No Content | 删除成功 |
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 409 | Conflict | 资源冲突 |
| 422 | Unprocessable Entity | 业务验证失败 |
| 429 | Too Many Requests | 限流 |
| 500 | Internal Server Error | 服务器错误 |
| 503 | Service Unavailable | 服务不可用 |

### 2. 错误响应格式

### ✅ 好的实践

```json
// 通用错误
{
  "code": 400,
  "message": "请求参数错误",
  "errors": [
    {
      "field": "email",
      "message": "邮箱格式不正确"
    },
    {
      "field": "password",
      "message": "密码长度必须在8-20之间"
    }
  ],
  "timestamp": "2026-02-13T10:30:00Z",
  "path": "/api/users",
  "traceId": "abc123def456"
}

// 业务错误
{
  "code": 40001,
  "message": "用户名已存在",
  "timestamp": "2026-02-13T10:30:00Z",
  "path": "/api/users",
  "traceId": "abc123def456"
}
```

```java
// 后端实现
@Data
@Builder
public class ErrorResponse {
    private Integer code;
    private String message;
    private List<FieldError> errors;
    private String timestamp;
    private String path;
    private String traceId;

    @Data
    @AllArgsConstructor
    public static class FieldError {
        private String field;
        private String message;
    }
}

@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理参数校验异常
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(
        MethodArgumentNotValidException ex,
        HttpServletRequest request
    ) {
        List<ErrorResponse.FieldError> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> new ErrorResponse.FieldError(
                error.getField(),
                error.getDefaultMessage()
            ))
            .collect(Collectors.toList());

        ErrorResponse response = ErrorResponse.builder()
            .code(400)
            .message("请求参数错误")
            .errors(errors)
            .timestamp(Instant.now().toString())
            .path(request.getRequestURI())
            .traceId(MDC.get("traceId"))
            .build();

        return ResponseEntity.badRequest().body(response);
    }

    /**
     * 处理业务异常
     */
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(
        BusinessException ex,
        HttpServletRequest request
    ) {
        ErrorResponse response = ErrorResponse.builder()
            .code(ex.getCode())
            .message(ex.getMessage())
            .timestamp(Instant.now().toString())
            .path(request.getRequestURI())
            .traceId(MDC.get("traceId"))
            .build();

        return ResponseEntity
            .status(ex.getHttpStatus())
            .body(response);
    }

    /**
     * 处理资源不存在异常
     */
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(
        ResourceNotFoundException ex,
        HttpServletRequest request
    ) {
        ErrorResponse response = ErrorResponse.builder()
            .code(404)
            .message(ex.getMessage())
            .timestamp(Instant.now().toString())
            .path(request.getRequestURI())
            .traceId(MDC.get("traceId"))
            .build();

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(response);
    }

    /**
     * 处理未知异常
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(
        Exception ex,
        HttpServletRequest request
    ) {
        log.error("Unexpected error", ex);

        ErrorResponse response = ErrorResponse.builder()
            .code(500)
            .message("服务器内部错误")
            .timestamp(Instant.now().toString())
            .path(request.getRequestURI())
            .traceId(MDC.get("traceId"))
            .build();

        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(response);
    }
}

// 自定义业务异常
public class BusinessException extends RuntimeException {
    private final int code;
    private final HttpStatus httpStatus;

    public BusinessException(int code, String message, HttpStatus httpStatus) {
        super(message);
        this.code = code;
        this.httpStatus = httpStatus;
    }

    // 预定义的业务异常
    public static BusinessException userNameExists() {
        return new BusinessException(40001, "用户名已存在", HttpStatus.BAD_REQUEST);
    }

    public static BusinessException emailExists() {
        return new BusinessException(40002, "邮箱已存在", HttpStatus.BAD_REQUEST);
    }

    public static BusinessException invalidCredentials() {
        return new BusinessException(40101, "用户名或密码错误", HttpStatus.UNAUTHORIZED);
    }
}
```

---

## 分页和过滤

### 1. 分页参数

### ✅ 好的实践

```http
# 基于页码的分页（适合UI展示）
GET /api/users?page=1&size=20

# 基于游标的分页（适合无限滚动）
GET /api/users?cursor=abc123&limit=20
```

```java
// 基于页码的分页
@Data
public class PageQuery {
    @Min(value = 0, message = "页码不能小于0")
    private int page = 0;

    @Min(value = 1, message = "每页数量不能小于1")
    @Max(value = 100, message = "每页数量不能大于100")
    private int size = 20;

    private String sort;
}

@Data
@Builder
public class PageResult<T> {
    private List<T> items;
    private Pagination pagination;

    @Data
    @Builder
    public static class Pagination {
        private int page;
        private int size;
        private long total;
        private int totalPages;
        private boolean hasNext;
        private boolean hasPrevious;
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    public ResponseEntity<ApiResponse<PageResult<UserVO>>> getUsers(
        @Valid PageQuery query
    ) {
        PageResult<UserVO> result = userService.queryUsers(query);
        return ResponseEntity.ok(ApiResponse.success(result));
    }
}

// Service实现
@Service
public class UserServiceImpl implements UserService {

    @Override
    public PageResult<UserVO> queryUsers(PageQuery query) {
        // 使用MyBatis-Plus分页
        Page<User> page = new Page<>(query.getPage(), query.getSize());
        Page<User> userPage = userMapper.selectPage(page, null);

        List<UserVO> userVOs = userPage.getRecords()
            .stream()
            .map(this::convertToVO)
            .collect(Collectors.toList());

        return PageResult.<UserVO>builder()
            .items(userVOs)
            .pagination(PageResult.Pagination.builder()
                .page((int) userPage.getCurrent())
                .size((int) userPage.getSize())
                .total(userPage.getTotal())
                .totalPages((int) userPage.getPages())
                .hasNext(userPage.hasNext())
                .hasPrevious(userPage.hasPrevious())
                .build())
            .build();
    }
}
```

### 2. 过滤和搜索

### ✅ 好的实践

```http
# 精确匹配
GET /api/users?status=active&role=admin

# 模糊搜索
GET /api/users?search=john

# 范围查询
GET /api/products?minPrice=100&maxPrice=1000
GET /api/orders?startDate=2026-01-01&endDate=2026-12-31

# 包含关系
GET /api/users?roles=admin,editor

# 排序
GET /api/users?sort=createdAt:desc,name:asc
```

```java
@Data
public class UserQuery extends PageQuery {
    private String status;
    private String role;
    private String search;
    private LocalDate startDate;
    private LocalDate endDate;

    // 转换为MyBatis-Plus查询条件
    public LambdaQueryWrapper<User> toQueryWrapper() {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();

        // 精确匹配
        if (StringUtils.hasText(status)) {
            wrapper.eq(User::getStatus, status);
        }
        if (StringUtils.hasText(role)) {
            wrapper.eq(User::getRole, role);
        }

        // 模糊搜索
        if (StringUtils.hasText(search)) {
            wrapper.and(w -> w
                .like(User::getUsername, search)
                .or()
                .like(User::getEmail, search)
            );
        }

        // 范围查询
        if (startDate != null) {
            wrapper.ge(User::getCreatedAt, startDate);
        }
        if (endDate != null) {
            wrapper.le(User::getCreatedAt, endDate);
        }

        // 排序
        if (StringUtils.hasText(getSort())) {
            parseSort(getSort()).forEach(sortField -> {
                if (sortField.isAsc()) {
                    wrapper.orderByAsc(getSortColumn(sortField.getField()));
                } else {
                    wrapper.orderByDesc(getSortColumn(sortField.getField()));
                }
            });
        }

        return wrapper;
    }
}
```

---

## 限流和缓存

### 1. 限流策略

### ✅ 好的实践

```java
// 使用Redis + 滑动窗口算法实现限流
@Component
public class RateLimiter {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    /**
     * 限流检查
     * @param key 限流键（如用户ID、IP地址）
     * @param maxRequests 最大请求数
     * @param windowSeconds 时间窗口（秒）
     * @return 是否允许通过
     */
    public boolean isAllowed(String key, int maxRequests, int windowSeconds) {
        String redisKey = "rate_limit:" + key;
        long now = System.currentTimeMillis();
        long windowStart = now - windowSeconds * 1000L;

        // 清理过期数据
        redisTemplate.opsForZSet().removeRangeByScore(redisKey, 0, windowStart);

        // 获取当前窗口内的请求数
        Long count = redisTemplate.opsForZSet().count(redisKey, windowStart, now);

        if (count != null && count >= maxRequests) {
            return false;
        }

        // 添加当前请求
        redisTemplate.opsForZSet().add(redisKey, String.valueOf(now), now);
        redisTemplate.expire(redisKey, windowSeconds, TimeUnit.SECONDS);

        return true;
    }
}

// 使用拦截器实现限流
@Component
public class RateLimitInterceptor implements HandlerInterceptor {

    @Autowired
    private RateLimiter rateLimiter;

    @Override
    public boolean preHandle(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler
    ) throws Exception {
        // 获取用户标识（可以是用户ID、IP地址等）
        String userId = getUserId(request);
        String key = userId != null ? "user:" + userId : "ip:" + getClientIP(request);

        // 限流检查（每分钟最多100次请求）
        if (!rateLimiter.isAllowed(key, 100, 60)) {
            response.setStatus(429);
            response.setContentType("application/json");
            response.getWriter().write("{\"code\":429,\"message\":\"请求过于频繁，请稍后再试\"}");
            return false;
        }

        return true;
    }
}

// 使用注解实现限流
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimit {
    int maxRequests() default 100;
    int windowSeconds() default 60;
}

@Aspect
@Component
public class RateLimitAspect {

    @Autowired
    private RateLimiter rateLimiter;

    @Around("@annotation(rateLimit)")
    public Object around(ProceedingJoinPoint pjp, RateLimit rateLimit) throws Throwable {
        HttpServletRequest request = ((ServletRequestAttributes)
            RequestContextHolder.currentRequestAttributes()).getRequest();

        String userId = getUserId(request);
        String key = userId != null ? "user:" + userId : "ip:" + getClientIP(request);

        if (!rateLimiter.isAllowed(key, rateLimit.maxRequests(), rateLimit.windowSeconds())) {
            throw new BusinessException(429, "请求过于频繁，请稍后再试", HttpStatus.TOO_MANY_REQUESTS);
        }

        return pjp.proceed();
    }
}

// 使用示例
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    @RateLimit(maxRequests = 50, windowSeconds = 60)
    public ResponseEntity<ApiResponse<UserVO>> getUser(@PathVariable Long id) {
        UserVO user = userService.getUser(id);
        return ResponseEntity.ok(ApiResponse.success(user));
    }
}
```

### 2. 缓存策略

### ✅ 好的实践

```java
// 使用Spring Cache + Redis
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();

        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .transactionAware()
            .build();
    }
}

// Service层使用缓存
@Service
public class UserServiceImpl implements UserService {

    /**
     * 获取用户（带缓存）
     */
    @Cacheable(value = "users", key = "#id", unless = "#result == null")
    public UserVO getUser(Long id) {
        log.debug("从数据库查询用户: {}", id);
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("用户不存在"));
        return convertToVO(user);
    }

    /**
     * 更新用户（清除缓存）
     */
    @CacheEvict(value = "users", key = "#id")
    public void updateUser(Long id, UpdateUserRequest request) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("用户不存在"));

        // 更新用户信息
        BeanUtils.copyProperties(request, user);
        userRepository.save(user);
    }

    /**
     * 删除用户（清除缓存）
     */
    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    /**
     * 查询用户列表（带缓存）
     */
    @Cacheable(value = "userList", key = "#query.toString()")
    public PageResult<UserVO> queryUsers(UserQuery query) {
        // 查询逻辑
    }
}

// HTTP缓存头
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserVO>> getUser(@PathVariable Long id) {
        UserVO user = userService.getUser(id);

        // 设置缓存头
        CacheControl cacheControl = CacheControl
            .maxAge(10, TimeUnit.MINUTES)
            .cachePublic();

        return ResponseEntity
            .ok()
            .cacheControl(cacheControl)
            .eTag(String.valueOf(user.hashCode()))
            .body(ApiResponse.success(user));
    }
}
```

---

## API文档

### 1. OpenAPI 3.0规范

### ✅ 好的实践

```yaml
openapi: 3.0.0
info:
  title: 用户管理API
  version: 1.0.0
  description: 提供用户管理相关的API接口
  contact:
    name: API Support
    email: api@example.com
servers:
  - url: http://localhost:8080
    description: 开发环境
  - url: https://api.example.com
    description: 生产环境

paths:
  /api/users:
    get:
      summary: 获取用户列表
      description: 分页查询用户列表，支持过滤和排序
      tags:
        - User
      parameters:
        - name: page
          in: query
          description: 页码（从0开始）
          schema:
            type: integer
            default: 0
        - name: size
          in: query
          description: 每页数量
          schema:
            type: integer
            default: 20
            minimum: 1
            maximum: 100
        - name: status
          in: query
          description: 用户状态
          schema:
            type: string
            enum: [active, inactive, blocked]
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserPageResult'
        '400':
          description: 请求参数错误
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

    post:
      summary: 创建用户
      description: 创建新用户
      tags:
        - User
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: 创建成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserVO'
        '400':
          description: 请求参数错误
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

  /api/users/{id}:
    get:
      summary: 获取用户详情
      tags:
        - User
      parameters:
        - name: id
          in: path
          required: true
          description: 用户ID
          schema:
            type: integer
            format: int64
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserVO'
        '404':
          description: 用户不存在
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

components:
  schemas:
    UserVO:
      type: object
      properties:
        id:
          type: integer
          format: int64
          description: 用户ID
        username:
          type: string
          description: 用户名
        email:
          type: string
          format: email
          description: 邮箱
        status:
          type: string
          enum: [active, inactive, blocked]
          description: 用户状态
        createdAt:
          type: string
          format: date-time
          description: 创建时间

    CreateUserRequest:
      type: object
      required:
        - username
        - email
        - password
      properties:
        username:
          type: string
          minLength: 3
          maxLength: 20
          pattern: '^[a-zA-Z0-9_]+$'
          description: 用户名
        email:
          type: string
          format: email
          description: 邮箱
        password:
          type: string
          minLength: 8
          maxLength: 20
          format: password
          description: 密码

    UserPageResult:
      type: object
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/UserVO'
        pagination:
          type: object
          properties:
            page:
              type: integer
            size:
              type: integer
            total:
              type: integer
              format: int64
            totalPages:
              type: integer

    ErrorResponse:
      type: object
      properties:
        code:
          type: integer
        message:
          type: string
        timestamp:
          type: string
          format: date-time
        path:
          type: string
```

### 2. 使用Swagger UI

```java
// 添加依赖
// pom.xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>

// 配置
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("用户管理API")
                .version("1.0.0")
                .description("提供用户管理相关的API接口")
                .contact(new Contact()
                    .name("API Support")
                    .email("api@example.com")))
            .servers(List.of(
                new Server().url("http://localhost:8080").description("开发环境"),
                new Server().url("https://api.example.com").description("生产环境")
            ));
    }
}

// Controller添加注解
@RestController
@RequestMapping("/api/users")
@Tag(name = "User", description = "用户管理API")
public class UserController {

    @GetMapping("/{id}")
    @Operation(summary = "获取用户详情", description = "根据用户ID获取用户详细信息")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "成功",
            content = @Content(schema = @Schema(implementation = UserVO.class))),
        @ApiResponse(responseCode = "404", description = "用户不存在",
            content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
    })
    public ResponseEntity<ApiResponse<UserVO>> getUser(
        @Parameter(description = "用户ID", required = true)
        @PathVariable Long id
    ) {
        UserVO user = userService.getUser(id);
        return ResponseEntity.ok(ApiResponse.success(user));
    }
}
```

**访问Swagger UI**：
- URL: http://localhost:8080/swagger-ui/index.html
- API文档: http://localhost:8080/v3/api-docs

---

## 安全性

### 1. 认证和授权

### ✅ 好的实践

```java
// JWT Token生成和验证
@Component
public class JwtTokenProvider {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private long expiration;

    /**
     * 生成Token
     */
    public String generateToken(User user) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + expiration);

        return Jwts.builder()
            .setSubject(user.getId().toString())
            .claim("username", user.getUsername())
            .claim("role", user.getRole())
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(Keys.hmacShaKeyFor(secret.getBytes()), SignatureAlgorithm.HS512)
            .compact();
    }

    /**
     * 验证Token
     */
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(Keys.hmacShaKeyFor(secret.getBytes()))
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }

    /**
     * 从Token中获取用户ID
     */
    public Long getUserIdFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(Keys.hmacShaKeyFor(secret.getBytes()))
            .build()
            .parseClaimsJws(token)
            .getBody();

        return Long.parseLong(claims.getSubject());
    }
}

// 认证过滤器
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtTokenProvider tokenProvider;

    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {
        try {
            String token = getTokenFromRequest(request);

            if (StringUtils.hasText(token) && tokenProvider.validateToken(token)) {
                Long userId = tokenProvider.getUserIdFromToken(token);

                // 设置认证信息到Security Context
                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(userId, null, null);
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            log.error("Could not set user authentication", e);
        }

        filterChain.doFilter(request, response);
    }

    private String getTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

### 2. 敏感数据保护

### ✅ 好的实践

```java
// 密码加密
@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public void createUser(CreateUserRequest request) {
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());

        // 密码加密
        user.setPassword(passwordEncoder.encode(request.getPassword()));

        userRepository.save(user);
    }
}

// 响应中隐藏敏感字段
@Data
public class UserVO {
    private Long id;
    private String username;
    private String email;

    @JsonIgnore  // 不返回密码
    private String password;

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)  // 只写不读
    private String ssn;
}
```

---

## 检查清单

### API设计检查清单

```yaml
基础设计:
  - [ ] 使用标准HTTP方法（GET、POST、PUT、PATCH、DELETE）
  - [ ] 资源使用复数名词
  - [ ] URL结构清晰，层级不超过3层
  - [ ] 查询参数用于过滤、排序、分页

版本管理:
  - [ ] API有明确的版本号
  - [ ] 版本号在URL中体现（/api/v1/）
  - [ ] 有版本弃用计划和通知机制

请求响应:
  - [ ] 请求和响应使用JSON格式
  - [ ] 字段命名使用驼峰命名法
  - [ ] 响应格式统一（包含code、message、data、timestamp）
  - [ ] 请求参数有完整的校验

错误处理:
  - [ ] 使用标准HTTP状态码
  - [ ] 错误响应包含详细的错误信息
  - [ ] 有全局异常处理机制
  - [ ] 生产环境不暴露敏感信息

分页和过滤:
  - [ ] 列表接口支持分页
  - [ ] 分页参数统一（page、size）
  - [ ] 支持排序（sort参数）
  - [ ] 支持过滤和搜索

性能优化:
  - [ ] 实现了限流机制
  - [ ] 使用了缓存策略
  - [ ] 响应头包含Cache-Control
  - [ ] 支持ETag

安全性:
  - [ ] 实现了认证机制（JWT）
  - [ ] 实现了授权机制
  - [ ] 密码使用加密存储
  - [ ] 响应中隐藏敏感字段
  - [ ] 防护了常见攻击（SQL注入、XSS等）

文档:
  - [ ] 有完整的OpenAPI规范文档
  - [ ] 有Swagger UI界面
  - [ ] 每个接口都有详细说明
  - [ ] 有请求和响应示例

测试:
  - [ ] 有集成测试覆盖所有接口
  - [ ] 有性能测试
  - [ ] 有安全测试
```

---

## 总结

1. **保持一致性**：整个API的设计风格要统一
2. **遵循标准**：使用HTTP标准、RESTful规范、OpenAPI规范
3. **考虑扩展性**：设计时考虑未来的变化，使用版本管理
4. **注重性能**：使用缓存、限流等机制
5. **保证安全性**：认证、授权、数据加密
6. **完善文档**：提供清晰的API文档
7. **全面测试**：接口测试、性能测试、安全测试
