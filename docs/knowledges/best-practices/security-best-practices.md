# 安全最佳实践

> 本文档总结了Web应用安全的最佳实践，涵盖认证授权、数据加密、常见攻击防范等内容。

## 📋 目录

- [认证和授权](#认证和授权)
- [HTTPS和证书管理](#https和证书管理)
- [密码安全策略](#密码安全策略)
- [数据加密](#数据加密)
- [敏感信息保护](#敏感信息保护)
- [常见攻击防范](#常见攻击防范)
- [安全审计](#安全审计)
- [检查清单](#检查清单)

---

## 认证和授权

### 1. JWT认证

### ✅ 好的实践

```java
// JwtTokenProvider.java
@Component
public class JwtTokenProvider {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private long expiration;

    @Value("${jwt.refresh-expiration}")
    private long refreshExpiration;

    /**
     * 生成访问令牌
     */
    public String generateAccessToken(User user) {
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
     * 生成刷新令牌
     */
    public String generateRefreshToken(User user) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + refreshExpiration);

        return Jwts.builder()
            .setSubject(user.getId().toString())
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(Keys.hmacShaKeyFor(secret.getBytes()), SignatureAlgorithm.HS512)
            .compact();
    }

    /**
     * 验证令牌
     */
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(Keys.hmacShaKeyFor(secret.getBytes()))
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            log.error("Invalid JWT token: {}", e.getMessage());
            return false;
        }
    }

    /**
     * 从令牌中获取用户ID
     */
    public Long getUserIdFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(Keys.hmacShaKeyFor(secret.getBytes()))
            .build()
            .parseClaimsJws(token)
            .getBody();

        return Long.parseLong(claims.getSubject());
    }

    /**
     * 检查令牌是否即将过期
     */
    public boolean isTokenExpiringSoon(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(Keys.hmacShaKeyFor(secret.getBytes()))
            .build()
            .parseClaimsJws(token)
            .getBody();

        Date expiration = claims.getExpiration();
        long timeUntilExpiration = expiration.getTime() - System.currentTimeMillis();

        // 如果5分钟内过期，返回true
        return timeUntilExpiration < 5 * 60 * 1000;
    }
}

// JwtAuthenticationFilter.java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtTokenProvider tokenProvider;

    @Autowired
    private UserDetailsService userDetailsService;

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

                UserDetails userDetails = userDetailsService.loadUserByUserId(userId);

                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );

                authentication.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request)
                );

                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            log.error("Could not set user authentication", e);
        }

        filterChain.doFilter(request, response);
    }

    /**
     * 从请求头中提取Token
     */
    private String getTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");

        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }

        return null;
    }
}

// AuthController.java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private AuthService authService;

    @Autowired
    private JwtTokenProvider tokenProvider;

    /**
     * 登录
     */
    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@Valid @RequestBody LoginRequest request) {
        User user = authService.authenticate(request.getUsername(), request.getPassword());

        String accessToken = tokenProvider.generateAccessToken(user);
        String refreshToken = tokenProvider.generateRefreshToken(user);

        LoginResponse response = new LoginResponse();
        response.setAccessToken(accessToken);
        response.setRefreshToken(refreshToken);
        response.setExpiresIn(3600);  // 1小时

        return ResponseEntity.ok(response);
    }

    /**
     * 刷新令牌
     */
    @PostMapping("/refresh")
    public ResponseEntity<RefreshTokenResponse> refreshToken(
        @Valid @RequestBody RefreshTokenRequest request
    ) {
        String refreshToken = request.getRefreshToken();

        if (!tokenProvider.validateToken(refreshToken)) {
            throw new UnauthorizedException("Invalid refresh token");
        }

        Long userId = tokenProvider.getUserIdFromToken(refreshToken);
        User user = authService.getUserById(userId);

        String newAccessToken = tokenProvider.generateAccessToken(user);

        RefreshTokenResponse response = new RefreshTokenResponse();
        response.setAccessToken(newAccessToken);
        response.setExpiresIn(3600);

        return ResponseEntity.ok(response);
    }

    /**
     * 登出（将令牌加入黑名单）
     */
    @PostMapping("/logout")
    public ResponseEntity<Void> logout(@RequestHeader("Authorization") String authorization) {
        String token = authorization.substring(7);
        authService.revokeToken(token);
        return ResponseEntity.noContent().build();
    }
}
```

**JWT安全配置**：

```yaml
# application.yml
jwt:
  # 密钥（至少256位，建议使用环境变量）
  secret: ${JWT_SECRET:your-256-bit-secret-key-here-must-be-at-least-32-characters}
  # 访问令牌过期时间（1小时）
  expiration: 3600000
  # 刷新令牌过期时间（7天）
  refresh-expiration: 604800000
```

### ❌ 不好的实践

```java
// 不要在JWT中存储敏感信息
public String generateToken(User user) {
    return Jwts.builder()
        .setSubject(user.getId().toString())
        .claim("password", user.getPassword())  // ❌ 不要存储密码
        .claim("ssn", user.getSsn())            // ❌ 不要存储敏感信息
        .signWith(Keys.hmacShaKeyFor(secret.getBytes()))
        .compact();
}

// 不要使用弱密钥
private String secret = "weak-key";  // ❌ 密钥太短

// 不要设置过长的过期时间
private long expiration = 365 * 24 * 60 * 60 * 1000;  // ❌ 1年太长
```

### 2. 基于角色的访问控制（RBAC）

### ✅ 好的实践

```java
// SecurityConfig.java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                // 公开端点
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()

                // 需要认证的端点
                .requestMatchers("/api/users/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers("/api/admin/**").hasRole("ADMIN")

                // 其他请求都需要认证
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

// 使用方法级安全注解
@Service
public class UserService {

    /**
     * 只有ADMIN角色可以访问
     */
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    /**
     * 用户只能访问自己的数据
     */
    @PreAuthorize("#userId == authentication.principal.id")
    public UserVO getUser(Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("用户不存在"));
        return convertToVO(user);
    }

    /**
     * 自定义权限检查
     */
    @PreAuthorize("@permissionEvaluator.canAccessResource(authentication, #resourceId, 'READ')")
    public Resource getResource(Long resourceId) {
        return resourceRepository.findById(resourceId).orElseThrow();
    }
}

// 自定义权限评估器
@Component("permissionEvaluator")
public class CustomPermissionEvaluator {

    @Autowired
    private PermissionService permissionService;

    public boolean canAccessResource(Authentication authentication, Long resourceId, String permission) {
        Long userId = ((UserDetails) authentication.getPrincipal()).getId();
        return permissionService.hasPermission(userId, resourceId, permission);
    }
}
```

---

## HTTPS和证书管理

### 1. HTTPS配置

### ✅ 好的实践

```yaml
# application.yml
server:
  port: 8443
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: ${SSL_KEYSTORE_PASSWORD}
    key-store-type: PKCS12
    key-alias: tomcat
  http2:
    enabled: true

# 强制HTTPS重定向
security:
  require-ssl: true
```

```java
// SecurityConfig.java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // 强制HTTPS
            .requiresChannel(channel -> channel
                .anyRequest().requiresSecure()
            )
            // HSTS（HTTP Strict Transport Security）
            .headers(headers -> headers
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000)  // 1年
                )
            );

        return http.build();
    }
}
```

### 2. 生成自签名证书（开发环境）

```bash
# 生成自签名证书
keytool -genkeypair \
  -alias tomcat \
  -keyalg RSA \
  -keysize 2048 \
  -storetype PKCS12 \
  -keystore keystore.p12 \
  -validity 365 \
  -storepass password
```

### 3. Let's Encrypt证书（生产环境）

```bash
# 安装certbot
sudo apt install certbot

# 获取证书
sudo certbot certonly \
  --standalone \
  -d example.com \
  -d www.example.com

# 证书自动续期
sudo certbot renew --dry-run
```

---

## 密码安全策略

### 1. 密码加密

### ✅ 好的实践

```java
// PasswordService.java
@Service
public class PasswordService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    /**
     * 加密密码
     */
    public String encodePassword(String rawPassword) {
        return passwordEncoder.encode(rawPassword);
    }

    /**
     * 验证密码
     */
    public boolean matches(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }

    /**
     * 检查密码强度
     */
    public boolean isStrongPassword(String password) {
        // 至少8个字符
        if (password.length() < 8) {
            return false;
        }

        // 必须包含大写字母
        if (!password.matches(".*[A-Z].*")) {
            return false;
        }

        // 必须包含小写字母
        if (!password.matches(".*[a-z].*")) {
            return false;
        }

        // 必须包含数字
        if (!password.matches(".*\\d.*")) {
            return false;
        }

        // 必须包含特殊字符
        if (!password.matches(".*[!@#$%^&*()_+\\-=\\[\\]{};':\"\\\\|,.<>/?].*")) {
            return false;
        }

        // 不能包含常见弱密码
        List<String> commonPasswords = Arrays.asList(
            "password", "123456", "12345678", "qwerty", "abc123"
        );
        if (commonPasswords.contains(password.toLowerCase())) {
            return false;
        }

        return true;
    }

    /**
     * 生成随机密码
     */
    public String generateRandomPassword(int length) {
        String chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*";
        SecureRandom random = new SecureRandom();
        StringBuilder password = new StringBuilder();

        for (int i = 0; i < length; i++) {
            password.append(chars.charAt(random.nextInt(chars.length())));
        }

        return password.toString();
    }
}

// CreateUserRequest.java
@Data
public class CreateUserRequest {

    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度必须在3-20之间")
    private String username;

    @NotBlank(message = "密码不能为空")
    @Size(min = 8, max = 20, message = "密码长度必须在8-20之间")
    @Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$",
        message = "密码必须包含大小写字母、数字和特殊字符"
    )
    private String password;
}
```

### ❌ 不好的实践

```java
// 不要使用MD5或SHA1
public String encodePassword(String password) {
    return DigestUtils.md5Hex(password);  // ❌ MD5不安全
}

// 不要明文存储密码
user.setPassword(request.getPassword());  // ❌ 明文存储

// 不要使用弱密码策略
if (password.length() >= 6) {  // ❌ 太短
    // 通过
}
```

### 2. 密码重置

### ✅ 好的实践

```java
// PasswordResetService.java
@Service
public class PasswordResetService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Autowired
    private EmailService emailService;

    /**
     * 发送密码重置邮件
     */
    public void sendPasswordResetEmail(String email) {
        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new ResourceNotFoundException("用户不存在"));

        // 生成随机令牌
        String token = generateResetToken();

        // 存储到Redis（15分钟过期）
        String key = "password_reset:" + token;
        redisTemplate.opsForValue().set(key, user.getId().toString(), Duration.ofMinutes(15));

        // 发送邮件
        String resetUrl = "https://example.com/reset-password?token=" + token;
        emailService.sendPasswordResetEmail(user.getEmail(), resetUrl);
    }

    /**
     * 重置密码
     */
    public void resetPassword(String token, String newPassword) {
        // 验证令牌
        String key = "password_reset:" + token;
        String userIdStr = redisTemplate.opsForValue().get(key);

        if (userIdStr == null) {
            throw new BusinessException("令牌无效或已过期");
        }

        Long userId = Long.parseLong(userIdStr);
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("用户不存在"));

        // 更新密码
        user.setPassword(passwordEncoder.encode(newPassword));
        userRepository.save(user);

        // 删除令牌
        redisTemplate.delete(key);

        // 发送通知邮件
        emailService.sendPasswordChangedNotification(user.getEmail());
    }

    /**
     * 生成重置令牌
     */
    private String generateResetToken() {
        SecureRandom random = new SecureRandom();
        byte[] bytes = new byte[32];
        random.nextBytes(bytes);
        return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
    }
}
```

---

## 数据加密

### 1. 对称加密（AES）

### ✅ 好的实践

```java
// AESEncryptor.java
@Component
public class AESEncryptor {

    private static final String ALGORITHM = "AES/GCM/NoPadding";
    private static final int GCM_IV_LENGTH = 12;
    private static final int GCM_TAG_LENGTH = 16;

    @Value("${encryption.key}")
    private String encryptionKey;

    /**
     * 加密
     */
    public String encrypt(String plainText) {
        try {
            // 生成随机IV
            byte[] iv = new byte[GCM_IV_LENGTH];
            SecureRandom random = new SecureRandom();
            random.nextBytes(iv);

            // 创建密钥
            SecretKeySpec keySpec = new SecretKeySpec(
                encryptionKey.getBytes(StandardCharsets.UTF_8),
                "AES"
            );

            // 创建密码器
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            GCMParameterSpec parameterSpec = new GCMParameterSpec(GCM_TAG_LENGTH * 8, iv);
            cipher.init(Cipher.ENCRYPT_MODE, keySpec, parameterSpec);

            // 加密
            byte[] cipherText = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));

            // 合并IV和密文
            byte[] encrypted = new byte[GCM_IV_LENGTH + cipherText.length];
            System.arraycopy(iv, 0, encrypted, 0, GCM_IV_LENGTH);
            System.arraycopy(cipherText, 0, encrypted, GCM_IV_LENGTH, cipherText.length);

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
            byte[] encrypted = Base64.getDecoder().decode(encryptedText);

            // 分离IV和密文
            byte[] iv = new byte[GCM_IV_LENGTH];
            System.arraycopy(encrypted, 0, iv, 0, GCM_IV_LENGTH);

            byte[] cipherText = new byte[encrypted.length - GCM_IV_LENGTH];
            System.arraycopy(encrypted, GCM_IV_LENGTH, cipherText, 0, cipherText.length);

            // 创建密钥
            SecretKeySpec keySpec = new SecretKeySpec(
                encryptionKey.getBytes(StandardCharsets.UTF_8),
                "AES"
            );

            // 创建密码器
            Cipher cipher = Cipher.getInstance(ALGORITHM);
            GCMParameterSpec parameterSpec = new GCMParameterSpec(GCM_TAG_LENGTH * 8, iv);
            cipher.init(Cipher.DECRYPT_MODE, keySpec, parameterSpec);

            // 解密
            byte[] plainText = cipher.doFinal(cipherText);

            return new String(plainText, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new RuntimeException("解密失败", e);
        }
    }
}

// 使用
@Service
public class UserService {

    @Autowired
    private AESEncryptor encryptor;

    public void createUser(CreateUserRequest request) {
        User user = new User();
        user.setUsername(request.getUsername());

        // 加密敏感信息
        if (request.getSsn() != null) {
            user.setSsn(encryptor.encrypt(request.getSsn()));
        }

        userRepository.save(user);
    }

    public UserVO getUser(Long id) {
        User user = userRepository.findById(id).orElseThrow();

        UserVO vo = new UserVO();
        vo.setId(user.getId());
        vo.setUsername(user.getUsername());

        // 解密敏感信息
        if (user.getSsn() != null) {
            vo.setSsn(encryptor.decrypt(user.getSsn()));
        }

        return vo;
    }
}
```

### 2. 非对称加密（RSA）

### ✅ 好的实践

```java
// RSAEncryptor.java
@Component
public class RSAEncryptor {

    private static final String ALGORITHM = "RSA/ECB/OAEPWithSHA-256AndMGF1Padding";

    @Value("${rsa.public-key}")
    private String publicKeyStr;

    @Value("${rsa.private-key}")
    private String privateKeyStr;

    /**
     * 使用公钥加密
     */
    public String encrypt(String plainText) {
        try {
            PublicKey publicKey = getPublicKey();

            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);

            byte[] encrypted = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));

            return Base64.getEncoder().encodeToString(encrypted);
        } catch (Exception e) {
            throw new RuntimeException("加密失败", e);
        }
    }

    /**
     * 使用私钥解密
     */
    public String decrypt(String encryptedText) {
        try {
            PrivateKey privateKey = getPrivateKey();

            Cipher cipher = Cipher.getInstance(ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, privateKey);

            byte[] decrypted = cipher.doFinal(Base64.getDecoder().decode(encryptedText));

            return new String(decrypted, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new RuntimeException("解密失败", e);
        }
    }

    private PublicKey getPublicKey() throws Exception {
        byte[] keyBytes = Base64.getDecoder().decode(publicKeyStr);
        X509EncodedKeySpec spec = new X509EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        return keyFactory.generatePublic(spec);
    }

    private PrivateKey getPrivateKey() throws Exception {
        byte[] keyBytes = Base64.getDecoder().decode(privateKeyStr);
        PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        return keyFactory.generatePrivate(spec);
    }
}
```

---

## 敏感信息保护

### 1. 配置文件加密

### ✅ 好的实践

```yaml
# application.yml
spring:
  datasource:
    # 使用环境变量
    url: jdbc:mysql://${DB_HOST:localhost}:3306/${DB_NAME:mydb}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

jwt:
  secret: ${JWT_SECRET}

encryption:
  key: ${ENCRYPTION_KEY}
```

```bash
# .env文件（不要提交到Git）
DB_HOST=localhost
DB_NAME=mydb
DB_USERNAME=root
DB_PASSWORD=secret_password
JWT_SECRET=your-256-bit-secret
ENCRYPTION_KEY=your-encryption-key
```

```properties
# .gitignore
.env
application-prod.yml
*.p12
*.jks
```

### 2. 响应中隐藏敏感字段

### ✅ 好的实践

```java
// UserVO.java
@Data
public class UserVO {
    private Long id;
    private String username;
    private String email;

    // 不返回密码
    @JsonIgnore
    private String password;

    // 只写不读
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    private String ssn;

    // 脱敏处理
    @JsonSerialize(using = PhoneMaskSerializer.class)
    private String phone;

    @JsonSerialize(using = EmailMaskSerializer.class)
    private String emailAddress;
}

// PhoneMaskSerializer.java
public class PhoneMaskSerializer extends JsonSerializer<String> {

    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider serializers)
        throws IOException {
        if (value == null || value.length() < 11) {
            gen.writeString(value);
            return;
        }

        // 脱敏：138****5678
        String masked = value.substring(0, 3) + "****" + value.substring(7);
        gen.writeString(masked);
    }
}

// EmailMaskSerializer.java
public class EmailMaskSerializer extends JsonSerializer<String> {

    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider serializers)
        throws IOException {
        if (value == null || !value.contains("@")) {
            gen.writeString(value);
            return;
        }

        String[] parts = value.split("@");
        String username = parts[0];
        String domain = parts[1];

        // 脱敏：t***t@example.com
        String masked = username.charAt(0) + "***" + username.charAt(username.length() - 1) + "@" + domain;
        gen.writeString(masked);
    }
}
```

### 3. 日志脱敏

### ✅ 好的实践

```java
// LogMaskingConverter.java
public class LogMaskingConverter extends MessageConverter {

    private static final Pattern PHONE_PATTERN = Pattern.compile("(\\d{3})\\d{4}(\\d{4})");
    private static final Pattern EMAIL_PATTERN = Pattern.compile("([a-zA-Z0-9])([a-zA-Z0-9.-]+)@");
    private static final Pattern PASSWORD_PATTERN = Pattern.compile("(password|pwd)=([^&\\s]+)");

    @Override
    public String convert(ILoggingEvent event) {
        String message = event.getFormattedMessage();

        // 脱敏手机号
        message = PHONE_PATTERN.matcher(message).replaceAll("$1****$2");

        // 脱敏邮箱
        message = EMAIL_PATTERN.matcher(message).replaceAll("$1***@");

        // 脱敏密码
        message = PASSWORD_PATTERN.matcher(message).replaceAll("$1=***");

        return message;
    }
}
```

```xml
<!-- logback-spring.xml -->
<configuration>
    <conversionRule conversionWord="msg"
                    converterClass="com.example.LogMaskingConverter" />

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

---

## 常见攻击防范

### 1. SQL注入防范

### ✅ 好的实践

```java
// 使用参数化查询（JPA）
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // ✅ 使用参数化查询
    @Query("SELECT u FROM User u WHERE u.username = :username")
    Optional<User> findByUsername(@Param("username") String username);

    // ✅ 使用方法名查询
    Optional<User> findByEmail(String email);
}

// 使用MyBatis参数化查询
@Mapper
public interface UserMapper {

    // ✅ 使用#{}参数化
    @Select("SELECT * FROM users WHERE username = #{username}")
    User findByUsername(@Param("username") String username);

    // ❌ 不要使用${}拼接
    // @Select("SELECT * FROM users WHERE username = '${username}'")
    // User findByUsernameBad(@Param("username") String username);
}
```

### ❌ 不好的实践

```java
// 字符串拼接（SQL注入风险）
String sql = "SELECT * FROM users WHERE username = '" + username + "'";
// 如果username = "admin' OR '1'='1"，则SQL变为：
// SELECT * FROM users WHERE username = 'admin' OR '1'='1'
```

### 2. XSS（跨站脚本）防范

### ✅ 好的实践

```java
// XSSFilter.java
@Component
@WebFilter(urlPatterns = "/*")
public class XSSFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
        chain.doFilter(new XSSRequestWrapper((HttpServletRequest) request), response);
    }
}

// XSSRequestWrapper.java
public class XSSRequestWrapper extends HttpServletRequestWrapper {

    public XSSRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    @Override
    public String getParameter(String name) {
        String value = super.getParameter(name);
        return cleanXSS(value);
    }

    @Override
    public String[] getParameterValues(String name) {
        String[] values = super.getParameterValues(name);
        if (values == null) {
            return null;
        }

        String[] cleanValues = new String[values.length];
        for (int i = 0; i < values.length; i++) {
            cleanValues[i] = cleanXSS(values[i]);
        }

        return cleanValues;
    }

    private String cleanXSS(String value) {
        if (value == null) {
            return null;
        }

        // 移除script标签
        value = value.replaceAll("<script>(.*?)</script>", "");

        // HTML实体编码
        value = StringEscapeUtils.escapeHtml4(value);

        return value;
    }
}
```

```vue
<!-- 前端XSS防范 -->
<template>
  <!-- ✅ Vue会自动转义 -->
  <div>{{ userInput }}</div>

  <!-- ✅ 使用v-text -->
  <div v-text="userInput"></div>

  <!-- ❌ 不要使用v-html（除非内容可信） -->
  <!-- <div v-html="userInput"></div> -->

  <!-- ✅ 如果必须使用v-html，先sanitize -->
  <div v-html="sanitizedHtml"></div>
</template>

<script setup lang="ts">
import DOMPurify from 'dompurify'

const userInput = ref('<script>alert("XSS")</script>')

// 使用DOMPurify清理HTML
const sanitizedHtml = computed(() => {
  return DOMPurify.sanitize(userInput.value)
})
</script>
```

### 3. CSRF（跨站请求伪造）防范

### ✅ 好的实践

```java
// SecurityConfig.java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // 启用CSRF保护
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .ignoringRequestMatchers("/api/public/**")  // 公开API不需要CSRF
            );

        return http.build();
    }
}
```

```typescript
// 前端添加CSRF Token
import axios from 'axios'

// 从Cookie中获取CSRF Token
function getCsrfToken(): string {
  const name = 'XSRF-TOKEN='
  const cookies = document.cookie.split(';')

  for (let cookie of cookies) {
    cookie = cookie.trim()
    if (cookie.indexOf(name) === 0) {
      return cookie.substring(name.length)
    }
  }

  return ''
}

// 请求拦截器：添加CSRF Token
axios.interceptors.request.use(config => {
  const token = getCsrfToken()
  if (token) {
    config.headers['X-XSRF-TOKEN'] = token
  }
  return config
})
```

### 4. 文件上传安全

### ✅ 好的实践

```java
// FileUploadService.java
@Service
public class FileUploadService {

    private static final List<String> ALLOWED_EXTENSIONS = Arrays.asList(
        "jpg", "jpeg", "png", "gif", "pdf", "doc", "docx", "xls", "xlsx"
    );

    private static final long MAX_FILE_SIZE = 10 * 1024 * 1024;  // 10MB

    @Value("${file.upload-dir}")
    private String uploadDir;

    /**
     * 上传文件
     */
    public String uploadFile(MultipartFile file) {
        // 1. 检查文件是否为空
        if (file.isEmpty()) {
            throw new BusinessException("文件不能为空");
        }

        // 2. 检查文件大小
        if (file.getSize() > MAX_FILE_SIZE) {
            throw new BusinessException("文件大小不能超过10MB");
        }

        // 3. 检查文件扩展名
        String originalFilename = file.getOriginalFilename();
        String extension = getFileExtension(originalFilename);

        if (!ALLOWED_EXTENSIONS.contains(extension.toLowerCase())) {
            throw new BusinessException("不支持的文件类型");
        }

        // 4. 检查文件内容类型
        String contentType = file.getContentType();
        if (!isValidContentType(contentType)) {
            throw new BusinessException("文件内容类型不合法");
        }

        // 5. 生成新文件名（避免文件名冲突和路径遍历）
        String newFilename = generateFilename(extension);

        // 6. 保存文件
        try {
            Path targetPath = Paths.get(uploadDir, newFilename);
            Files.copy(file.getInputStream(), targetPath, StandardCopyOption.REPLACE_EXISTING);

            return newFilename;
        } catch (IOException e) {
            throw new RuntimeException("文件上传失败", e);
        }
    }

    private String getFileExtension(String filename) {
        int lastDotIndex = filename.lastIndexOf('.');
        if (lastDotIndex == -1) {
            return "";
        }
        return filename.substring(lastDotIndex + 1);
    }

    private boolean isValidContentType(String contentType) {
        List<String> allowedTypes = Arrays.asList(
            "image/jpeg", "image/png", "image/gif",
            "application/pdf",
            "application/msword",
            "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        );
        return allowedTypes.contains(contentType);
    }

    private String generateFilename(String extension) {
        String uuid = UUID.randomUUID().toString();
        return uuid + "." + extension;
    }
}
```

---

## 安全审计

### 1. 操作日志

### ✅ 好的实践

```java
// AuditLog.java
@Entity
@Table(name = "audit_logs")
public class AuditLog {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long userId;
    private String username;
    private String action;  // CREATE, UPDATE, DELETE
    private String resource;  // User, Order, etc.
    private Long resourceId;
    private String ipAddress;
    private String userAgent;

    @Column(columnDefinition = "JSON")
    private String details;

    private LocalDateTime createdAt;
}

// AuditLogService.java
@Service
public class AuditLogService {

    @Autowired
    private AuditLogRepository auditLogRepository;

    public void log(String action, String resource, Long resourceId, String details) {
        HttpServletRequest request = ((ServletRequestAttributes)
            RequestContextHolder.currentRequestAttributes()).getRequest();

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        AuditLog log = new AuditLog();
        log.setUserId(getUserId(authentication));
        log.setUsername(getUsername(authentication));
        log.setAction(action);
        log.setResource(resource);
        log.setResourceId(resourceId);
        log.setIpAddress(getClientIP(request));
        log.setUserAgent(request.getHeader("User-Agent"));
        log.setDetails(details);
        log.setCreatedAt(LocalDateTime.now());

        auditLogRepository.save(log);
    }

    private String getClientIP(HttpServletRequest request) {
        String ip = request.getHeader("X-Forwarded-For");
        if (ip == null || ip.isEmpty()) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}

// 使用
@Service
public class UserService {

    @Autowired
    private AuditLogService auditLogService;

    public void deleteUser(Long id) {
        User user = userRepository.findById(id).orElseThrow();

        userRepository.deleteById(id);

        // 记录审计日志
        auditLogService.log("DELETE", "User", id, "删除用户: " + user.getUsername());
    }
}
```

### 2. 登录失败监控

### ✅ 好的实践

```java
// LoginAttemptService.java
@Service
public class LoginAttemptService {

    private static final int MAX_ATTEMPTS = 5;
    private static final int LOCK_TIME_MINUTES = 15;

    @Autowired
    private RedisTemplate<String, Integer> redisTemplate;

    /**
     * 登录失败
     */
    public void loginFailed(String username) {
        String key = "login_attempts:" + username;
        Integer attempts = redisTemplate.opsForValue().get(key);

        if (attempts == null) {
            attempts = 0;
        }

        attempts++;
        redisTemplate.opsForValue().set(key, attempts, Duration.ofMinutes(LOCK_TIME_MINUTES));

        if (attempts >= MAX_ATTEMPTS) {
            // 锁定账户
            lockAccount(username);
        }
    }

    /**
     * 登录成功
     */
    public void loginSucceeded(String username) {
        String key = "login_attempts:" + username;
        redisTemplate.delete(key);
    }

    /**
     * 检查是否被锁定
     */
    public boolean isLocked(String username) {
        String key = "login_attempts:" + username;
        Integer attempts = redisTemplate.opsForValue().get(key);

        return attempts != null && attempts >= MAX_ATTEMPTS;
    }

    private void lockAccount(String username) {
        log.warn("账户被锁定: {}", username);
        // 发送告警邮件
    }
}
```

---

## 检查清单

```yaml
认证授权:
  - [ ] 使用JWT进行认证
  - [ ] Token过期时间合理（访问令牌1小时，刷新令牌7天）
  - [ ] 实现Token刷新机制
  - [ ] 实现基于角色的访问控制（RBAC）
  - [ ] 使用方法级安全注解（@PreAuthorize）

HTTPS:
  - [ ] 生产环境强制使用HTTPS
  - [ ] 配置HSTS
  - [ ] 使用有效的SSL证书
  - [ ] 定期更新证书

密码安全:
  - [ ] 使用BCrypt加密密码
  - [ ] 密码强度验证（大小写、数字、特殊字符）
  - [ ] 密码最小长度8位
  - [ ] 实现密码重置功能
  - [ ] 密码重置令牌过期时间（15分钟）

数据加密:
  - [ ] 敏感数据使用AES加密
  - [ ] 加密密钥存储在环境变量中
  - [ ] 使用安全的加密算法（AES-GCM）

敏感信息保护:
  - [ ] 配置文件使用环境变量
  - [ ] 敏感文件不提交到Git（.env, *.p12）
  - [ ] 响应中隐藏敏感字段（@JsonIgnore）
  - [ ] 日志脱敏
  - [ ] 手机号、邮箱脱敏

攻击防范:
  - [ ] 使用参数化查询防范SQL注入
  - [ ] XSS过滤
  - [ ] CSRF保护
  - [ ] 文件上传大小限制
  - [ ] 文件扩展名白名单
  - [ ] 限流防DDoS

安全审计:
  - [ ] 记录操作日志
  - [ ] 登录失败监控
  - [ ] 异常登录告警
  - [ ] 定期安全审计
```

---

## 总结

1. **认证授权**：使用JWT + RBAC，Token过期时间合理
2. **HTTPS**：生产环境强制HTTPS，配置HSTS
3. **密码安全**：BCrypt加密，密码强度验证
4. **数据加密**：敏感数据AES加密，密钥环境变量存储
5. **敏感信息保护**：配置加密、日志脱敏、响应脱敏
6. **攻击防范**：SQL注入、XSS、CSRF、文件上传安全
7. **安全审计**：操作日志、登录监控、异常告警
