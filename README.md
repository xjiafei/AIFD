# AI全流程研发框架 (AI-Driven Full-Stack Development Framework)

这是一个基于Claude Code的AI驱动全流程研发框架，旨在自动化从需求到部署的完整软件开发生命周期。

## 项目概述

本框架通过Claude Code的技能(Skills)、代理(Agents)和MCP能力，协调产品设计、技术设计、开发、测试和部署的全流程自动化。

### 技术栈

- **后端**: Java + Spring Boot (Maven多模块项目)
- **前端**: Vue.js + TypeScript
- **测试**: Python (pytest)
- **部署**: Docker + Kubernetes

## 快速开始

### 1. 初始化项目

框架已经初始化完成，目录结构已创建。

### 2. 初始化Git仓库

为每个代码仓库初始化Git：

```bash
# 后端仓库
cd repos/backend
git init
git remote add origin <your-backend-repo-url>

# 前端仓库
cd repos/frontend
git init
git remote add origin <your-frontend-repo-url>

# 测试仓库
cd repos/testing
git init
git remote add origin <your-testing-repo-url>

# DevOps仓库
cd repos/devops
git init
git remote add origin <your-devops-repo-url>
```

### 3. 开发第一个功能

```bash
# 1. 启动产品设计流程
/design-feature user-authentication

# 2. 审核产品设计文档后，启动技术设计
/tech-design user-authentication

# 3. 审核技术设计文档后，开始实现
/implement user-authentication
```

## 工作流程

### 阶段一：产品设计
- 输入：用户需求描述
- 输出：需求文档、PRD、用户故事、原型设计
- 技能：`/design-feature <feature-name>`

### 阶段二：技术设计
- 输入：已批准的产品设计
- 输出：架构设计、API设计、前端设计、测试设计、部署计划
- 技能：`/tech-design <feature-name>`

### 阶段三：代码实现
- 输入：已批准的技术设计
- 输出：后端代码、前端代码、测试代码、部署脚本
- 技能：`/implement <feature-name>`

### 阶段四：审核与部署
- 人工代码审核
- 测试执行
- 部署到目标环境

## 目录结构

```
.
├── .claude/              # Claude Code配置
│   ├── skills/           # 自定义技能
│   ├── agents/           # 专用代理
│   ├── commands/         # 自定义命令
│   └── hooks/            # Git钩子
├── docs/                 # 所有文档
│   ├── specs/            # 项目基线规格
│   ├── feature*-specs/   # 增量功能规格
│   └── knowledges/       # 知识库
│       ├── templates/    # 模板
│       ├── standards/    # 编码规范
│       └── domain/       # 领域知识
├── workspace/            # AI临时工作区
└── repos/                # 代码仓库
    ├── backend/          # 后端代码
    ├── frontend/         # 前端代码
    ├── testing/          # 测试代码
    └── devops/           # 部署脚本
```

## 自定义技能

- `/design-feature` - 产品设计工作流
- `/tech-design` - 技术设计工作流
- `/implement` - 代码实现工作流
- `/create-spec` - 创建功能规格目录

## 知识库

- **编码规范**: `docs/knowledges/standards/`
  - Java编码规范
  - 前端编码规范
  - Git提交规范

- **设计模板**: `docs/knowledges/templates/`
  - REST API设计模板
  - 数据库设计规范
  - 前端组件模板
  - 测试用例模板

## 重要说明

1. **人工审核关卡**：每个阶段完成后必须人工审核批准才能进入下一阶段
2. **独立Git仓库**：每个repos子目录是独立的Git仓库
3. **文档驱动**：所有开发工作基于规格文档进行
4. **标准遵循**：所有生成的代码必须遵循知识库中的编码规范

## 详细文档

查看 [CLAUDE.md](./CLAUDE.md) 获取完整的架构说明和命令参考。

## 许可证

[指定您的许可证]
