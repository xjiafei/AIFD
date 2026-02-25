# 性能优化最佳实践

> 本文档总结了前端、后端、数据库和网络的性能优化最佳实践，帮助构建高性能的Web应用。

## 📋 目录

- [前端性能优化](#前端性能优化)
- [后端性能优化](#后端性能优化)
- [数据库性能优化](#数据库性能优化)
- [网络性能优化](#网络性能优化)
- [性能监控和分析](#性能监控和分析)
- [检查清单](#检查清单)

---

## 前端性能优化

### 1. 加载性能优化

#### 代码分割（Code Splitting）

### ✅ 好的实践

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  build: {
    rollupOptions: {
      output: {
        // 手动分割代码
        manualChunks: {
          // 框架代码
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          // UI库
          'ui-vendor': ['element-plus'],
          // 工具库
          'utils-vendor': ['axios', 'dayjs']
        }
      }
    },
    // 代码分割阈值
    chunkSizeWarningLimit: 500
  }
})
```

```typescript
// 路由懒加载
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      component: () => import('@/views/Home.vue')  // 懒加载
    },
    {
      path: '/users',
      component: () => import('@/views/Users.vue')  // 懒加载
    },
    {
      path: '/dashboard',
      component: () => import('@/views/Dashboard.vue')  // 懒加载
    }
  ]
})
```

#### 资源压缩

### ✅ 好的实践

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import viteCompression from 'vite-plugin-compression'

export default defineConfig({
  plugins: [
    // Gzip压缩
    viteCompression({
      algorithm: 'gzip',
      ext: '.gz',
      threshold: 10240,  // 大于10KB的文件才压缩
      deleteOriginFile: false
    }),
    // Brotli压缩（更高压缩率）
    viteCompression({
      algorithm: 'brotliCompress',
      ext: '.br',
      threshold: 10240,
      deleteOriginFile: false
    })
  ],
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,  // 移除console
        drop_debugger: true  // 移除debugger
      }
    }
  }
})
```

#### 图片优化

### ✅ 好的实践

```vue
<template>
  <!-- 1. 使用WebP格式 -->
  <picture>
    <source srcset="/images/photo.webp" type="image/webp">
    <source srcset="/images/photo.jpg" type="image/jpeg">
    <img src="/images/photo.jpg" alt="Photo">
  </picture>

  <!-- 2. 响应式图片 -->
  <img
    srcset="/images/photo-320w.jpg 320w,
            /images/photo-640w.jpg 640w,
            /images/photo-1280w.jpg 1280w"
    sizes="(max-width: 320px) 280px,
           (max-width: 640px) 600px,
           1280px"
    src="/images/photo-640w.jpg"
    alt="Responsive photo"
  >

  <!-- 3. 懒加载图片 -->
  <img
    src="/images/placeholder.jpg"
    data-src="/images/real-image.jpg"
    loading="lazy"
    alt="Lazy loaded image"
  >

  <!-- 4. 使用CSS Sprites（小图标） -->
  <i class="icon icon-user"></i>
</template>

<style scoped>
.icon {
  background-image: url('/images/sprites.png');
  width: 24px;
  height: 24px;
  display: inline-block;
}

.icon-user {
  background-position: 0 0;
}
</style>
```

```typescript
// 图片懒加载实现
import { onMounted } from 'vue'

export function useLazyLoad() {
  onMounted(() => {
    const images = document.querySelectorAll('img[data-src]')

    const imageObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target as HTMLImageElement
          img.src = img.dataset.src!
          img.removeAttribute('data-src')
          imageObserver.unobserve(img)
        }
      })
    })

    images.forEach(img => imageObserver.observe(img))
  })
}
```

### 2. 渲染性能优化

#### 虚拟列表

### ✅ 好的实践

