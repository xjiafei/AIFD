# 知识库文档索引

> AI全流程研发知识库，包含标准规范、最佳实践、代码模板等

## 📁 目录结构

```
knowledges/
├── standards/              # 编码规范
│   ├── git-commit-standards.md
│   ├── java-standards.md
│   ├── frontend-standards.md
│   ├── python-standards.md
│   ├── api-design-standards.md
│   ├── database-design-standards.md
│   ├── code-review-standards.md
│   ├── testing-standards.md
│   └── security-standards.md
├── best-practices/         # 最佳实践
│   ├── backend-best-practices.md
│   ├── frontend-best-practices.md
│   ├── api-best-practices.md
│   ├── database-best-practices.md
│   ├── testing-best-practices.md
│   ├── performance-optimization.md
│   └── security-best-practices.md
├── templates/              # 代码模板
│   ├── backend-patterns/
│   │   ├── service-template.md
│   │   ├── controller-template.md
│   │   ├── repository-template.md
│   │   └── db-design/
│   │       └── database-naming-conventions.md
│   ├── frontend-patterns/
│   │   ├── component-template.md
│   │   ├── page-template.md
│   │   └── store-template.md
│   ├── testing-patterns/
│   │   ├── unit-test-template.md
│   │   ├── integration-test-template.md
│   │   └── e2e-test-template.md
│   └── product-templates/
│       └── prd-template.md
├── domain/                 # 领域知识
│   ├── ddd-practices.md
│   └── domain-modeling.md
├── ui-guidelines/          # UI设计规范
│   ├── ui-design-standards.md
│   └── interaction-design.md
└── skills-guide/           # Skills使用指南
    ├── README.md
    ├── senior-pm-guide.md
    └── senior-pm-demo.md
```

## 📚 文档列表

### 1. 编码规范（Standards）

#### 1.1 语言规范
- **[Git提交规范](standards/git-commit-standards.md)**
  Git提交消息的约定式提交规范

- **[Java编码规范](standards/java-standards.md)**
  Java/Spring Boot后端编码规范

- **[前端编码规范](standards/frontend-standards.md)**
  Vue.js/TypeScript前端编码规范

- **[Python编码规范](standards/python-standards.md)** ✅
  Python测试代码编码规范（PEP 8）

#### 1.2 设计规范
- **[API设计规范](standards/api-design-standards.md)** ✅
  RESTful API设计标准和OpenAPI 3.0规范

- **[数据库设计规范](standards/database-design-standards.md)** 📝
  数据库表设计、索引、性能优化规范

#### 1.3 流程规范
- **[代码审查规范](standards/code-review-standards.md)** 📝
  Code Review流程和检查清单

- **[测试规范](standards/testing-standards.md)** 📝
  单元测试、集成测试、E2E测试规范

- **[安全开发规范](standards/security-standards.md)** ✅
  安全编码和常见漏洞防范

### 2. 最佳实践（Best Practices）

#### 2.1 开发最佳实践
- **[后端开发最佳实践](best-practices/backend-best-practices.md)** ✅
  Java/Spring Boot开发最佳实践

- **[前端开发最佳实践](best-practices/frontend-best-practices.md)** ✅
  Vue.js/TypeScript开发最佳实践

#### 2.2 设计最佳实践
- **[API设计最佳实践](best-practices/api-best-practices.md)** ✅
  API设计模式和常见场景

- **[数据库设计最佳实践](best-practices/database-best-practices.md)** ✅
  数据库设计模式和优化策略

#### 2.3 质量最佳实践
- **[测试最佳实践](best-practices/testing-best-practices.md)** ✅
  测试策略和测试金字塔

- **[性能优化实践](best-practices/performance-optimization.md)** ✅
  前后端性能优化技巧

- **[安全最佳实践](best-practices/security-best-practices.md)** ✅
  安全防护和加密策略

### 3. 代码模板（Templates）

#### 3.1 后端模板
- **[Service模板](templates/backend-patterns/service-template.md)** 📝
  Spring Boot Service层代码模板

- **[Controller模板](templates/backend-patterns/controller-template.md)** 📝
  Spring Boot Controller层代码模板

- **[Repository模板](templates/backend-patterns/repository-template.md)** 📝
  JPA Repository接口模板

- **[REST API模板](templates/backend-patterns/rest-api-template.md)** ✅
  RESTful API设计模板

