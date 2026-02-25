# Java编码规范

## 命名约定

- **类名**: PascalCase (例: UserService, OrderController)
- **方法名**: camelCase (例: getUserById, createOrder)
- **常量**: UPPER_SNAKE_CASE (例: MAX_RETRY_COUNT)
- **包名**: 小写，使用点分隔 (例: com.example.service)

## 代码结构

### Controller层
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserDTO user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}
```

### Service层
```java
@Service
@Transactional
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public UserDTO createUser(CreateUserRequest request) {
        // Business logic here
    }
}
```

### Repository层
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```

## 最佳实践

1. 使用@Slf4j进行日志记录
2. 使用@Valid进行参数验证
3. 使用DTO进行数据传输
4. 异常统一处理使用@ControllerAdvice
5. 使用Optional处理可能为空的返回值
6. Service方法添加@Transactional注解
7. 避免在Controller中编写业务逻辑