```vue
<!-- VirtualList.vue -->
<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

interface Props {
  items: any[]
  itemHeight: number
  visibleCount?: number
}

const props = withDefaults(defineProps<Props>(), {
  visibleCount: 10
})

const scrollTop = ref(0)
const containerRef = ref<HTMLElement>()

// 可见区域的起始索引
const startIndex = computed(() => {
  return Math.floor(scrollTop.value / props.itemHeight)
})

// 可见区域的结束索引
const endIndex = computed(() => {
  return startIndex.value + props.visibleCount
})

// 可见的数据
const visibleItems = computed(() => {
  return props.items.slice(startIndex.value, endIndex.value)
})

// 总高度
const totalHeight = computed(() => {
  return props.items.length * props.itemHeight
})

// 偏移量
const offsetY = computed(() => {
  return startIndex.value * props.itemHeight
})

const handleScroll = (e: Event) => {
  scrollTop.value = (e.target as HTMLElement).scrollTop
}

onMounted(() => {
  containerRef.value?.addEventListener('scroll', handleScroll)
})

onUnmounted(() => {
  containerRef.value?.removeEventListener('scroll', handleScroll)
})
</script>

<template>
  <div ref="containerRef" class="virtual-list" :style="{ height: '500px', overflow: 'auto' }">
    <div class="virtual-list-phantom" :style="{ height: totalHeight + 'px' }"></div>
    <div class="virtual-list-content" :style="{ transform: `translateY(${offsetY}px)` }">
      <div
        v-for="(item, index) in visibleItems"
        :key="startIndex + index"
        class="virtual-list-item"
        :style="{ height: itemHeight + 'px' }"
      >
        <slot :item="item" :index="startIndex + index"></slot>
      </div>
    </div>
  </div>
</template>

<style scoped>
.virtual-list {
  position: relative;
}

.virtual-list-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.virtual-list-content {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
}
</style>
```

```vue
<!-- 使用虚拟列表 -->
<template>
  <VirtualList
    :items="users"
    :item-height="50"
    :visible-count="20"
  >
    <template #default="{ item }">
      <div class="user-item">
        <span>{{ item.name }}</span>
        <span>{{ item.email }}</span>
      </div>
    </template>
  </VirtualList>
</template>
```

#### 防抖和节流

### ✅ 好的实践

```typescript
// utils/performance.ts

/**
 * 防抖：延迟执行，多次触发只执行最后一次
 * 适用场景：搜索框输入、窗口resize
 */
export function debounce<T extends (...args: any[]) => any>(
  func: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: ReturnType<typeof setTimeout>

  return function (...args: Parameters<T>) {
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => {
      func(...args)
    }, delay)
  }
}

/**
 * 节流：固定时间间隔执行
 * 适用场景：滚动事件、按钮点击
 */
export function throttle<T extends (...args: any[]) => any>(
  func: T,
  delay: number
): (...args: Parameters<T>) => void {
  let lastTime = 0

  return function (...args: Parameters<T>) {
    const now = Date.now()
    if (now - lastTime >= delay) {
      func(...args)
      lastTime = now
    }
  }
}
```

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { debounce, throttle } from '@/utils/performance'

const searchQuery = ref('')

// 搜索（防抖）
const handleSearch = debounce((query: string) => {
  console.log('搜索:', query)
  // 调用API
}, 300)

// 滚动（节流）
const handleScroll = throttle(() => {
  console.log('滚动位置:', window.scrollY)
}, 100)
</script>

<template>
  <input
    v-model="searchQuery"
    @input="handleSearch(searchQuery)"
    placeholder="搜索..."
  >
</template>
```

#### 组件懒加载

### ✅ 好的实践

```vue
<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

// 异步组件
const HeavyComponent = defineAsyncComponent(() =>
  import('@/components/HeavyComponent.vue')
)

// 带加载状态的异步组件
const AnotherHeavyComponent = defineAsyncComponent({
  loader: () => import('@/components/AnotherHeavyComponent.vue'),
  loadingComponent: () => import('@/components/Loading.vue'),
  errorComponent: () => import('@/components/Error.vue'),
  delay: 200,
  timeout: 3000
})
</script>

