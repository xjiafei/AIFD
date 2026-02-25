---
name: design-feature
description: |
  为新功能启动快速产品设计工作流，适合简单功能的快速设计。
  Use when the user wants to quickly design a feature, or mentions '快速设计', 'design feature', '功能设计'.
---

# 快速功能设计技能

为新功能启动产品设计阶段，适合简单功能的快速设计流程。

> 💡 **提示**：如果是复杂或核心功能，建议使用 `/senior-pm` 进行深度产品设计。

## 参数

- 功能名称: {args} (如未提供，询问用户)

## 工作流程

### 步骤1：创建功能目录结构

通过检查现有的 `docs/feature*-specs/` 目录找到下一个可用的功能编号，创建完整目录结构：

```
docs/feature{N}-specs/
├── requirements/           # 原始需求
├── product/                # 产品设计
│   ├── prd/                # PRD文档
│   ├── prototype/          # 原型设计
│   └── user_story/         # 用户故事
├── tech/                   # 技术设计
│   ├── arch/
│   ├── backend_design/
│   ├── frontend_design/
│   └── test_design/
├── testing/
├── deploy/
└── review_records/
    ├── product_review/
    ├── tech_review/
    └── code_review/
```

### 步骤2：需求分析

**要求用户详细描述功能需求**，并分析识别：

#### 核心问题
- 要解决什么问题？
- 为什么现在需要这个功能？
- 不做会有什么影响？

#### 用户角色
- 受影响的用户角色有哪些？
- 每个角色的需求是什么？

#### 业务目标
- 这个功能的业务目标是什么？
- 如何衡量成功？

#### 成功指标
- 关键指标（KPI）是什么？
- 目标值是多少？

#### 约束和依赖
- 技术约束
- 时间约束
- 资源约束
- 依赖的其他系统或功能

**输出文档**：`docs/feature{N}-specs/requirements/requirements.md`

### 步骤3：产品需求文档（PRD）

生成简化版PRD，包括以下章节：

#### 3.1 功能概述
- 功能名称
- 功能描述
- 功能目标
- 目标用户

#### 3.2 用户故事和用例
- 主要用户故事
- 典型使用场景
- 用户操作流程

#### 3.3 功能需求
- 核心功能列表（优先级：P0/P1/P2）
- 功能详细说明
- 业务规则

#### 3.4 非功能需求
- 性能要求（响应时间、并发量）
- 安全要求（权限、数据保护）
- 兼容性要求（浏览器、设备）
- 可用性要求

#### 3.5 验收标准
- 功能验收标准
- 质量验收标准
- 性能验收标准

#### 3.6 范围外项目
- 明确不做什么
- 未来考虑的功能

**参考资源**：
- 模板：`docs/knowledges/templates/product-templates/prd-template.md`
- 领域知识：`docs/knowledges/domain/`

**输出文档**：`docs/feature{N}-specs/product/prd/prd.md`

### 步骤4：用户故事

将需求分解为详细的用户故事。

**格式**：
```markdown
## Story {N}: {故事标题}

**作为** {用户类型}
**我想要** {目标}
**以便** {收益}

### 验收标准
- [ ] 标准1
- [ ] 标准2
- [ ] 标准3

### 优先级
P0/P1/P2

### 估算
{故事点}

### 依赖
{依赖的其他故事}
```

**要求**：
- 每个用户故事都要有明确的验收标准
- 验收标准必须可测试
- 标注优先级（P0必须、P1重要、P2可选）

**输出文档**：`docs/feature{N}-specs/product/user_story/stories.md`

### 步骤5：UI/UX原型

创建原型设计说明文档。

**内容包括**：

#### 5.1 页面列表
- 页面1：{页面名称} - {页面目标}
- 页面2：{页面名称} - {页面目标}

#### 5.2 页面详细设计

对每个页面：

**页面布局**
- 使用文字描述布局结构
- 如需要，使用ASCII图表示意

**关键元素**
- 列出页面上的关键元素
- 说明每个元素的作用

**交互说明**
- 用户操作（点击、输入、滑动等）
- 系统反馈（提示、跳转、变化等）

**状态说明**
- 初始状态
- 加载状态
- 成功状态
- 错误状态
- 空状态

#### 5.3 用户流程
- 描述用户从进入到完成目标的完整流程
- 标注关键决策点

**参考资源**：
- UI指南：`docs/knowledges/ui-guidelines/`（如有）

**输出文档**：`docs/feature{N}-specs/product/prototype/wireframes.md`

