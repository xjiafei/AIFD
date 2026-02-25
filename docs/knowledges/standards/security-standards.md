# 安全开发规范

> AI全流程研发知识库 - 安全编码和常见漏洞防范
>
> **版本**: v1.0
> **更新日期**: 2026-02-13
> **适用范围**: 所有代码开发

## 目录

- [1. OWASP Top 10](#1-owasp-top-10)
- [2. 身份认证安全](#2-身份认证安全)
- [3. 会话管理安全](#3-会话管理安全)
- [4. 输入验证](#4-输入验证)
- [5. 输出编码](#5-输出编码)
- [6. SQL注入防护](#6-sql注入防护)
- [7. XSS防护](#7-xss防护)
- [8. CSRF防护](#8-csrf防护)
- [9. 访问控制](#9-访问控制)
- [10. 敏感数据保护](#10-敏感数据保护)
- [11. 日志和监控](#11-日志和监控)
- [12. 依赖安全](#12-依赖安全)

---

## 1. OWASP Top 10

### 1.1 2021版OWASP Top 10

```
A01:2021 – 访问控制失效（Broken Access Control）
A02:2021 – 加密机制失效（Cryptographic Failures）
A03:2021 – 注入（Injection）
A04:2021 – 不安全设计（Insecure Design）
A05:2021 – 安全配置错误（Security Misconfiguration）
A06:2021 – 易受攻击和过时的组件（Vulnerable and Outdated Components）
A07:2021 – 身份识别和身份验证失败（Identification and Authentication Failures）
A08:2021 – 软件和数据完整性失效（Software and Data Integrity Failures）
A09:2021 – 安全日志和监控失效（Security Logging and Monitoring Failures）
A10:2021 – 服务器端请求伪造（Server-Side Request Forgery, SSRF）
```

### 1.2 重点关注

**必须防护**：
1. SQL注入
2. XSS（跨站脚本）
3. CSRF（跨站请求伪造）
4. 未授权访问
5. 敏感数据泄露

---

## 2. 身份认证安全

### 2.1 密码策略

#### 密码强度要求
```yaml
最小长度: 8位
必须包含:
  - 大写字母
  - 小写字母
  - 数字
  - 特殊字符（可选，推荐）

禁止:
  - 常见弱密码（123456, password）
  - 用户名相同
  - 连续字符（abc123, 111111）
  - 键盘序列（qwerty, asdfgh）
```

#### 密码存储
```java
✅ 好的实践（使用BCrypt）：

import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

public class PasswordService {
    private final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(12);

    // 密码加密
    public String encode(String rawPassword) {
        return encoder.encode(rawPassword);
    }

    // 密码验证
    public boolean matches(String rawPassword, String encodedPassword) {
        return encoder.matches(rawPassword, encodedPassword);
    }
}

❌ 不好的实践：

// 明文存储
user.setPassword(rawPassword);

// MD5/SHA1（已不安全）
String hash = DigestUtils.md5Hex(password);

// 自定义加密算法（不推荐）
String encrypted = myCustomEncrypt(password);
```

### 2.2 多因素认证（MFA）

#### 实现TOTP
```java
import com.google.common.io.BaseEncoding;
import org.apache.commons.codec.binary.Base32;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public class TotpService {

    /**
     * 生成密钥
     */
    public String generateSecret() {
        byte[] buffer = new byte[20];
        new SecureRandom().nextBytes(buffer);
        return new Base32().encodeAsString(buffer);
    }

    /**
     * 验证TOTP码
     */
    public boolean verify(String secret, String code) {
        long currentTime = System.currentTimeMillis() / 1000 / 30;

        // 允许前后30秒的时间窗口
        for (int i = -1; i <= 1; i++) {
            if (generateTotp(secret, currentTime + i).equals(code)) {
                return true;
            }
        }

        return false;
    }

    private String generateTotp(String secret, long time) {
        try {
            byte[] key = new Base32().decode(secret);
            byte[] data = ByteBuffer.allocate(8).putLong(time).array();

            Mac mac = Mac.getInstance("HmacSHA1");
            mac.init(new SecretKeySpec(key, "HmacSHA1"));
            byte[] hash = mac.doFinal(data);

            int offset = hash[hash.length - 1] & 0x0F;
            int binary = ((hash[offset] & 0x7F) << 24)
                | ((hash[offset + 1] & 0xFF) << 16)
                | ((hash[offset + 2] & 0xFF) << 8)
                | (hash[offset + 3] & 0xFF);

            int otp = binary % 1000000;
            return String.format("%06d", otp);
        } catch (Exception e) {
            throw new RuntimeException("生成TOTP失败", e);
        }
    }
}
```

### 2.3 登录防护

#### 防暴力破解
```java
@Service
public class LoginAttemptService {

    private static final int MAX_ATTEMPTS = 5;
    private static final int LOCK_TIME_MINUTES = 10;

    private final LoadingCache<String, Integer> attemptsCache;

    public LoginAttemptService() {
        attemptsCache = CacheBuilder.newBuilder()
            .expireAfterWrite(LOCK_TIME_MINUTES, TimeUnit.MINUTES)
            .build(new CacheLoader<String, Integer>() {
                @Override
                public Integer load(String key) {
                    return 0;
                }
            });
    }

    /**
     * 登录成功，清除失败记录
     */
    public void loginSucceeded(String username) {
        attemptsCache.invalidate(username);
    }

    /**
     * 登录失败，记录失败次数
     */
    public void loginFailed(String username) {
        int attempts = attemptsCache.getUnchecked(username);
        attemptsCache.put(username, attempts + 1);
    }

    /**
     * 检查是否被锁定
     */
    public boolean isBlocked(String username) {
        return attemptsCache.getUnchecked(username) >= MAX_ATTEMPTS;
    }
}
```

---

## 3. 会话管理安全

### 3.1 JWT Token安全

#### 安全的JWT实现
```java
import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import java.security.Key;
import java.util.Date;

@Service
public class JwtTokenProvider {

    // 使用环境变量存储密钥
    private final Key secretKey;
    private final long validityInMilliseconds = 86400000; // 24小时

    public JwtTokenProvider(@Value("${jwt.secret}") String secret) {
        // 密钥长度至少256位
        this.secretKey = Keys.hmacShaKeyFor(secret.getBytes());
    }

    /**
     * 生成Token
     */
    public String createToken(String username, List<String> roles) {
        Claims claims = Jwts.claims().setSubject(username);
        claims.put("roles", roles);

        Date now = new Date();
        Date validity = new Date(now.getTime() + validityInMilliseconds);

        return Jwts.builder()
            .setClaims(claims)
            .setIssuedAt(now)
            .setExpiration(validity)
            .signWith(secretKey, SignatureAlgorithm.HS256)
            .compact();
    }

    /**
     * 验证Token
     */
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(secretKey)
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            log.error("Invalid JWT token: {}", e.getMessage());
            return false;
        }
    }

    /**
     * 获取用户名
     */
    public String getUsername(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(secretKey)
            .build()
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }
}
```

#### JWT安全规则
```
✅ 好的实践：
- 使用强密钥（至少256位）
- 密钥存储在环境变量，不硬编码
- 设置合理的过期时间（不超过24小时）
- 使用HTTPS传输
- 实现Token刷新机制
- 敏感操作需要重新验证

❌ 不好的实践：
- 密钥硬编码在代码中
- 密钥过短（如"secret"）
- 永不过期的Token
- 在Token中存储敏感信息（密码、信用卡号）
- 使用HTTP传输Token
```

### 3.2 Session安全

#### Session配置
```yaml
# application.yml
server:
  servlet:
    session:
      timeout: 30m              # 会话超时时间
      cookie:
        http-only: true         # 防止XSS读取Cookie
        secure: true            # 仅HTTPS传输
        same-site: strict       # 防止CSRF
        name: SESSIONID         # 自定义Cookie名称
```

#### Session管理
```java
@Configuration
public class SessionConfig {

    @Bean
    public SessionRegistry sessionRegistry() {
        return new SessionRegistryImpl();
    }

    @Bean
    public HttpSessionEventPublisher httpSessionEventPublisher() {
        return new HttpSessionEventPublisher();
    }
}

@Service
public class SessionService {

    @Autowired
    private SessionRegistry sessionRegistry;

    /**
     * 限制并发登录
     */
    public void restrictConcurrentSessions(String username, int maxSessions) {
        List<SessionInformation> sessions = sessionRegistry
            .getAllSessions(username, false);

        if (sessions.size() >= maxSessions) {
            // 踢出最早的会话
            sessions.get(0).expireNow();
        }
    }

    /**
     * 强制用户下线
     */
    public void forceLogout(String username) {
        sessionRegistry.getAllSessions(username, false)
            .forEach(SessionInformation::expireNow);
    }
}
```

---

## 4. 输入验证

### 4.1 后端验证

#### Bean Validation
```java
public class UserRequest {

    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度必须在3-20之间")
    @Pattern(regexp = "^[a-zA-Z0-9_]+$", message = "用户名只能包含字母、数字和下划线")
    private String username;

    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;

    @NotBlank(message = "密码不能为空")
    @Size(min = 8, max = 50, message = "密码长度必须在8-50之间")
    @Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d).+$",
        message = "密码必须包含大小写字母和数字"
    )
    private String password;

    @NotNull(message = "年龄不能为空")
    @Min(value = 1, message = "年龄必须大于0")
    @Max(value = 150, message = "年龄必须小于150")
    private Integer age;

    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;
}

@RestController
public class UserController {

    @PostMapping("/users")
    public ResponseEntity<?> createUser(@Valid @RequestBody UserRequest request) {
        // 验证通过，执行业务逻辑
        return ResponseEntity.ok().build();
    }
}
```

### 4.2 输入验证规则

```
✅ 好的实践：
- 白名单验证（只允许已知安全的输入）
- 后端必须验证，前端验证只是辅助
- 验证数据类型、长度、格式、范围
- 拒绝特殊字符（如<>'"等）

❌ 不好的实践：
- 黑名单验证（容易被绕过）
- 只在前端验证
- 直接相信用户输入
- 对输入不做任何验证
```

#### 文件上传验证
```java
@Service
public class FileUploadService {

    private static final List<String> ALLOWED_EXTENSIONS =
        Arrays.asList("jpg", "jpeg", "png", "pdf");
    private static final long MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

    public void validateFile(MultipartFile file) {
        // 验证文件是否为空
        if (file.isEmpty()) {
            throw new IllegalArgumentException("文件不能为空");
        }

        // 验证文件大小
        if (file.getSize() > MAX_FILE_SIZE) {
            throw new IllegalArgumentException("文件大小不能超过10MB");
        }

        // 验证文件扩展名
        String filename = file.getOriginalFilename();
        String extension = filename.substring(filename.lastIndexOf(".") + 1).toLowerCase();
        if (!ALLOWED_EXTENSIONS.contains(extension)) {
            throw new IllegalArgumentException("不支持的文件类型");
        }

        // 验证文件MIME类型
        String contentType = file.getContentType();
        if (!isValidContentType(contentType)) {
            throw new IllegalArgumentException("不支持的文件类型");
        }

        // 验证文件内容（防止文件伪装）
        try {
            String detectedType = Files.probeContentType(file.getResource().getFile().toPath());
            if (!isValidContentType(detectedType)) {
                throw new IllegalArgumentException("文件内容与扩展名不匹配");
            }
        } catch (IOException e) {
            throw new RuntimeException("文件验证失败", e);
        }
    }

    private boolean isValidContentType(String contentType) {
        return contentType != null && (
            contentType.startsWith("image/") ||
            contentType.equals("application/pdf")
        );
    }
}
```

---

## 5. 输出编码

### 5.1 HTML编码

#### 防止XSS
```java
import org.springframework.web.util.HtmlUtils;

public class OutputEncoder {

    /**
     * HTML编码
     */
    public String encodeHtml(String input) {
        return HtmlUtils.htmlEscape(input);
    }

    /**
     * JavaScript编码
     */
    public String encodeJavaScript(String input) {
        return input
            .replace("\\", "\\\\")
            .replace("'", "\\'")
            .replace("\"", "\\\"")
            .replace("\n", "\\n")
            .replace("\r", "\\r")
            .replace("<", "\\x3c")
            .replace(">", "\\x3e");
    }

    /**
     * URL编码
     */
    public String encodeUrl(String input) {
        try {
            return URLEncoder.encode(input, StandardCharsets.UTF_8.toString());
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException(e);
        }
    }
}
```

#### 模板引擎自动编码
```html
<!-- Thymeleaf（自动HTML编码） -->
<p th:text="${userInput}"></p>

<!-- 原始输出（慎用） -->
<p th:utext="${trustedHtml}"></p>

<!-- Vue.js（自动HTML编码） -->
<p>{{ userInput }}</p>

<!-- 原始HTML（慎用） -->
<p v-html="trustedHtml"></p>
```

---

## 6. SQL注入防护

### 6.1 参数化查询

#### MyBatis安全实践
```xml
✅ 好的实践（使用 #{}）：

<!-- 参数化查询，自动防SQL注入 -->
<select id="getUserById" resultType="User">
    SELECT * FROM users WHERE id = #{id}
</select>

<select id="searchUsers" resultType="User">
    SELECT * FROM users
    WHERE username LIKE CONCAT('%', #{keyword}, '%')
    AND status = #{status}
</select>

❌ 不好的实践（使用 ${}）：

<!-- 字符串拼接，易受SQL注入攻击 -->
<select id="getUserById" resultType="User">
    SELECT * FROM users WHERE id = ${id}
</select>

<select id="searchUsers" resultType="User">
    SELECT * FROM users WHERE ${columnName} = #{value}
</select>
```

#### JPA安全实践
```java
✅ 好的实践（参数化查询）：

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // JPQL参数化
    @Query("SELECT u FROM User u WHERE u.username = :username")
    User findByUsername(@Param("username") String username);

    // 方法名查询（自动参数化）
    User findByEmailAndStatus(String email, String status);

    // 原生SQL参数化
    @Query(value = "SELECT * FROM users WHERE email = :email", nativeQuery = true)
    User findByEmailNative(@Param("email") String email);
}

❌ 不好的实践（字符串拼接）：

@Repository
public class UserRepositoryImpl {

    @PersistenceContext
    private EntityManager em;

    // 危险！容易SQL注入
    public User findByUsername(String username) {
        String sql = "SELECT * FROM users WHERE username = '" + username + "'";
        return (User) em.createNativeQuery(sql, User.class).getSingleResult();
    }
}
```

### 6.2 动态SQL安全

#### 安全的动态条件
```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    /**
     * 安全的动态查询
     */
    public List<User> search(UserSearchRequest request) {
        // 使用Specification构建动态查询
        Specification<User> spec = (root, query, cb) -> {
            List<Predicate> predicates = new ArrayList<>();

            // 用户名
            if (StringUtils.hasText(request.getUsername())) {
                predicates.add(cb.like(
                    root.get("username"),
                    "%" + request.getUsername() + "%"
                ));
            }

            // 状态
            if (request.getStatus() != null) {
                predicates.add(cb.equal(
                    root.get("status"),
                    request.getStatus()
                ));
            }

            // 日期范围
            if (request.getStartDate() != null) {
                predicates.add(cb.greaterThanOrEqualTo(
                    root.get("createdAt"),
                    request.getStartDate()
                ));
            }

            if (request.getEndDate() != null) {
                predicates.add(cb.lessThanOrEqualTo(
                    root.get("createdAt"),
                    request.getEndDate()
                ));
            }

            return cb.and(predicates.toArray(new Predicate[0]));
        };

        return userRepository.findAll(spec);
    }
}
```

---

## 7. XSS防护

### 7.1 内容安全策略（CSP）

#### CSP配置
```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.headers()
            .contentSecurityPolicy(
                "default-src 'self'; " +
                "script-src 'self' 'unsafe-inline' https://cdn.example.com; " +
                "style-src 'self' 'unsafe-inline'; " +
                "img-src 'self' data: https:; " +
                "font-src 'self' data:; " +
                "connect-src 'self' https://api.example.com; " +
                "frame-ancestors 'none';"
            );

        return http.build();
    }
}
```

### 7.2 XSS过滤

#### 全局XSS过滤器
```java
import org.jsoup.Jsoup;
import org.jsoup.safety.Safelist;

public class XssFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        XssHttpServletRequestWrapper wrappedRequest =
            new XssHttpServletRequestWrapper((HttpServletRequest) request);

        chain.doFilter(wrappedRequest, response);
    }
}

public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {

    public XssHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    @Override
    public String getParameter(String name) {
        String value = super.getParameter(name);
        return cleanXss(value);
    }

    @Override
    public String[] getParameterValues(String name) {
        String[] values = super.getParameterValues(name);
        if (values == null) {
            return null;
        }
        String[] cleanValues = new String[values.length];
        for (int i = 0; i < values.length; i++) {
            cleanValues[i] = cleanXss(values[i]);
        }
        return cleanValues;
    }

    private String cleanXss(String value) {
        if (value == null) {
            return null;
        }
        // 使用Jsoup清理XSS
        return Jsoup.clean(value, Safelist.basic());
    }
}
```

---

## 8. CSRF防护

### 8.1 Spring Security CSRF

#### 启用CSRF保护
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            .and()
            .authorizeHttpRequests()
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated();

        return http.build();
    }
}
```

#### 前端发送CSRF Token
```javascript
// 获取CSRF Token
function getCsrfToken() {
  const name = 'XSRF-TOKEN';
  const match = document.cookie.match(new RegExp('(^| )' + name + '=([^;]+)'));
  return match ? match[2] : null;
}

// Axios配置
axios.defaults.headers.common['X-XSRF-TOKEN'] = getCsrfToken();

// 发送请求
axios.post('/api/users', userData);
```

### 8.2 双重提交Cookie

#### 自定义CSRF实现
```java
@Component
public class CsrfTokenFilter extends OncePerRequestFilter {

    private static final String CSRF_TOKEN_HEADER = "X-CSRF-TOKEN";
    private static final String CSRF_TOKEN_COOKIE = "CSRF-TOKEN";

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain)
            throws ServletException, IOException {

        // 对于安全方法（GET、HEAD、OPTIONS），不验证CSRF
        if (isSafeMethod(request.getMethod())) {
            chain.doFilter(request, response);
            return;
        }

        // 获取Cookie中的Token
        String cookieToken = getCookieValue(request, CSRF_TOKEN_COOKIE);

        // 获取Header中的Token
        String headerToken = request.getHeader(CSRF_TOKEN_HEADER);

        // 验证Token是否匹配
        if (cookieToken == null || !cookieToken.equals(headerToken)) {
            response.sendError(HttpServletResponse.SC_FORBIDDEN, "Invalid CSRF Token");
            return;
        }

        chain.doFilter(request, response);
    }

    private boolean isSafeMethod(String method) {
        return "GET".equals(method) || "HEAD".equals(method) || "OPTIONS".equals(method);
    }

    private String getCookieValue(HttpServletRequest request, String name) {
        Cookie[] cookies = request.getCookies();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if (name.equals(cookie.getName())) {
                    return cookie.getValue();
                }
            }
        }
        return null;
    }
}
```

---

## 9. 访问控制

### 9.1 基于角色的访问控制（RBAC）

#### 权限注解
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    // 需要ADMIN角色
    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/{id}")
    public ResponseEntity<?> deleteUser(@PathVariable Long id) {
        // 删除用户
        return ResponseEntity.ok().build();
    }

    // 需要USER或ADMIN角色
    @PreAuthorize("hasAnyRole('USER', 'ADMIN')")
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        // 获取用户
        return ResponseEntity.ok(user);
    }

    // 只能访问自己的数据
    @PreAuthorize("#id == authentication.principal.id or hasRole('ADMIN')")
    @PutMapping("/{id}")
    public ResponseEntity<?> updateUser(@PathVariable Long id, @RequestBody UserRequest request) {
        // 更新用户
        return ResponseEntity.ok().build();
    }
}
```

### 9.2 对象级权限控制

#### 自定义权限验证
```java
@Service
public class PermissionService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private OrderRepository orderRepository;

    /**
     * 检查用户是否有权限访问订单
     */
    public boolean canAccessOrder(Authentication auth, Long orderId) {
        User user = (User) auth.getPrincipal();

        // 管理员可以访问所有订单
        if (user.hasRole("ADMIN")) {
            return true;
        }

        // 普通用户只能访问自己的订单
        Order order = orderRepository.findById(orderId).orElse(null);
        return order != null && order.getUserId().equals(user.getId());
    }
}

@RestController
public class OrderController {

    @Autowired
    private PermissionService permissionService;

    @GetMapping("/orders/{id}")
    @PreAuthorize("@permissionService.canAccessOrder(authentication, #id)")
    public ResponseEntity<Order> getOrder(@PathVariable Long id) {
        // 获取订单
        return ResponseEntity.ok(order);
    }
}
```

---

## 10. 敏感数据保护

### 10.1 数据加密

#### 加密敏感字段
```java
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

@Component
public class EncryptionService {

    @Value("${encryption.key}")
    private String encryptionKey;

    private static final String ALGORITHM = "AES";

    /**
     * 加密
     */
    public String encrypt(String plainText) {
        try {
            SecretKeySpec key = new SecretKeySpec(encryptionKey.getBytes(), ALGORITHM);
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, key);
            byte[] encrypted = cipher.doFinal(plainText.getBytes());
            return Base64.getEncoder().encodeToString(encrypted);
        } catch (Exception e) {
            throw new RuntimeException("加密失败", e);
        }
    }

    /**
     * 解密
     */
    public String decrypt(String encryptedText) {
        try {
            SecretKeySpec key = new SecretKeySpec(encryptionKey.getBytes(), ALGORITHM);
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, key);
            byte[] decrypted = cipher.doFinal(Base64.getDecoder().decode(encryptedText));
            return new String(decrypted);
        } catch (Exception e) {
            throw new RuntimeException("解密失败", e);
        }
    }
}

@Entity
public class User {

    @Id
    private Long id;

    private String username;

    // 加密存储手机号
    @Convert(converter = PhoneNumberConverter.class)
    private String phoneNumber;

    // 加密存储身份证号
    @Convert(converter = IdCardConverter.class)
    private String idCard;
}

@Converter
public class PhoneNumberConverter implements AttributeConverter<String, String> {

    @Autowired
    private EncryptionService encryptionService;

    @Override
    public String convertToDatabaseColumn(String attribute) {
        return encryptionService.encrypt(attribute);
    }

    @Override
    public String convertToEntityAttribute(String dbData) {
        return encryptionService.decrypt(dbData);
    }
}
```

### 10.2 敏感数据脱敏

#### 日志脱敏
```java
@Slf4j
public class DataMaskingUtil {

    /**
     * 手机号脱敏：138****1234
     */
    public static String maskPhone(String phone) {
        if (phone == null || phone.length() != 11) {
            return phone;
        }
        return phone.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
    }

    /**
     * 身份证脱敏：110***********1234
     */
    public static String maskIdCard(String idCard) {
        if (idCard == null || idCard.length() < 8) {
            return idCard;
        }
        return idCard.replaceAll("(\\d{3})\\d+(\\d{4})", "$1***********$2");
    }

    /**
     * 邮箱脱敏：abc***@example.com
     */
    public static String maskEmail(String email) {
        if (email == null || !email.contains("@")) {
            return email;
        }
        String[] parts = email.split("@");
        String username = parts[0];
        if (username.length() <= 3) {
            return email;
        }
        return username.substring(0, 3) + "***@" + parts[1];
    }

    /**
     * 银行卡脱敏：1234 **** **** 5678
     */
    public static String maskBankCard(String cardNumber) {
        if (cardNumber == null || cardNumber.length() < 8) {
            return cardNumber;
        }
        return cardNumber.replaceAll("(\\d{4})\\d+(\\d{4})", "$1 **** **** $2");
    }
}

// 使用示例
@Slf4j
@Service
public class UserService {

    public void updatePhone(Long userId, String newPhone) {
        // 记录日志时脱敏
        log.info("用户 {} 更新手机号为: {}", userId, DataMaskingUtil.maskPhone(newPhone));

        // 业务逻辑
    }
}
```

#### API响应脱敏
```java
@JsonSerialize(using = PhoneMaskSerializer.class)
private String phoneNumber;

public class PhoneMaskSerializer extends JsonSerializer<String> {
    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider serializers)
            throws IOException {
        gen.writeString(DataMaskingUtil.maskPhone(value));
    }
}
```

---

## 11. 日志和监控

### 11.1 安全日志

#### 记录关键操作
```java
@Aspect
@Component
@Slf4j
public class SecurityAuditAspect {

    @Autowired
    private AuditLogRepository auditLogRepository;

    /**
     * 记录所有删除操作
     */
    @AfterReturning("@annotation(org.springframework.web.bind.annotation.DeleteMapping)")
    public void logDeleteOperation(JoinPoint joinPoint) {
        String username = SecurityContextHolder.getContext()
            .getAuthentication().getName();

        String operation = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();

        AuditLog log = new AuditLog();
        log.setUsername(username);
        log.setOperation("DELETE");
        log.setTarget(operation);
        log.setDetails(Arrays.toString(args));
        log.setTimestamp(LocalDateTime.now());
        log.setIpAddress(getClientIp());

        auditLogRepository.save(log);

        logger.info("AUDIT: User {} performed DELETE on {} with args {}",
            username, operation, Arrays.toString(args));
    }

    /**
     * 记录登录失败
     */
    public void logFailedLogin(String username, String reason) {
        AuditLog log = new AuditLog();
        log.setUsername(username);
        log.setOperation("LOGIN_FAILED");
        log.setDetails(reason);
        log.setTimestamp(LocalDateTime.now());
        log.setIpAddress(getClientIp());
        log.setLevel("WARNING");

        auditLogRepository.save(log);

        logger.warn("SECURITY: Failed login attempt for user {} - reason: {}",
            username, reason);
    }

    private String getClientIp() {
        ServletRequestAttributes attributes =
            (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        String ip = request.getHeader("X-Forwarded-For");
        if (ip == null || ip.isEmpty()) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}
```

### 11.2 异常监控

#### 全局异常处理
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @Autowired
    private AlertService alertService;

    /**
     * 处理访问拒绝异常
     */
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<?> handleAccessDenied(AccessDeniedException e, HttpServletRequest request) {
        String username = SecurityContextHolder.getContext()
            .getAuthentication().getName();

        log.warn("SECURITY: Access denied for user {} on {}",
            username, request.getRequestURI());

        // 记录安全事件
        securityEventService.recordAccessDenied(username, request.getRequestURI());

        return ResponseEntity.status(HttpStatus.FORBIDDEN)
            .body(Map.of("error", "访问被拒绝"));
    }

    /**
     * 处理SQL异常（可能是SQL注入攻击）
     */
    @ExceptionHandler(SQLException.class)
    public ResponseEntity<?> handleSQLException(SQLException e, HttpServletRequest request) {
        String username = getCurrentUsername();

        log.error("SECURITY: Possible SQL injection detected - User: {}, URI: {}, Error: {}",
            username, request.getRequestURI(), e.getMessage());

        // 发送安全告警
        alertService.sendAlert("可能的SQL注入攻击",
            "用户: " + username + ", URI: " + request.getRequestURI());

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(Map.of("error", "服务器错误"));
    }

    /**
     * 处理认证失败异常
     */
    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<?> handleAuthenticationException(AuthenticationException e) {
        log.warn("SECURITY: Authentication failed - {}", e.getMessage());

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(Map.of("error", "认证失败"));
    }
}
```

---

## 12. 依赖安全

### 12.1 依赖漏洞扫描

#### Maven依赖检查
```xml
<!-- pom.xml -->
<build>
  <plugins>
    <!-- OWASP Dependency Check -->
    <plugin>
      <groupId>org.owasp</groupId>
      <artifactId>dependency-check-maven</artifactId>
      <version>8.4.0</version>
      <executions>
        <execution>
          <goals>
            <goal>check</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>
      </configuration>
    </plugin>
  </plugins>
</build>
```

#### NPM依赖检查
```bash
# 检查已知漏洞
npm audit

# 自动修复
npm audit fix

# 强制修复（可能破坏兼容性）
npm audit fix --force
```

### 12.2 安全依赖管理

```
✅ 好的实践：
- 定期更新依赖版本
- 使用依赖锁定文件（package-lock.json, pom.xml）
- 只使用官方仓库
- 审查新增依赖
- 使用自动化工具扫描漏洞

❌ 不好的实践：
- 长期不更新依赖
- 使用不可信的第三方仓库
- 盲目添加依赖
- 忽略安全警告
```

---

## 总结

### 安全开发检查清单

```yaml
身份认证:
  - [ ] 密码使用BCrypt加密
  - [ ] 密码强度符合要求
  - [ ] 实现防暴力破解机制
  - [ ] 支持多因素认证（MFA）

会话管理:
  - [ ] JWT密钥强度足够
  - [ ] Token有过期时间
  - [ ] Session使用HttpOnly和Secure
  - [ ] 实现会话超时

输入验证:
  - [ ] 所有输入都经过后端验证
  - [ ] 使用白名单验证
  - [ ] 文件上传验证完整
  - [ ] 拒绝特殊字符

输出编码:
  - [ ] 用户输入输出时HTML编码
  - [ ] 模板引擎自动编码
  - [ ] 避免原始HTML输出

SQL注入:
  - [ ] 使用参数化查询
  - [ ] 避免字符串拼接SQL
  - [ ] 动态SQL使用安全构建器

XSS防护:
  - [ ] 配置CSP
  - [ ] 实现XSS过滤器
  - [ ] 输出编码

CSRF防护:
  - [ ] 启用CSRF保护
  - [ ] 验证CSRF Token
  - [ ] 使用SameSite Cookie

访问控制:
  - [ ] 实现RBAC
  - [ ] 对象级权限控制
  - [ ] 最小权限原则

数据保护:
  - [ ] 敏感数据加密存储
  - [ ] 日志脱敏
  - [ ] API响应脱敏
  - [ ] HTTPS传输

日志监控:
  - [ ] 记录关键操作
  - [ ] 记录安全事件
  - [ ] 异常监控告警

依赖安全:
  - [ ] 定期扫描漏洞
  - [ ] 及时更新依赖
  - [ ] 使用官方仓库
```

### 参考资源

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [CWE Top 25](https://cwe.mitre.org/top25/)
- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)

---

**记住**：安全是一个持续的过程，需要在开发的每个阶段都考虑安全问题！
