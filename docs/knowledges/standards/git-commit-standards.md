# Git提交规范

## 提交消息格式

```
<type>: [feature-{N}] <subject>

<body>

<footer>
```

### Type类型

- **feat**: 新功能
- **fix**: 缺陷修复
- **docs**: 文档更新
- **test**: 测试相关

### 示例

```
feat: [feature-001] 实现用户登录功能

- 添加登录API端点
- 实现JWT token生成
- 添加用户认证中间件

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

## 重要规则

1. 每个提交必须关联feature编号
2. Subject使用祈使句，不超过50字符
3. Body详细说明改动内容和原因
4. 每个仓库独立提交
5. 提交前确保测试通过
