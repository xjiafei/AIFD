# 交互设计规范

> AI全流程研发知识库 - 交互设计原则和模式
>
> **版本**: v1.0
> **更新日期**: 2026-02-13
> **适用范围**: 所有交互设计

## 目录

- [1. 交互设计原则](#1-交互设计原则)
- [2. 导航设计](#2-导航设计)
- [3. 反馈机制](#3-反馈机制)
- [4. 加载状态](#4-加载状态)
- [5. 错误处理](#5-错误处理)
- [6. 表单交互](#6-表单交互)
- [7. 数据展示](#7-数据展示)
- [8. 动效设计](#8-动效设计)
- [9. 手势交互](#9-手势交互)
- [10. 微交互](#10-微交互)

---

## 1. 交互设计原则

### 1.1 费茨定律（Fitts' Law）

**原则**：到达目标的时间与目标大小和距离有关

**应用**：
```
✅ 好的设计：
- 主要操作按钮足够大（最小44×44px）
- 相关操作放在一起，减少移动距离
- 频繁使用的功能放在易于触及的位置

❌ 不好的设计：
- 按钮过小，难以点击
- 相关操作分散在不同位置
- 频繁操作需要大范围移动鼠标
```

**实践示例**：
```html
<!-- 移动端操作栏 -->
<div class="action-bar">
  <!-- 主要操作（大按钮，易触达） -->
  <button class="btn-primary" style="height: 48px; width: 100%;">
    提交订单
  </button>

  <!-- 次要操作（靠近主操作） -->
  <div style="margin-top: 8px;">
    <button class="btn-text">查看详情</button>
    <button class="btn-text">保存草稿</button>
  </div>
</div>
```

### 1.2 希克定律（Hick's Law）

**原则**：选项越多,决策时间越长

**应用**：
```
✅ 好的设计：
- 菜单项控制在5-7个
- 使用分组减少选择压力
- 提供默认选项和推荐选项

❌ 不好的设计：
- 一次性展示20+选项
- 所有选项平铺，无分类
- 无默认值，用户需要从头选择
```

**实践示例**：
```html
<!-- 分组选择器 -->
<select>
  <optgroup label="常用选项">
    <option selected>北京</option>
    <option>上海</option>
    <option>深圳</option>
  </optgroup>
  <optgroup label="其他城市">
    <option>成都</option>
    <option>杭州</option>
    <!-- 更多选项 -->
  </optgroup>
</select>
```

### 1.3 米勒定律（Miller's Law）

**原则**：人的短期记忆容量约为7±2个信息单元

**应用**：
```
✅ 好的设计：
- 导航菜单控制在7个以内
- 步骤条不超过7步
- 分块展示信息（chunking）

❌ 不好的设计：
- 顶部导航10+菜单项
- 表单一次性展示30个字段
- 长列表无分组
```

**实践示例**：
```html
<!-- 步骤条（5步） -->
<div class="steps">
  <div class="step active">填写信息</div>
  <div class="step">上传资料</div>
  <div class="step">确认订单</div>
  <div class="step">支付</div>
  <div class="step">完成</div>
</div>

<!-- 如果步骤过多，使用折叠 -->
<div class="steps-collapsed">
  <div class="step completed">步骤1-3</div>
  <div class="step active">步骤4</div>
  <div class="step">步骤5-7</div>
</div>
```

### 1.4 雅各布定律（Jakob's Law）

**原则**：用户在其他网站上花费的时间最多，他们期望你的网站以相同的方式工作

**应用**：
```
✅ 好的设计：
- Logo在左上角，点击返回首页
- 搜索框在顶部右侧
- 购物车图标在右上角
- 登录/注册在右上角

❌ 不好的设计：
- Logo在右下角
- 搜索框在底部
- 违反用户习惯的操作方式
```

### 1.5 泰思勒定律（Tesler's Law）

**原则**：复杂性守恒,不能消除只能转移

**应用**：
```
✅ 好的设计：
- 为用户提供默认值,减少配置
- 使用向导模式,分步引导
- 提供模板,减少从零开始的复杂度

❌ 不好的设计：
- 把所有复杂性推给用户
- 无默认值,全部手动配置
- 无引导,用户不知道如何开始
```

---

## 2. 导航设计

### 2.1 顶部导航

#### 标准顶部导航
```html
<header class="header">
  <!-- Logo -->
  <div class="logo">
    <a href="/">
      <img src="logo.png" alt="公司名称">
    </a>
  </div>

  <!-- 主导航 -->
  <nav class="main-nav">
    <a href="/products">产品</a>
    <a href="/solutions">解决方案</a>
    <a href="/pricing">价格</a>
    <a href="/about">关于我们</a>
  </nav>

  <!-- 右侧操作 -->
  <div class="header-actions">
    <button class="btn-icon">
      <i class="icon-search"></i>
    </button>
    <button class="btn-text">登录</button>
    <button class="btn-primary">注册</button>
  </div>
</header>
```

#### 导航规则
```
✅ 好的设计：
- Logo在左侧,点击返回首页
- 主导航居中或紧跟Logo
- 登录/注册在右上角
- 当前页面高亮显示
- 移动端收起为汉堡菜单

❌ 不好的设计：
- Logo不可点击
- 导航项过多(超过7个)
- 当前页面无高亮
- 移动端导航不可用
```

### 2.2 侧边导航

#### 标准侧边导航
```html
<aside class="sidebar">
  <!-- 可折叠侧边栏 -->
  <div class="sidebar-header">
    <img src="logo.png" alt="Logo" class="sidebar-logo">
    <button class="btn-collapse">
      <i class="icon-menu-fold"></i>
    </button>
  </div>

  <!-- 导航菜单 -->
  <nav class="sidebar-nav">
    <div class="nav-group">
      <div class="nav-group-title">工作台</div>
      <a href="/dashboard" class="nav-item active">
        <i class="icon-dashboard"></i>
        <span>概览</span>
      </a>
      <a href="/projects" class="nav-item">
        <i class="icon-folder"></i>
        <span>项目</span>
      </a>
    </div>

    <div class="nav-group">
      <div class="nav-group-title">系统</div>
      <a href="/settings" class="nav-item">
        <i class="icon-settings"></i>
        <span>设置</span>
      </a>
    </div>
  </nav>
</aside>
```

#### 侧边导航规则
```
✅ 好的设计：
- 支持展开/收起
- 使用图标+文字
- 分组管理（不超过5组）
- 当前页面高亮
- 滚动时固定

❌ 不好的设计：
- 不支持折叠,占用空间
- 纯文字,无图标
- 所有菜单平铺,无分组
- 滚动时消失
```

### 2.3 面包屑导航

#### 标准面包屑
```html
<nav class="breadcrumb" aria-label="面包屑">
  <a href="/">首页</a>
  <span class="separator">/</span>
  <a href="/products">产品</a>
  <span class="separator">/</span>
  <span class="current">详情</span>
</nav>
```

#### 面包屑规则
```
✅ 好的设计：
- 展示完整路径层级
- 最后一级不可点击（当前页面）
- 分隔符统一（/或>）
- 位置在页面顶部，内容上方

❌ 不好的设计：
- 只显示"返回"，无完整路径
- 所有层级都可点击（包括当前页）
- 层级过深（超过5级）
```

### 2.4 标签页导航

#### 标准标签页
```html
<div class="tabs">
  <button class="tab active">基本信息</button>
  <button class="tab">详细信息</button>
  <button class="tab">配置</button>
</div>

<div class="tab-content">
  <!-- 当前标签页内容 -->
</div>
```

#### 标签页规则
```
✅ 好的设计：
- 当前标签页明显高亮
- 标签页数量控制在5-7个
- 支持键盘切换（左右箭头）
- 内容切换有过渡动画

❌ 不好的设计：
- 标签页过多，需要滚动
- 当前标签页不明显
- 切换时闪烁，无过渡
```

---

## 3. 反馈机制

### 3.1 操作反馈

#### Toast提示
```html
<!-- 成功提示 -->
<div class="toast toast-success">
  <i class="icon-check-circle"></i>
  <span>保存成功</span>
</div>

<!-- 错误提示 -->
<div class="toast toast-error">
  <i class="icon-x-circle"></i>
  <span>保存失败，请重试</span>
</div>

<!-- 警告提示 -->
<div class="toast toast-warning">
  <i class="icon-alert-triangle"></i>
  <span>网络连接不稳定</span>
</div>

<!-- 信息提示 -->
<div class="toast toast-info">
  <i class="icon-info"></i>
  <span>新版本可用</span>
</div>
```

#### Toast规则
```
✅ 好的设计：
- 自动消失（2-3秒）
- 位置固定（顶部居中或右上角）
- 一次只显示一个
- 文字简洁（20字以内）
- 支持手动关闭

❌ 不好的设计：
- 不会自动消失
- 位置不固定
- 同时显示多个，堆叠混乱
- 文字冗长
- 无法关闭
```

### 3.2 确认对话框

#### 标准确认框
```html
<div class="modal confirm-modal">
  <div class="modal-content">
    <div class="modal-icon">
      <i class="icon-alert-triangle"></i>
    </div>
    <h3 class="modal-title">确认删除？</h3>
    <p class="modal-message">
      删除后将无法恢复，确定要删除这条记录吗？
    </p>
    <div class="modal-actions">
      <button class="btn btn-default">取消</button>
      <button class="btn btn-danger">删除</button>
    </div>
  </div>
</div>
```

#### 确认框规则
```
✅ 好的设计：
- 危险操作必须确认
- 说明操作后果
- 危险按钮使用红色
- 默认焦点在取消按钮
- 支持ESC键关闭

❌ 不好的设计：
- 所有操作都需要确认（过度使用）
- 确认信息模糊："确定要这样做吗？"
- 危险按钮使用主色调（蓝色）
- 默认焦点在危险按钮
```

### 3.3 进度反馈

#### 进度条
```html
<!-- 确定进度 -->
<div class="progress">
  <div class="progress-bar" style="width: 60%;">
    <span>60%</span>
  </div>
</div>

<!-- 不确定进度 -->
<div class="progress">
  <div class="progress-bar indeterminate"></div>
</div>
```

#### 进度反馈规则
```
✅ 好的设计：
- 能准确计算进度时显示百分比
- 无法计算时使用不确定进度条
- 长时间操作提供取消功能
- 显示剩余时间（如果可计算）

❌ 不好的设计：
- 进度条不准确（长时间停在99%）
- 长时间操作无进度提示
- 无法取消，用户只能等待
```

---

## 4. 加载状态

### 4.1 加载动画

#### 全局加载
```html
<!-- Spinner加载 -->
<div class="loading-overlay">
  <div class="spinner"></div>
  <p>加载中...</p>
</div>
```

#### 局部加载
```html
<!-- 骨架屏 -->
<div class="skeleton">
  <div class="skeleton-header"></div>
  <div class="skeleton-content">
    <div class="skeleton-line"></div>
    <div class="skeleton-line"></div>
    <div class="skeleton-line short"></div>
  </div>
</div>
```

#### 按钮加载
```html
<button class="btn btn-primary loading" disabled>
  <span class="btn-spinner"></span>
  <span>提交中...</span>
</button>
```

### 4.2 加载策略

#### 策略选择
```
快速操作（< 300ms）：
  → 无需加载提示

短时操作（300ms - 3s）：
  → 使用Spinner或按钮loading状态

中时操作（3s - 10s）：
  → 使用进度条 + 提示文字

长时操作（> 10s）：
  → 使用骨架屏 + 进度条 + 提供取消功能
```

### 4.3 加载规则

```
✅ 好的设计：
- 首次加载使用骨架屏
- 按钮操作显示loading状态
- 长时间加载提供取消功能
- 加载失败提供重试按钮

❌ 不好的设计：
- 长时间空白，无任何提示
- 按钮点击后无反应
- 加载失败后无法重试
- 全屏loading，阻塞所有操作
```

---

## 5. 错误处理

### 5.1 表单错误

#### 实时验证
```html
<div class="form-item">
  <label for="email">邮箱</label>
  <input
    type="email"
    id="email"
    class="form-input error"
    value="invalid-email"
    aria-invalid="true"
    aria-describedby="email-error"
  >
  <div id="email-error" class="form-error">
    <i class="icon-alert-circle"></i>
    请输入有效的邮箱地址
  </div>
</div>
```

#### 表单错误规则
```
✅ 好的设计：
- 失去焦点时验证
- 错误信息明确："请输入有效的邮箱地址"
- 错误字段高亮（红色边框）
- 提供修改建议

❌ 不好的设计：
- 输入时就验证（过于敏感）
- 提交后才验证所有字段
- 错误信息模糊："格式错误"
- 不告知如何修正
```

### 5.2 页面错误

#### 404页面
```html
<div class="error-page">
  <div class="error-code">404</div>
  <h1 class="error-title">页面未找到</h1>
  <p class="error-message">
    抱歉，您访问的页面不存在或已被删除
  </p>
  <div class="error-actions">
    <a href="/" class="btn btn-primary">返回首页</a>
    <button class="btn btn-default" onclick="history.back()">
      返回上一页
    </button>
  </div>
</div>
```

#### 500页面
```html
<div class="error-page">
  <div class="error-code">500</div>
  <h1 class="error-title">服务器错误</h1>
  <p class="error-message">
    抱歉，服务器出现了一些问题，请稍后重试
  </p>
  <div class="error-actions">
    <button class="btn btn-primary" onclick="location.reload()">
      刷新页面
    </button>
    <a href="/" class="btn btn-default">返回首页</a>
  </div>
</div>
```

#### 错误页面规则
```
✅ 好的设计：
- 明确告知错误原因
- 提供解决方案或下一步操作
- 保持品牌风格，避免生硬
- 提供搜索或导航功能

❌ 不好的设计：
- 只显示技术错误码
- 无任何操作建议
- 页面风格与网站不符
- 无法返回或导航
```

### 5.3 网络错误

#### 离线提示
```html
<div class="offline-banner">
  <i class="icon-wifi-off"></i>
  <span>网络连接已断开，请检查网络设置</span>
  <button class="btn-text" onclick="location.reload()">
    重试
  </button>
</div>
```

#### 网络错误规则
```
✅ 好的设计：
- 明确提示网络问题
- 提供重试功能
- 离线时缓存用户操作
- 恢复后自动重试

❌ 不好的设计：
- 提示"服务器错误"（模糊）
- 无法重试，用户操作丢失
- 恢复后需要重新操作
```

---

## 6. 表单交互

### 6.1 输入框

#### 占位符
```html
<!-- 好的占位符 -->
<input type="text" placeholder="请输入手机号">

<!-- 不好的占位符 -->
<input type="text" placeholder="手机号（必填，11位数字）">
```

#### 输入框规则
```
✅ 好的设计：
- 占位符简洁，提示输入格式
- 必填项用*标记
- 焦点时高亮边框
- 输入时显示字符计数（如果有限制）

❌ 不好的设计：
- 占位符过长，包含所有说明
- 必填项不标记
- 无焦点样式
- 超出长度限制才提示
```

### 6.2 选择器

#### 下拉选择
```html
<select class="form-select">
  <option value="">请选择城市</option>
  <option value="beijing">北京</option>
  <option value="shanghai">上海</option>
</select>
```

#### 单选/多选
```html
<!-- 单选 -->
<div class="radio-group">
  <label class="radio">
    <input type="radio" name="gender" value="male">
    <span>男</span>
  </label>
  <label class="radio">
    <input type="radio" name="gender" value="female">
    <span>女</span>
  </label>
</div>

<!-- 多选 -->
<div class="checkbox-group">
  <label class="checkbox">
    <input type="checkbox" value="reading">
    <span>阅读</span>
  </label>
  <label class="checkbox">
    <input type="checkbox" value="sports">
    <span>运动</span>
  </label>
</div>
```

#### 选择器规则
```
✅ 好的设计：
- 选项≤7个用单选/多选，>7个用下拉
- 提供默认选项
- 选项按逻辑分组或排序
- 选中状态明显

❌ 不好的设计：
- 2个选项用下拉（应该用单选）
- 20个选项用单选（应该用下拉或搜索）
- 无默认值
- 选项顺序混乱
```

### 6.3 日期时间选择

#### 日期选择器
```html
<input
  type="text"
  class="datepicker"
  placeholder="选择日期"
  readonly
>
```

#### 日期选择器规则
```
✅ 好的设计：
- 提供日历面板
- 支持快捷选择（今天、本周、本月）
- 支持键盘输入（MM/DD/YYYY）
- 禁用不可选日期（灰色显示）

❌ 不好的设计：
- 只能手动输入
- 无快捷选择
- 不禁用不可选日期，提交时才报错
```

### 6.4 文件上传

#### 上传组件
```html
<div class="upload">
  <input type="file" id="file-input" hidden>
  <label for="file-input" class="upload-btn">
    <i class="icon-upload"></i>
    <span>点击上传</span>
  </label>
  <p class="upload-hint">
    支持jpg、png格式，大小不超过2MB
  </p>
</div>

<!-- 上传列表 -->
<div class="upload-list">
  <div class="upload-item">
    <i class="icon-file"></i>
    <span class="upload-name">图片.jpg</span>
    <span class="upload-size">1.2MB</span>
    <div class="upload-progress">
      <div class="progress-bar" style="width: 60%"></div>
    </div>
    <button class="btn-icon upload-remove">
      <i class="icon-x"></i>
    </button>
  </div>
</div>
```

#### 文件上传规则
```
✅ 好的设计：
- 支持点击和拖拽上传
- 明确说明文件要求（格式、大小）
- 显示上传进度
- 上传失败提供重试
- 支持预览（图片）

❌ 不好的设计：
- 只能点击选择
- 不说明文件要求，上传后才报错
- 无进度提示
- 上传失败后无法重试
- 无法预览
```

---

## 7. 数据展示

### 7.1 列表

#### 简单列表
```html
<ul class="list">
  <li class="list-item">
    <div class="list-title">标题</div>
    <div class="list-meta">2026-02-13</div>
  </li>
</ul>
```

#### 列表规则
```
✅ 好的设计：
- 支持分页或虚拟滚动
- 空状态友好提示
- 加载更多时显示loading
- 支持下拉刷新（移动端）

❌ 不好的设计：
- 一次性加载所有数据
- 空状态无提示或提示生硬
- 加载更多无反馈
```

### 7.2 表格

#### 响应式表格
```html
<!-- 桌面端：标准表格 -->
<table class="table">
  <thead>
    <tr>
      <th>姓名</th>
      <th>年龄</th>
      <th>操作</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>张三</td>
      <td>25</td>
      <td>
        <button class="btn-text">编辑</button>
      </td>
    </tr>
  </tbody>
</table>

<!-- 移动端：卡片列表 -->
<div class="card-list">
  <div class="card">
    <div class="card-row">
      <span class="label">姓名</span>
      <span class="value">张三</span>
    </div>
    <div class="card-row">
      <span class="label">年龄</span>
      <span class="value">25</span>
    </div>
    <div class="card-actions">
      <button class="btn-text">编辑</button>
    </div>
  </div>
</div>
```

#### 表格规则
```
✅ 好的设计：
- 支持排序和筛选
- 固定表头，内容可滚动
- 行hover效果
- 移动端转为卡片列表
- 空状态友好提示

❌ 不好的设计：
- 不支持排序筛选
- 表头随内容滚动
- 无hover效果
- 移动端显示不全
```

### 7.3 图表

#### 图表交互
```
✅ 好的设计：
- 支持鼠标悬停查看详细数据
- 支持点击筛选
- 支持缩放（时间范围）
- 提供图例说明

❌ 不好的设计：
- 无交互，只能看
- 无图例，不知道各项含义
- 数据密集时无法查看细节
```

---

## 8. 动效设计

### 8.1 动效原则

#### 时长原则
```css
/* 微动效 */
--duration-instant: 100ms;   /* 即时反馈 */
--duration-fast: 200ms;      /* 快速 */
--duration-base: 300ms;      /* 标准 */
--duration-slow: 500ms;      /* 慢速 */

/* 使用场景 */
.btn:hover {
  transition: background 100ms;  /* 即时反馈 */
}

.modal {
  transition: opacity 300ms;  /* 标准时长 */
}

.sidebar {
  transition: width 500ms;  /* 慢速，明显变化 */
}
```

#### 缓动函数
```css
/* 标准缓动 */
--easing-standard: cubic-bezier(0.4, 0, 0.2, 1);

/* 进入（加速） */
--easing-in: cubic-bezier(0.4, 0, 1, 1);

/* 退出（减速） */
--easing-out: cubic-bezier(0, 0, 0.2, 1);

/* 强调（先快后慢） */
--easing-emphasized: cubic-bezier(0.4, 0, 0, 1);
```

### 8.2 常见动效

#### 淡入淡出
```css
.fade-enter {
  opacity: 0;
}

.fade-enter-active {
  opacity: 1;
  transition: opacity 300ms ease-out;
}

.fade-exit {
  opacity: 1;
}

.fade-exit-active {
  opacity: 0;
  transition: opacity 200ms ease-in;
}
```

#### 滑动
```css
.slide-enter {
  transform: translateY(-20px);
  opacity: 0;
}

.slide-enter-active {
  transform: translateY(0);
  opacity: 1;
  transition: all 300ms ease-out;
}
```

#### 缩放
```css
.scale-enter {
  transform: scale(0.9);
  opacity: 0;
}

.scale-enter-active {
  transform: scale(1);
  opacity: 1;
  transition: all 200ms ease-out;
}
```

### 8.3 动效规则

```
✅ 好的设计：
- 动效时长200-500ms
- 使用缓动函数，避免线性
- 微交互用快速动效（100-200ms）
- 页面切换用标准动效（300ms）
- 提供关闭动效的选项（无障碍）

❌ 不好的设计：
- 动效过慢（>1s），影响体验
- 动效过快（<100ms），难以察觉
- 使用线性动效，不自然
- 过度使用动效，眼花缭乱
- 无法关闭动效
```

---

## 9. 手势交互

### 9.1 移动端手势

#### 标准手势
```
点击（Tap）：
  - 选择、确认
  - 最小触摸区域：44×44px

双击（Double Tap）：
  - 图片放大
  - 不推荐作为主要操作

长按（Long Press）：
  - 显示上下文菜单
  - 进入编辑模式

滑动（Swipe）：
  - 切换页面
  - 删除列表项

拖拽（Drag）：
  - 拖动排序
  - 调整大小

捏合（Pinch）：
  - 图片缩放
  - 地图缩放
```

### 9.2 手势规则

```
✅ 好的设计：
- 触摸区域≥44×44px
- 手势操作提供视觉反馈
- 滑动删除时显示"撤销"
- 长按后显示提示

❌ 不好的设计：
- 触摸区域过小
- 手势操作无反馈
- 删除后无法撤销
- 隐藏手势，用户不知道
```

---

## 10. 微交互

### 10.1 按钮微交互

```css
.btn {
  transition: all 100ms ease-out;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
```

### 10.2 输入框微交互

```css
.input {
  transition: border-color 200ms;
}

.input:focus {
  border-color: #1890ff;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
}
```

### 10.3 加载微交互

```css
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.spinner {
  animation: spin 1s linear infinite;
}
```

### 10.4 微交互规则

```
✅ 好的设计：
- 按钮点击有视觉反馈
- 输入框焦点状态明显
- 加载动画流畅
- 开关切换有动画

❌ 不好的设计：
- 按钮点击无反馈
- 输入框焦点不明显
- 加载时页面冻结
- 开关切换生硬
```

---

## 总结

### 交互设计检查清单

```yaml
导航:
  - [ ] 顶部导航清晰，Logo可点击
  - [ ] 侧边导航支持折叠
  - [ ] 面包屑展示完整路径
  - [ ] 标签页数量合理

反馈:
  - [ ] 操作后立即反馈
  - [ ] Toast自动消失
  - [ ] 危险操作需确认
  - [ ] 长时间操作显示进度

加载:
  - [ ] 快速操作无需提示
  - [ ] 中时操作使用进度条
  - [ ] 长时操作使用骨架屏
  - [ ] 加载失败可重试

错误:
  - [ ] 表单实时验证
  - [ ] 错误信息明确
  - [ ] 错误页面友好
  - [ ] 网络错误可重试

表单:
  - [ ] 占位符简洁
  - [ ] 必填项标记清晰
  - [ ] 选择器类型合适
  - [ ] 文件上传显示进度

数据:
  - [ ] 列表支持分页
  - [ ] 表格支持排序筛选
  - [ ] 空状态友好提示
  - [ ] 图表支持交互

动效:
  - [ ] 时长200-500ms
  - [ ] 使用缓动函数
  - [ ] 不过度使用
  - [ ] 可关闭动效

手势:
  - [ ] 触摸区域≥44×44px
  - [ ] 手势有视觉反馈
  - [ ] 支持常用手势

微交互:
  - [ ] 按钮有点击反馈
  - [ ] 输入框有焦点样式
  - [ ] 加载动画流畅
```

### 参考资源

- [Material Design - Motion](https://material.io/design/motion)
- [Apple Human Interface Guidelines - Interaction](https://developer.apple.com/design/human-interface-guidelines/inputs)
- [Nielsen Norman Group](https://www.nngroup.com/)
- [Laws of UX](https://lawsofux.com/)

---

**记住**：好的交互设计是用户无感知的，让操作变得自然、流畅、符合直觉！