- **[数据库命名规范](templates/backend-patterns/db-design/database-naming-conventions.md)** ✅
  数据库表、字段、索引命名规范

#### 3.2 前端模板
- **[Vue组件模板](templates/frontend-patterns/component-template.md)** 📝
  Vue 3 Composition API组件模板

- **[页面模板](templates/frontend-patterns/page-template.md)** 📝
  Vue页面组件模板

- **[Store模板](templates/frontend-patterns/store-template.md)** 📝
  Pinia状态管理模板

#### 3.3 测试模板
- **[单元测试模板](templates/testing-patterns/unit-test-template.md)** 📝
  JUnit/Vitest单元测试模板

- **[集成测试模板](templates/testing-patterns/integration-test-template.md)** 📝
  API集成测试模板（pytest）

- **[E2E测试模板](templates/testing-patterns/e2e-test-template.md)** 📝
  端到端测试模板（Playwright）

#### 3.4 产品模板
- **[PRD模板](templates/product-templates/prd-template.md)** ✅
  完整的产品需求文档模板

### 4. 领域知识（Domain）

- **[DDD实践指南](domain/ddd-practices.md)** 📝
  领域驱动设计实践

- **[领域建模指南](domain/domain-modeling.md)** 📝
  领域模型设计方法

### 5. UI设计规范（UI Guidelines）

- **[UI设计规范](ui-guidelines/ui-design-standards.md)** ✅
  界面设计统一规范

- **[交互设计规范](ui-guidelines/interaction-design.md)** ✅
  交互设计原则和模式

### 6. Skills使用指南

- **[Skills总览](skills-guide/README.md)** ✅
  所有自定义技能的使用指南

- **[资深PM使用指南](skills-guide/senior-pm-guide.md)** ✅
  /senior-pm技能详细使用说明

- **[资深PM演示](skills-guide/senior-pm-demo.md)** ✅
  完整的使用演示案例

## 🎯 使用指南

### 新加入项目

1. **阅读编码规范**（必读）
   - Git提交规范
   - 对应语言的编码规范（Java/TypeScript/Python）
   - API设计规范

2. **了解最佳实践**（推荐）
   - 后端/前端开发最佳实践
   - 测试最佳实践

3. **使用代码模板**（实践）
   - 根据模板快速生成标准化代码

### 开发新功能

1. **产品设计阶段**
   → 参考 [PRD模板](templates/product-templates/prd-template.md)
   → 使用 `/senior-pm` 技能生成PRD

2. **技术设计阶段**
   → 参考 [API设计规范](standards/api-design-standards.md)
   → 参考 [数据库设计规范](standards/database-design-standards.md)
   → 使用 `/tech-design` 技能生成技术设计

3. **代码实现阶段**
   → 参考对应的代码模板
   → 遵循编码规范
   → 使用 `/implement` 技能生成代码

4. **代码审查阶段**
   → 参考 [代码审查规范](standards/code-review-standards.md)

5. **测试阶段**
   → 参考 [测试规范](standards/testing-standards.md)
   → 参考测试模板

## 📖 文档状态说明

- ✅ **已完成**：文档已编写完成
- 📝 **待完成**：文档规划中，即将创建
- 🔄 **更新中**：文档正在更新优化

## 🔄 持续更新

知识库会持续更新和完善：

- **每次项目交付后**：总结经验教训，更新最佳实践
- **发现新模式时**：添加到相应的模板库
- **遇到问题时**：记录解决方案，更新规范
- **技术栈升级时**：更新相关文档

## 💡 贡献指南

### 更新现有文档

如果发现文档中的问题或有改进建议：
1. 直接修改对应的文档
2. 在文档顶部添加修订记录
3. 提交Git commit，说明修改原因

### 添加新文档

如果需要添加新的知识库文档：
1. 确定文档类型（标准/最佳实践/模板/领域知识）
2. 放到对应的目录下
3. 更新本README.md的索引
4. 遵循现有文档的格式和风格

### 文档格式要求

- 使用Markdown格式
- 包含目录（使用`[TOC]`或手动创建）
- 代码示例要完整可运行
- 提供正反对比示例（✅ 好的 vs ❌ 不好的）
- 包含实际场景的使用案例

## 📞 反馈与建议

如有任何问题或建议，请：
- 在项目中创建Issue
- 在团队会议上提出
- 直接修改文档并提交PR

---

**记住**：好的规范是团队智慧的结晶，需要大家共同维护和完善！💪