<template>
  <Suspense>
    <template #default>
      <HeavyComponent />
    </template>
    <template #fallback>
      <div>加载中...</div>
    </template>
  </Suspense>
</template>
```

### 3. 交互性能优化

#### 长列表优化

### ✅ 好的实践

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const users = ref<User[]>([])
const currentPage = ref(1)
const pageSize = 20

// 分页加载（而非一次加载所有数据）
const visibleUsers = computed(() => {
  const start = (currentPage.value - 1) * pageSize
  const end = start + pageSize
  return users.value.slice(start, end)
})

// 无限滚动
const loadMore = () => {
  // 加载下一页
  currentPage.value++
}
</script>

<template>
  <div class="user-list">
    <div v-for="user in visibleUsers" :key="user.id" class="user-item">
      {{ user.name }}
    </div>

    <div ref="sentinel" @intersect="loadMore"></div>
  </div>
</template>
```

#### 避免不必要的渲染

### ✅ 好的实践

```vue
<script setup lang="ts">
import { ref, computed, shallowRef } from 'vue'

// 1. 使用computed缓存计算结果
const users = ref<User[]>([])
const activeUsers = computed(() => {
  return users.value.filter(u => u.isActive)
})

// 2. 使用shallowRef（对象内部变化不触发更新）
const config = shallowRef({
  theme: 'dark',
  fontSize: 14
})

// 3. 使用v-memo（Vue 3.2+）
// 只有dependencies变化时才重新渲染
</script>

<template>
  <!-- v-memo: 只有user.id变化时才重新渲染 -->
  <div v-for="user in users" :key="user.id" v-memo="[user.id]">
    <UserCard :user="user" />
  </div>

  <!-- v-once: 只渲染一次 -->
  <div v-once>
    <h1>{{ staticTitle }}</h1>
  </div>
</template>
```

---

## 后端性能优化

### 1. 缓存策略

#### Redis缓存

### ✅ 好的实践

```java
// CacheConfig.java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        // 默认配置
        RedisCacheConfiguration defaultConfig = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))  // 默认10分钟过期
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();

        // 自定义不同缓存的过期时间
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();

        // 用户缓存：1小时
        cacheConfigurations.put("users",
            defaultConfig.entryTtl(Duration.ofHours(1)));

        // 热点数据缓存：10分钟
        cacheConfigurations.put("hotData",
            defaultConfig.entryTtl(Duration.ofMinutes(10)));

        // 配置缓存：1天
        cacheConfigurations.put("config",
            defaultConfig.entryTtl(Duration.ofDays(1)));

        return RedisCacheManager.builder(factory)
            .cacheDefaults(defaultConfig)
            .withInitialCacheConfigurations(cacheConfigurations)
            .transactionAware()
            .build();
    }
}

// UserService.java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    /**
     * 查询用户（带缓存）
     * key: users::用户ID
     * 过期时间: 1小时
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
     * 批量清除缓存
     */
    @CacheEvict(value = "users", allEntries = true)
    public void clearAllUserCache() {
        log.info("清除所有用户缓存");
    }
}
```

#### 本地缓存（Caffeine）

### ✅ 好的实践

```java
// LocalCacheConfig.java
@Configuration
public class LocalCacheConfig {

    /**
     * 配置本地缓存
     * 适用场景：高频访问、数据量小、一致性要求不高
     */
    @Bean
    public CacheManager caffeineCacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();

        List<CaffeineCache> caches = Arrays.asList(
            // 配置缓存：最大1000条，1小时过期
            buildCache("config", 1000, 60),
            // 字典缓存：最大5000条，30分钟过期
            buildCache("dict", 5000, 30),
            // 权限缓存：最大10000条，10分钟过期
            buildCache("permission", 10000, 10)
        );

        cacheManager.setCaches(caches);
        return cacheManager;
    }

    private CaffeineCache buildCache(String name, int maxSize, int expireMinutes) {
        return new CaffeineCache(
            name,
            Caffeine.newBuilder()
                .maximumSize(maxSize)
                .expireAfterWrite(Duration.ofMinutes(expireMinutes))
                .recordStats()  // 记录统计信息
                .build()
        );
    }
}
```

