---
description: 使用方案模板执行实施方案规划工作流，生成设计产物。
handoffs: 
  - label: 创建任务
    agent: speckit.tasks
    prompt: 将方案拆解为具体任务
    send: true
  - label: 创建检查清单
    agent: speckit.checklist
    prompt: 为以下领域创建检查清单...
---

## 用户输入

```text
$ARGUMENTS
```

继续操作前，你**必须**参考用户输入（若不为空）。

## 大纲

1. **环境搭建**：从代码仓库根目录运行 `.specify/scripts/powershell/setup-plan.ps1 -Json`，并解析 JSON 以获取 FEATURE_SPEC（功能规格）、IMPL_PLAN（实施方案）、SPECS_DIR（规格目录）、BRANCH（分支）。若参数中包含单引号（如 "I'm Groot"），需使用转义语法：例如 'I'\''m Groot'（或尽可能使用双引号："I'm Groot"）。

2. **加载上下文**：读取 FEATURE_SPEC 和 `.specify/memory/constitution.md` 文件。加载 IMPL_PLAN 模板（已完成复制）。

3. **执行方案工作流**：按照 IMPL_PLAN 模板的结构执行以下操作：
   - 填写技术背景部分（未知内容标记为“待确认（NEEDS CLARIFICATION）”）
   - 根据合规性文件填写合规性检查部分
   - 评估检查项（若违规项无合理解释则触发错误）
   - 0 阶段：生成 research.md 文件（解决所有“待确认”项）
   - 1 阶段：生成 data-model.md、contracts/ 目录、quickstart.md 文件
   - 1 阶段：运行智能体脚本更新智能体上下文
   - 设计完成后重新评估合规性检查

4. **停止并报告**：命令在 2 阶段规划完成后结束。报告分支、IMPL_PLAN 路径及已生成的产物。

## 阶段说明

### 0 阶段：大纲与调研

1. **从上述技术背景中提取未知项**：
   - 每个“待确认”项 → 对应调研任务
   - 每个依赖项 → 对应最佳实践调研任务
   - 每个集成项 → 对应模式调研任务

2. **生成并分派调研智能体任务**：

   ```text
   针对技术背景中的每个未知项：
     任务：“为{功能上下文}调研{未知项}”
   针对每个技术选型：
     任务：“查找{领域}中{技术}的最佳实践”
   ```

3. **将调研结果整理到 `research.md` 文件中**，格式如下：
   - 决策：[选定方案]
   - 理由：[选择依据]
   - 备选方案：[其他评估过的方案]

**输出**：包含所有“待确认”项解决方案的 research.md 文件

### 1 阶段：设计与契约

**前提条件**：`research.md` 文件已完成

1. **从功能规格中提取实体** → 写入 `data-model.md`：
   - 实体名称、字段、关系
   - 需求中的验证规则
   - 适用的状态转换逻辑

2. **根据功能需求生成 API 契约**：
   - 每个用户操作 → 对应接口端点
   - 采用标准 REST/GraphQL 模式
   - 将 OpenAPI/GraphQL 架构输出到 `/contracts/` 目录

3. **智能体上下文更新**：
   - 运行 `.specify/scripts/powershell/update-agent-context.ps1 -AgentType claude`
   - 这些脚本会检测当前使用的 AI 智能体类型
   - 更新对应的智能体专属上下文文件
   - 仅添加当前方案中的新技术内容
   - 保留标记之间的手动添加内容

**输出**：data-model.md 文件、/contracts/ 目录下文件、quickstart.md 文件、智能体专属文件

## 核心规则

- 使用绝对路径
- 若检查项未通过或“待确认”项未解决，触发错误（ERROR）