### 步骤6：请求审核

#### 6.1 创建文档摘要

生成所有产品设计文档的摘要：

```markdown
# Feature {N} 产品设计文档摘要

## 功能概述
{一段话描述功能}

## 核心文档
- ✅ 需求分析：docs/feature{N}-specs/requirements/requirements.md
- ✅ PRD：docs/feature{N}-specs/product/prd/prd.md
- ✅ 用户故事：docs/feature{N}-specs/product/user_story/stories.md
- ✅ 原型设计：docs/feature{N}-specs/product/prototype/wireframes.md

## 关键要点
1. 要点1
2. 要点2
3. 要点3

## 待确认事项
1. 事项1
2. 事项2

## 下一步
审核通过后，使用 `/tech-design {feature-name}` 进入技术设计阶段。
```

#### 6.2 请求用户审核

明确告知用户：
- 产品设计文档已完成
- 请审核所有文档
- 指出任何需要修改的地方
- 确认是否批准进入技术设计阶段

#### 6.3 记录审核反馈

在 `docs/feature{N}-specs/review_records/product_review/review-{timestamp}.md` 中记录：
- 审核时间
- 审核人
- 审核意见
- 修改建议
- 审核结论（通过/待修改/不通过）

#### 6.4 迭代或推进

- **如果需要修改**：根据反馈修改文档，重新请求审核
- **如果批准**：通知用户可以继续执行 `/tech-design {feature-name}`

## 重要说明

### 1. 知识库优先
始终先阅读现有知识库文档以确保一致性：
- 查看 `docs/knowledges/templates/` 中的模板
- 查看 `docs/knowledges/domain/` 中的领域知识
- 查看 `docs/knowledges/standards/` 中的编码规范
- 查看 `docs/knowledges/best-practices/` 中的最佳实践

### 2. 基线规格
如果这是第一个功能，还要在 `docs/specs/` 中初始化基线规格：
- 项目整体架构
- 技术栈选型
- 数据库设计规范
- API设计规范

### 3. 文档质量
- 保持文档专注和可操作
- 所有文档使用Markdown格式
- 长文档包含目录（使用 `[TOC]` 或手动创建）
- 交叉引用相关文档

### 4. 文档结构示例

```markdown
# 功能名称

## 目录
- [功能概述](#功能概述)
- [用户故事](#用户故事)
- [功能需求](#功能需求)
- ...

## 功能概述
...

## 相关文档
- [技术设计](../tech/arch/architecture.md)
- [API设计](../tech/backend_design/api-spec.md)
```

## 输出示例

```
✅ 产品设计已完成！

📁 Feature编号：feature003

📄 已生成文档：
- ✅ 需求分析：docs/feature003-specs/requirements/requirements.md
- ✅ PRD：docs/feature003-specs/product/prd/prd.md
- ✅ 用户故事：docs/feature003-specs/product/user_story/stories.md
- ✅ 原型设计：docs/feature003-specs/product/prototype/wireframes.md

🎯 核心要点：
1. 实现用户登录功能，支持手机号+验证码
2. 主要用户：所有APP用户
3. 核心指标：登录成功率>99%，响应时间<2秒

📋 待确认事项：
1. 是否支持第三方登录（微信、支付宝）？
2. 验证码有效期设置为多久？

---

请审核以上文档，确认无误后回复"批准"或提出修改意见。

审核通过后，使用 `/tech-design 用户登录` 进入技术设计阶段。
```

## 与 senior-pm 的对比

| 特性 | design-feature | senior-pm |
|------|----------------|-----------|
| 需求分析 | 基础分析 | 深度分析（5W2H） |
| PRD章节 | 6个章节 | 10个章节 |
| 竞品分析 | 可选 | 必须（3-5个） |
| 商业价值 | 定性描述 | ROI量化 |
| 数据指标 | 基础指标 | 详细埋点方案 |
| 交付时间 | 30分钟-1小时 | 2-4小时 |
| 适用场景 | 简单功能 | 核心/复杂功能 |

## 使用建议

**使用 design-feature 的场景**：
- ✅ CRUD操作
- ✅ 简单的表单提交
- ✅ 数据展示页面
- ✅ 导出导入功能
- ✅ 简单的配置管理

**改用 senior-pm 的场景**：
- ⚠️ 登录认证系统
- ⚠️ 支付流程
- ⚠️ 权限管理
- ⚠️ 核心业务流程
- ⚠️ 需要ROI分析的功能