#### 多级缓存

### ✅ 好的实践

```java
// MultiLevelCacheService.java
@Service
public class MultiLevelCacheService {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    private final Cache<String, Object> localCache = Caffeine.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(Duration.ofMinutes(5))
        .build();

    /**
     * 多级缓存查询
     * L1: 本地缓存（Caffeine）
     * L2: 分布式缓存（Redis）
     * L3: 数据库
     */
    public <T> T get(String key, Class<T> type, Supplier<T> loader) {
        // L1: 本地缓存
        Object value = localCache.getIfPresent(key);
        if (value != null) {
            log.debug("从本地缓存获取: {}", key);
            return type.cast(value);
        }

        // L2: Redis缓存
        value = redisTemplate.opsForValue().get(key);
        if (value != null) {
            log.debug("从Redis获取: {}", key);
            localCache.put(key, value);  // 回填本地缓存
            return type.cast(value);
        }

        // L3: 数据库
        log.debug("从数据库获取: {}", key);
        T data = loader.get();
        if (data != null) {
            // 写入Redis
            redisTemplate.opsForValue().set(key, data, Duration.ofHours(1));
            // 写入本地缓存
            localCache.put(key, data);
        }

        return data;
    }

    /**
     * 清除缓存
     */
    public void evict(String key) {
        localCache.invalidate(key);
        redisTemplate.delete(key);
    }
}
```

### 2. 数据库查询优化

#### 批量查询

### ✅ 好的实践

```java
// 避免N+1查询
@Service
public class OrderService {

    /**
     * ❌ 不好的实践：N+1查询
     */
    public List<OrderVO> getOrders_Bad() {
        List<Order> orders = orderRepository.findAll();

        return orders.stream()
            .map(order -> {
                // 每个订单都会查询一次用户（N+1问题）
                User user = userRepository.findById(order.getUserId()).orElse(null);

                OrderVO vo = new OrderVO();
                vo.setId(order.getId());
                vo.setUserName(user != null ? user.getUsername() : null);
                return vo;
            })
            .collect(Collectors.toList());
    }

    /**
     * ✅ 好的实践：批量查询
     */
    public List<OrderVO> getOrders_Good() {
        List<Order> orders = orderRepository.findAll();

        // 批量查询用户
        Set<Long> userIds = orders.stream()
            .map(Order::getUserId)
            .collect(Collectors.toSet());

        Map<Long, User> userMap = userRepository.findAllById(userIds)
            .stream()
            .collect(Collectors.toMap(User::getId, Function.identity()));

        return orders.stream()
            .map(order -> {
                User user = userMap.get(order.getUserId());

                OrderVO vo = new OrderVO();
                vo.setId(order.getId());
                vo.setUserName(user != null ? user.getUsername() : null);
                return vo;
            })
            .collect(Collectors.toList());
    }
}
```

#### 分页查询优化

### ✅ 好的实践

```java
@Service
public class ProductService {

    /**
     * ❌ 不好的实践：大offset分页
     */
    public Page<Product> getProducts_Bad(int page, int size) {
        // page=1000, size=20
        // MySQL需要扫描1000*20=20000条记录，然后丢弃前19980条
        return productRepository.findAll(PageRequest.of(page, size));
    }

    /**
     * ✅ 好的实践：游标分页
     */
    public List<Product> getProducts_Good(Long lastId, int size) {
        // 使用上一页最后一条记录的ID
        return productRepository.findByIdGreaterThan(lastId, PageRequest.of(0, size));
    }
}

// ProductRepository.java
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Query("SELECT p FROM Product p WHERE p.id > :lastId ORDER BY p.id ASC")
    List<Product> findByIdGreaterThan(@Param("lastId") Long lastId, Pageable pageable);
}
```

