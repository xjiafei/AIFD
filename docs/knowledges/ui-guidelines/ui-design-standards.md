# UI设计规范

> AI全流程研发知识库 - UI设计统一规范
>
> **版本**: v1.0
> **更新日期**: 2026-02-13
> **适用范围**: 所有前端界面设计

## 目录

- [1. 设计原则](#1-设计原则)
- [2. 布局规范](#2-布局规范)
- [3. 颜色规范](#3-颜色规范)
- [4. 字体规范](#4-字体规范)
- [5. 组件规范](#5-组件规范)
- [6. 图标规范](#6-图标规范)
- [7. 间距规范](#7-间距规范)
- [8. 响应式设计](#8-响应式设计)
- [9. 无障碍设计](#9-无障碍设计)
- [10. 暗黑模式](#10-暗黑模式)

---

## 1. 设计原则

### 1.1 核心原则

#### 一致性（Consistency）
- 视觉一致：相同类型的元素使用统一的样式
- 交互一致：相同功能的操作保持一致的交互方式
- 术语一致：相同概念使用统一的术语

**示例**：
```
✅ 好的设计：
- 所有主操作按钮都使用蓝色
- 所有删除操作都需要二次确认
- 统一使用"新建"而非混用"创建"、"添加"

❌ 不好的设计：
- 主操作按钮有的蓝色有的绿色
- 有的删除需要确认，有的直接删除
- 术语混乱："新建用户"、"创建订单"、"添加商品"
```

#### 反馈（Feedback）
- 操作反馈：用户操作后立即给予视觉或文字反馈
- 状态反馈：清晰展示系统当前状态（加载中、成功、失败）
- 结果反馈：明确告知操作结果

**示例**：
```
✅ 好的设计：
- 按钮点击后显示loading状态
- 表单提交后显示成功提示Toast
- 文件上传显示进度条

❌ 不好的设计：
- 按钮点击后无反应，用户不知道是否成功
- 表单提交后无提示，用户不确定是否成功
- 长时间操作无进度提示
```

#### 效率（Efficiency）
- 减少步骤：最小化用户达成目标的步骤
- 智能默认：提供合理的默认值
- 快捷操作：为高频操作提供快捷方式

**示例**：
```
✅ 好的设计：
- 登录后自动跳转到用户上次访问的页面
- 表单提供常用选项的快捷选择
- 支持键盘快捷键（Ctrl+S保存）

❌ 不好的设计：
- 每次登录都跳转到首页，需要手动导航
- 每次都要从头选择，无历史记录
- 只能用鼠标操作，无快捷键
```

#### 容错（Fault Tolerance）
- 预防错误：通过设计减少用户犯错的可能
- 错误提示：明确告知错误原因和解决方法
- 撤销操作：允许用户撤销重要操作

**示例**：
```
✅ 好的设计：
- 重要操作（删除、支付）需要二次确认
- 表单实时验证，提前提示错误
- 提供"撤销"功能，允许恢复

❌ 不好的设计：
- 点击删除后直接删除，无确认
- 表单提交后才提示所有错误
- 重要操作无法撤销
```

### 1.2 设计理念

#### 以用户为中心
- 理解用户需求和使用场景
- 为用户提供价值，而非炫技
- 不断迭代优化用户体验

#### 简洁至上
- 去除不必要的元素
- 每个元素都有明确的目的
- "少即是多"

#### 优雅降级
- 优先保证核心功能可用
- 在低端设备或弱网环境下仍可使用
- 渐进增强，提供更好的体验

---

## 2. 布局规范

### 2.1 网格系统

#### 12列网格系统
- 使用12列网格布局，方便进行响应式设计
- 列间距（gutter）: 16px 或 24px

**示例**：
```css
/* 容器宽度 */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 16px;
}

/* 12列网格 */
.row {
  display: flex;
  margin: 0 -8px;
}

.col-1 { width: 8.33%; }
.col-2 { width: 16.66%; }
.col-3 { width: 25%; }
.col-4 { width: 33.33%; }
.col-6 { width: 50%; }
.col-12 { width: 100%; }
```

#### 栅格断点
```
xs: < 576px    (超小屏 - 手机)
sm: ≥ 576px    (小屏 - 手机横屏)
md: ≥ 768px    (中屏 - 平板)
lg: ≥ 992px    (大屏 - 桌面)
xl: ≥ 1200px   (超大屏 - 大桌面)
xxl: ≥ 1600px  (超超大屏 - 超宽屏)
```

### 2.2 页面布局

#### 标准布局结构
```
┌─────────────────────────────────────┐
│           Header (64px)             │  顶部导航栏
├─────────────────────────────────────┤
│       │                             │
│       │                             │
│ Side  │        Content             │  侧边栏 + 内容区
│ bar   │                             │
│       │                             │
│       │                             │
├─────────────────────────────────────┤
│          Footer (48px)              │  底部信息栏
└─────────────────────────────────────┘
```

#### 布局参数
```yaml
Header高度: 64px
Sidebar宽度:
  - 展开: 240px
  - 收起: 64px
Content最小宽度: 800px
Footer高度: 48px
```

### 2.3 卡片布局

#### 卡片组件
```css
.card {
  background: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  padding: 24px;
  margin-bottom: 16px;
}

.card-header {
  margin-bottom: 16px;
  font-size: 16px;
  font-weight: 600;
}

.card-body {
  font-size: 14px;
  line-height: 1.5;
}
```

**使用场景**：
- 信息分组展示
- 表单分块
- 列表项

---

## 3. 颜色规范

### 3.1 主题色

#### 品牌主色（Primary）
```css
--color-primary: #1890ff;         /* 主色 */
--color-primary-light: #40a9ff;   /* 浅色 */
--color-primary-lighter: #69c0ff; /* 更浅 */
--color-primary-dark: #096dd9;    /* 深色 */
--color-primary-darker: #0050b3;  /* 更深 */
```

**使用场景**：
- 主操作按钮
- 链接
- 选中状态
- 重要信息提示

#### 辅助色
```css
/* 成功色（Success） */
--color-success: #52c41a;
--color-success-light: #73d13d;
--color-success-lighter: #95de64;

/* 警告色（Warning） */
--color-warning: #faad14;
--color-warning-light: #ffc53d;
--color-warning-lighter: #ffd666;

/* 错误色（Error） */
--color-error: #ff4d4f;
--color-error-light: #ff7875;
--color-error-lighter: #ff9c9e;

/* 信息色（Info） */
--color-info: #1890ff;
--color-info-light: #40a9ff;
--color-info-lighter: #69c0ff;
```

**使用场景**：
- 成功色：成功提示、完成状态
- 警告色：警告提示、需要注意的信息
- 错误色：错误提示、失败状态、删除操作
- 信息色：普通提示、辅助信息

### 3.2 中性色

#### 文字颜色
```css
--color-text-primary: #000000d9;    /* 主要文字 (85%透明度) */
--color-text-secondary: #00000073;  /* 次要文字 (45%透明度) */
--color-text-tertiary: #00000040;   /* 辅助文字 (25%透明度) */
--color-text-disabled: #00000026;   /* 禁用文字 (15%透明度) */
```

#### 背景颜色
```css
--color-bg-default: #ffffff;     /* 默认背景 */
--color-bg-hover: #f5f5f5;       /* 悬停背景 */
--color-bg-active: #e6e6e6;      /* 激活背景 */
--color-bg-disabled: #f5f5f5;    /* 禁用背景 */
--color-bg-container: #fafafa;   /* 容器背景 */
```

#### 边框颜色
```css
--color-border-default: #d9d9d9;  /* 默认边框 */
--color-border-light: #e8e8e8;    /* 浅色边框 */
--color-border-lighter: #f0f0f0;  /* 更浅边框 */
```

### 3.3 颜色使用规则

#### 文字与背景对比度
- 正常文字（14px）：至少 4.5:1
- 大号文字（18px或14px粗体）：至少 3:1

**示例**：
```
✅ 好的设计：
- 白色背景 + 黑色文字（#000000）：对比度 21:1
- 蓝色背景（#1890ff）+ 白色文字：对比度 4.6:1

❌ 不好的设计：
- 浅灰背景（#f5f5f5）+ 浅灰文字（#d9d9d9）：对比度不足
- 黄色背景 + 白色文字：对比度不足
```

#### 颜色语义化
```
✅ 好的设计：
- 绿色表示成功、正常
- 红色表示错误、危险
- 黄色表示警告、需要注意
- 蓝色表示信息、链接

❌ 不好的设计：
- 红色表示成功（违反直觉）
- 绿色表示错误（混淆语义）
```

---

## 4. 字体规范

### 4.1 字体族

#### 西文字体
```css
font-family:
  -apple-system,           /* iOS & macOS */
  BlinkMacSystemFont,      /* macOS */
  "Segoe UI",              /* Windows */
  Roboto,                  /* Android */
  "Helvetica Neue",        /* macOS */
  Arial,                   /* 通用 */
  sans-serif;              /* 后备 */
```

#### 中文字体
```css
font-family:
  "PingFang SC",           /* macOS & iOS */
  "Microsoft YaHei",       /* Windows */
  "Hiragino Sans GB",      /* macOS */
  "Source Han Sans CN",    /* 思源黑体 */
  sans-serif;
```

#### 等宽字体（代码）
```css
font-family:
  "Fira Code",
  "Consolas",
  "Monaco",
  "Courier New",
  monospace;
```

### 4.2 字体大小

#### 标准字号
```css
--font-size-xs: 12px;      /* 辅助文字 */
--font-size-sm: 13px;      /* 次要文字 */
--font-size-base: 14px;    /* 正文（默认） */
--font-size-md: 16px;      /* 小标题 */
--font-size-lg: 18px;      /* 中标题 */
--font-size-xl: 20px;      /* 大标题 */
--font-size-xxl: 24px;     /* 页面标题 */
--font-size-xxxl: 32px;    /* 特大标题 */
```

**使用场景**：
```
32px - H1页面主标题
24px - H2章节标题
20px - H3小节标题
18px - H4段落标题
16px - 重要信息、卡片标题
14px - 正文、按钮、表单
13px - 次要说明
12px - 辅助信息、时间戳
```

### 4.3 字重（Font Weight）

```css
--font-weight-light: 300;    /* 细体 */
--font-weight-regular: 400;  /* 常规（默认） */
--font-weight-medium: 500;   /* 中粗 */
--font-weight-semibold: 600; /* 半粗 */
--font-weight-bold: 700;     /* 粗体 */
```

**使用规则**：
```
✅ 好的设计：
- 标题使用 600 或 700
- 正文使用 400
- 强调信息使用 500 或 600

❌ 不好的设计：
- 过度使用粗体，降低重点信息的辨识度
- 大段正文使用细体（300），降低可读性
```

### 4.4 行高（Line Height）

```css
--line-height-tight: 1.2;    /* 标题 */
--line-height-normal: 1.5;   /* 正文（默认） */
--line-height-relaxed: 1.8;  /* 长文本 */
--line-height-loose: 2.0;    /* 特殊场景 */
```

**使用规则**：
- 标题：1.2 - 1.3
- 正文：1.5 - 1.6
- 长文本阅读：1.8 - 2.0

---

## 5. 组件规范

### 5.1 按钮（Button）

#### 按钮类型
```html
<!-- 主要按钮（Primary） -->
<button class="btn btn-primary">确定</button>

<!-- 默认按钮（Default） -->
<button class="btn btn-default">取消</button>

<!-- 文字按钮（Text） -->
<button class="btn btn-text">了解更多</button>

<!-- 危险按钮（Danger） -->
<button class="btn btn-danger">删除</button>
```

#### 按钮尺寸
```css
/* 大按钮 */
.btn-large {
  height: 40px;
  padding: 0 24px;
  font-size: 16px;
}

/* 默认按钮 */
.btn-default {
  height: 32px;
  padding: 0 16px;
  font-size: 14px;
}

/* 小按钮 */
.btn-small {
  height: 24px;
  padding: 0 8px;
  font-size: 12px;
}
```

#### 按钮状态
```css
/* 默认状态 */
.btn-primary {
  background: #1890ff;
  color: #ffffff;
  border: 1px solid #1890ff;
}

/* 悬停状态 */
.btn-primary:hover {
  background: #40a9ff;
  border-color: #40a9ff;
}

/* 激活状态 */
.btn-primary:active {
  background: #096dd9;
  border-color: #096dd9;
}

/* 禁用状态 */
.btn-primary:disabled {
  background: #f5f5f5;
  color: #00000026;
  border-color: #d9d9d9;
  cursor: not-allowed;
}
```

#### 按钮使用规则
```
✅ 好的设计：
- 主操作使用主要按钮（蓝色）
- 次要操作使用默认按钮（灰色）
- 危险操作使用危险按钮（红色）
- 每个区域最多一个主要按钮

❌ 不好的设计：
- 所有按钮都使用主要按钮
- 删除操作使用主要按钮（应该用危险按钮）
- 按钮文字过长（超过8个字符）
```

### 5.2 表单（Form）

#### 表单布局
```html
<form class="form">
  <div class="form-item">
    <label class="form-label">用户名</label>
    <input type="text" class="form-input" placeholder="请输入用户名">
    <div class="form-error">用户名不能为空</div>
  </div>
</form>
```

#### 表单样式
```css
.form-item {
  margin-bottom: 24px;
}

.form-label {
  display: block;
  margin-bottom: 8px;
  font-size: 14px;
  font-weight: 500;
  color: #000000d9;
}

.form-input {
  width: 100%;
  height: 32px;
  padding: 0 12px;
  border: 1px solid #d9d9d9;
  border-radius: 4px;
  font-size: 14px;
}

.form-input:focus {
  border-color: #1890ff;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
}

.form-error {
  margin-top: 4px;
  font-size: 12px;
  color: #ff4d4f;
}
```

#### 表单验证规则
```
✅ 好的设计：
- 必填项标记清晰（红色*号）
- 实时验证，即时反馈
- 错误提示明确，告知如何修正
- 成功后清晰提示

❌ 不好的设计：
- 提交后才验证所有字段
- 错误提示模糊："格式错误"
- 验证规则过于严格，体验差
```

### 5.3 表格（Table）

#### 表格结构
```html
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
        <a href="#">编辑</a>
        <a href="#">删除</a>
      </td>
    </tr>
  </tbody>
</table>
```

#### 表格样式
```css
.table {
  width: 100%;
  border-collapse: collapse;
  background: #ffffff;
}

.table th {
  background: #fafafa;
  font-weight: 600;
  text-align: left;
  padding: 16px;
  border-bottom: 1px solid #e8e8e8;
}

.table td {
  padding: 16px;
  border-bottom: 1px solid #f0f0f0;
}

.table tr:hover {
  background: #fafafa;
}
```

#### 表格设计规则
```
✅ 好的设计：
- 标题行固定，内容可滚动
- 行高适中（48px），便于点击
- 操作列固定在右侧
- 支持排序、筛选、分页

❌ 不好的设计：
- 表格内容过多，无分页
- 单元格内容过长，未省略
- 无hover效果，难以辨识当前行
```

### 5.4 弹窗（Modal）

#### 弹窗结构
```html
<div class="modal">
  <div class="modal-mask"></div>
  <div class="modal-content">
    <div class="modal-header">
      <h3>标题</h3>
      <button class="modal-close">×</button>
    </div>
    <div class="modal-body">
      内容区域
    </div>
    <div class="modal-footer">
      <button class="btn btn-default">取消</button>
      <button class="btn btn-primary">确定</button>
    </div>
  </div>
</div>
```

#### 弹窗样式
```css
.modal-mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal-content {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 520px;
  background: #ffffff;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 1001;
}

.modal-header {
  padding: 16px 24px;
  border-bottom: 1px solid #f0f0f0;
}

.modal-body {
  padding: 24px;
  max-height: 60vh;
  overflow-y: auto;
}

.modal-footer {
  padding: 10px 16px;
  border-top: 1px solid #f0f0f0;
  text-align: right;
}
```

#### 弹窗使用规则
```
✅ 好的设计：
- 标题简洁明了
- 内容可滚动，高度不超过屏幕的60%
- 主要操作在右侧，次要操作在左侧
- 点击遮罩或ESC键可关闭

❌ 不好的设计：
- 弹窗嵌套弹窗
- 内容过多，无法滚动
- 无明确的关闭方式
- 强制用户完成操作才能关闭
```

---

## 6. 图标规范

### 6.1 图标库选择

#### 推荐图标库
- **Ant Design Icons**: 适合中后台系统
- **Material Icons**: 适合移动端和Web
- **Feather Icons**: 简洁风格
- **Heroicons**: 现代设计

### 6.2 图标尺寸

```css
--icon-xs: 12px;    /* 超小图标 */
--icon-sm: 14px;    /* 小图标 */
--icon-md: 16px;    /* 中等图标（默认） */
--icon-lg: 20px;    /* 大图标 */
--icon-xl: 24px;    /* 超大图标 */
```

**使用场景**：
```
24px - 导航图标、功能图标
20px - 按钮图标
16px - 表单图标、文字图标
14px - 小型操作图标
12px - 辅助图标
```

### 6.3 图标使用规则

```
✅ 好的设计：
- 图标与文字垂直居中对齐
- 图标与文字间距4-8px
- 使用语义化的图标（删除用垃圾桶）
- 图标颜色与文字颜色一致

❌ 不好的设计：
- 图标与文字不对齐
- 图标与文字太近或太远
- 图标语义不明确
- 图标颜色与文字颜色不一致
```

---

## 7. 间距规范

### 7.1 间距系统

#### 8px基准间距
```css
--space-xs: 4px;     /* 0.5x */
--space-sm: 8px;     /* 1x - 基准 */
--space-md: 16px;    /* 2x */
--space-lg: 24px;    /* 3x */
--space-xl: 32px;    /* 4x */
--space-xxl: 48px;   /* 6x */
--space-xxxl: 64px;  /* 8x */
```

### 7.2 间距使用场景

#### 组件内间距
```
4px  - 紧密相关的元素（图标与文字）
8px  - 相关元素（标签与输入框）
16px - 组件内部块（卡片内容）
24px - 组件间（表单项之间）
```

#### 页面间距
```
16px - 小屏内边距
24px - 中屏内边距
32px - 大屏内边距
48px - 章节间距
64px - 大区域间距
```

### 7.3 间距规则

```
✅ 好的设计：
- 使用8的倍数作为间距
- 相关元素间距小，不相关元素间距大
- 保持视觉呼吸感

❌ 不好的设计：
- 间距不统一（有的10px，有的15px）
- 元素之间过于紧密或过于分散
- 忽略视觉层次
```

---

## 8. 响应式设计

### 8.1 断点设计

#### 标准断点
```css
/* 超小屏（手机） */
@media (max-width: 575px) {
  .container { padding: 0 16px; }
}

/* 小屏（手机横屏） */
@media (min-width: 576px) and (max-width: 767px) {
  .container { padding: 0 24px; }
}

/* 中屏（平板） */
@media (min-width: 768px) and (max-width: 991px) {
  .container { padding: 0 32px; }
}

/* 大屏（桌面） */
@media (min-width: 992px) {
  .container { max-width: 1200px; }
}
```

### 8.2 响应式规则

#### 移动端优先
```css
/* 基础样式（移动端） */
.card {
  width: 100%;
  padding: 16px;
}

/* 平板及以上 */
@media (min-width: 768px) {
  .card {
    width: 50%;
    padding: 24px;
  }
}

/* 桌面 */
@media (min-width: 992px) {
  .card {
    width: 33.33%;
  }
}
```

#### 响应式策略
```
✅ 好的设计：
- 移动端单列布局，桌面端多列布局
- 移动端隐藏次要信息，桌面端完整展示
- 触摸目标至少44×44px

❌ 不好的设计：
- 移动端强制显示桌面布局
- 文字过小，不便阅读
- 按钮过小，难以点击
```

---

## 9. 无障碍设计

### 9.1 语义化HTML

```html
✅ 好的设计：
<nav>
  <ul>
    <li><a href="#home">首页</a></li>
  </ul>
</nav>

<article>
  <h1>文章标题</h1>
  <p>文章内容...</p>
</article>

❌ 不好的设计：
<div class="nav">
  <div class="item">首页</div>
</div>

<div class="article">
  <div class="title">文章标题</div>
  <div class="content">文章内容...</div>
</div>
```

### 9.2 可访问性（A11y）

#### ARIA属性
```html
<!-- 按钮 -->
<button aria-label="关闭对话框">×</button>

<!-- 加载状态 -->
<div role="status" aria-live="polite" aria-busy="true">
  正在加载...
</div>

<!-- 表单 -->
<input
  type="text"
  id="username"
  aria-required="true"
  aria-invalid="false"
  aria-describedby="username-error"
>
<div id="username-error">用户名不能为空</div>
```

#### 键盘导航
```
✅ 好的设计：
- 支持Tab键导航
- 支持Enter/Space激活
- 支持ESC关闭弹窗
- 焦点可见（outline或自定义样式）

❌ 不好的设计：
- 无法用键盘操作
- 焦点不可见
- Tab顺序混乱
```

---

## 10. 暗黑模式

### 10.1 颜色定义

#### 亮色模式
```css
:root {
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f5;
  --text-primary: #000000d9;
  --text-secondary: #00000073;
}
```

#### 暗色模式
```css
[data-theme="dark"] {
  --bg-primary: #1f1f1f;
  --bg-secondary: #2a2a2a;
  --text-primary: #ffffffd9;
  --text-secondary: #ffffff73;
}
```

### 10.2 暗黑模式规则

```
✅ 好的设计：
- 使用CSS变量管理颜色
- 降低对比度，避免刺眼
- 保持品牌色可识别
- 图片和图标适配暗色模式

❌ 不好的设计：
- 纯黑背景（#000000），过于刺眼
- 纯白文字（#ffffff），对比度过高
- 图片在暗色模式下过亮
```

---

## 总结

### 设计检查清单

```yaml
布局:
  - [ ] 使用12列网格系统
  - [ ] 断点设置合理
  - [ ] 移动端优先设计

颜色:
  - [ ] 使用统一的色彩系统
  - [ ] 文字与背景对比度达标（≥4.5:1）
  - [ ] 颜色语义化清晰

字体:
  - [ ] 使用系统字体族
  - [ ] 字号符合规范（12-32px）
  - [ ] 行高合理（1.5-1.8）

组件:
  - [ ] 按钮类型清晰（主要、默认、危险）
  - [ ] 表单验证即时反馈
  - [ ] 表格支持排序和筛选
  - [ ] 弹窗可关闭

间距:
  - [ ] 使用8px基准间距
  - [ ] 间距统一，有规律

响应式:
  - [ ] 支持多种屏幕尺寸
  - [ ] 移动端体验良好

无障碍:
  - [ ] 语义化HTML
  - [ ] 支持键盘导航
  - [ ] ARIA属性完整

暗黑模式:
  - [ ] 支持暗黑模式切换
  - [ ] 颜色对比度合理
```

### 参考资源

- [Material Design](https://material.io/design)
- [Ant Design](https://ant.design/docs/spec/introduce-cn)
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [Web Content Accessibility Guidelines (WCAG)](https://www.w3.org/WAI/WCAG21/quickref/)

---

**记住**：好的UI设计是为用户服务的，而非炫技。保持简洁、一致、高效、易用！