### 3. 异步处理

#### CompletableFuture

### ✅ 好的实践

```java
@Service
public class OrderService {

    @Autowired
    private UserService userService;

    @Autowired
    private ProductService productService;

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private Executor asyncExecutor;

    /**
     * ❌ 不好的实践：串行执行
     */
    public OrderDetailVO getOrderDetail_Bad(Long orderId) {
        Order order = orderRepository.findById(orderId).orElseThrow();

        // 串行执行，总耗时 = 耗时1 + 耗时2 + 耗时3
        User user = userService.getUser(order.getUserId());  // 耗时100ms
        Product product = productService.getProduct(order.getProductId());  // 耗时100ms
        Payment payment = paymentService.getPayment(order.getPaymentId());  // 耗时100ms

        return buildOrderDetail(order, user, product, payment);
    }

    /**
     * ✅ 好的实践：并行执行
     */
    public OrderDetailVO getOrderDetail_Good(Long orderId) {
        Order order = orderRepository.findById(orderId).orElseThrow();

        // 并行执行，总耗时 = max(耗时1, 耗时2, 耗时3) ≈ 100ms
        CompletableFuture<User> userFuture = CompletableFuture
            .supplyAsync(() -> userService.getUser(order.getUserId()), asyncExecutor);

        CompletableFuture<Product> productFuture = CompletableFuture
            .supplyAsync(() -> productService.getProduct(order.getProductId()), asyncExecutor);

        CompletableFuture<Payment> paymentFuture = CompletableFuture
            .supplyAsync(() -> paymentService.getPayment(order.getPaymentId()), asyncExecutor);

        // 等待所有任务完成
        CompletableFuture.allOf(userFuture, productFuture, paymentFuture).join();

        return buildOrderDetail(
            order,
            userFuture.join(),
            productFuture.join(),
            paymentFuture.join()
        );
    }
}

// AsyncConfig.java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "asyncExecutor")
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}
```

### 4. 连接池优化

#### HikariCP配置

### ✅ 好的实践

```yaml
# application.yml
spring:
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      # 最小连接数
      minimum-idle: 10
      # 最大连接数
      maximum-pool-size: 50
      # 连接超时时间（毫秒）
      connection-timeout: 30000
      # 空闲连接最大存活时间（毫秒）
      idle-timeout: 600000
      # 连接最大存活时间（毫秒）
      max-lifetime: 1800000
      # 连接测试查询
      connection-test-query: SELECT 1
      # 自动提交
      auto-commit: true
      # 连接池名称
      pool-name: HikariPool
```

---

## 数据库性能优化

### 1. 索引优化

详见 [database-best-practices.md](./database-best-practices.md#索引优化)

### 2. 查询优化

#### 使用EXPLAIN分析

```sql
-- 分析查询执行计划
EXPLAIN SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.city_id = 1
ORDER BY u.created_at DESC
LIMIT 10;
```

#### 避免全表扫描

### ✅ 好的实践

```sql
-- 使用索引
SELECT * FROM users WHERE email = 'test@example.com';
-- 需要在email字段上创建索引

-- 避免在WHERE中使用函数
SELECT * FROM users WHERE DATE(created_at) = '2026-02-13';
-- 改为：
SELECT * FROM users WHERE created_at >= '2026-02-13' AND created_at < '2026-02-14';

-- 避免使用!=或<>
SELECT * FROM users WHERE status != 'deleted';
-- 改为：
SELECT * FROM users WHERE status IN ('active', 'inactive');
```

### 3. 读写分离

### ✅ 好的实践

```java
// 配置主从数据源
@Configuration
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties("spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.slave")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public DataSource dynamicDataSource() {
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put("master", masterDataSource());
        targetDataSources.put("slave", slaveDataSource());

        DynamicRoutingDataSource dataSource = new DynamicRoutingDataSource();
        dataSource.setTargetDataSources(targetDataSources);
        dataSource.setDefaultTargetDataSource(masterDataSource());

        return dataSource;
    }
}

// 动态路由数据源
public class DynamicRoutingDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContextHolder.getDataSource();
    }
}

// 数据源上下文
public class DataSourceContextHolder {

    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

    public static void setDataSource(String dataSource) {
        contextHolder.set(dataSource);
    }

    public static String getDataSource() {
        return contextHolder.get();
    }

    public static void clearDataSource() {
        contextHolder.remove();
    }
}

// 使用注解切换数据源
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DataSource {
    String value() default "master";
}

@Aspect
@Component
public class DataSourceAspect {

    @Around("@annotation(dataSource)")
    public Object around(ProceedingJoinPoint pjp, DataSource dataSource) throws Throwable {
        try {
            DataSourceContextHolder.setDataSource(dataSource.value());
            return pjp.proceed();
        } finally {
            DataSourceContextHolder.clearDataSource();
        }
    }
}

// 使用
@Service
public class UserService {

    @DataSource("master")
    public void createUser(User user) {
        // 写操作使用主库
        userRepository.save(user);
    }

    @DataSource("slave")
    public User getUser(Long id) {
        // 读操作使用从库
        return userRepository.findById(id).orElse(null);
    }
}
```

---

## 网络性能优化

### 1. HTTP/2

### ✅ 好的实践

```yaml
# application.yml
server:
  port: 8443
  http2:
    enabled: true
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: password
    key-store-type: PKCS12
```

### 2. CDN加速

### ✅ 好的实践

```html
<!-- 静态资源使用CDN -->
<link rel="stylesheet" href="https://cdn.example.com/css/main.css">
<script src="https://cdn.example.com/js/app.js"></script>

<!-- 第三方库使用公共CDN -->
<script src="https://cdn.jsdelivr.net/npm/vue@3.3.4/dist/vue.global.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios@1.4.0/dist/axios.min.js"></script>
```

### 3. 资源压缩

```nginx
# nginx.conf
http {
    # 启用Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/rss+xml
        font/truetype
        font/opentype
        application/vnd.ms-fontobject
        image/svg+xml;

    # 启用Brotli压缩（需要安装模块）
    brotli on;
    brotli_comp_level 6;
    brotli_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript;
}
```

### 4. 浏览器缓存

```nginx
# nginx.conf
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location ~* \.(html)$ {
    expires 10m;
    add_header Cache-Control "public, must-revalidate";
}
```

---

## 性能监控和分析

### 1. 前端性能监控

```typescript
// performance.ts
export class PerformanceMonitor {

  /**
   * 监控页面加载性能
   */
  static monitorPageLoad() {
    window.addEventListener('load', () => {
      const perfData = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming

      const metrics = {
        // DNS查询时间
        dns: perfData.domainLookupEnd - perfData.domainLookupStart,
        // TCP连接时间
        tcp: perfData.connectEnd - perfData.connectStart,
        // SSL握手时间
        ssl: perfData.connectEnd - perfData.secureConnectionStart,
        // TTFB（首字节时间）
        ttfb: perfData.responseStart - perfData.requestStart,
        // 页面下载时间
        download: perfData.responseEnd - perfData.responseStart,
        // DOM解析时间
        domParse: perfData.domInteractive - perfData.responseEnd,
        // 资源加载时间
        resource: perfData.loadEventStart - perfData.domContentLoadedEventEnd,
        // 总加载时间
        total: perfData.loadEventEnd - perfData.fetchStart
      }

      console.log('页面性能指标:', metrics)

      // 上报到监控平台
      this.reportMetrics(metrics)
    })
  }

  /**
   * 监控资源加载
   */
  static monitorResources() {
    const resources = performance.getEntriesByType('resource')

    resources.forEach(resource => {
      if (resource.duration > 1000) {
        console.warn('慢资源:', resource.name, resource.duration + 'ms')
      }
    })
  }

  /**
   * 监控长任务
   */
  static monitorLongTasks() {
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        console.warn('长任务:', entry.duration + 'ms')
      })
    })

    observer.observe({ entryTypes: ['longtask'] })
  }

  /**
   * 上报指标
   */
  private static reportMetrics(metrics: any) {
    // 上报到监控平台（如Sentry、DataDog等）
    fetch('/api/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(metrics)
    })
  }
}

// 启动监控
PerformanceMonitor.monitorPageLoad()
PerformanceMonitor.monitorResources()
PerformanceMonitor.monitorLongTasks()
```

### 2. 后端性能监控

```java
// PerformanceInterceptor.java
@Component
public class PerformanceInterceptor implements HandlerInterceptor {

    private static final Logger log = LoggerFactory.getLogger(PerformanceInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // 记录开始时间
        request.setAttribute("startTime", System.currentTimeMillis());
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        long startTime = (Long) request.getAttribute("startTime");
        long duration = System.currentTimeMillis() - startTime;

        String uri = request.getRequestURI();
        String method = request.getMethod();

        // 慢接口告警（超过1秒）
        if (duration > 1000) {
            log.warn("慢接口: {} {} 耗时: {}ms", method, uri, duration);
        } else {
            log.debug("接口: {} {} 耗时: {}ms", method, uri, duration);
        }

        // 上报到监控平台
        reportMetrics(method, uri, duration);
    }

    private void reportMetrics(String method, String uri, long duration) {
        // 上报到Prometheus、Grafana等监控平台
    }
}
```

---

## 检查清单

```yaml
前端性能:
  加载性能:
    - [ ] 代码分割（路由懒加载、组件懒加载）
    - [ ] 资源压缩（Gzip、Brotli）
    - [ ] 图片优化（WebP、响应式图片、懒加载）
    - [ ] 使用CDN
    - [ ] Tree Shaking

  渲染性能:
    - [ ] 虚拟列表（长列表）
    - [ ] 防抖节流
    - [ ] 避免不必要的渲染（v-memo、computed）
    - [ ] 使用Web Worker处理耗时任务

  交互性能:
    - [ ] 骨架屏
    - [ ] 加载动画
    - [ ] 分页加载
    - [ ] 无限滚动

后端性能:
  缓存:
    - [ ] Redis缓存（热点数据）
    - [ ] 本地缓存（Caffeine）
    - [ ] 多级缓存
    - [ ] 缓存穿透、击穿、雪崩防护

  数据库:
    - [ ] 索引优化
    - [ ] 查询优化（避免N+1、使用批量查询）
    - [ ] 连接池配置
    - [ ] 读写分离

  异步:
    - [ ] 异步任务（@Async）
    - [ ] 并行处理（CompletableFuture）
    - [ ] 消息队列（MQ）

网络性能:
  - [ ] HTTP/2
  - [ ] CDN加速
  - [ ] 资源压缩（Gzip、Brotli）
  - [ ] 浏览器缓存
  - [ ] 减少HTTP请求

监控:
  - [ ] 前端性能监控
  - [ ] 后端性能监控
  - [ ] 慢接口告警
  - [ ] 资源监控
```

---

## 总结

1. **前端优化**：代码分割、资源压缩、图片优化、虚拟列表
2. **后端优化**：缓存策略、异步处理、连接池优化
3. **数据库优化**：索引、查询优化、读写分离
4. **网络优化**：HTTP/2、CDN、资源压缩、缓存
5. **持续监控**：性能指标监控、慢接口告警、资源监控
