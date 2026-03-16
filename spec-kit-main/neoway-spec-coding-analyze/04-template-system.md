# Spec Kit 模板系统分析

## 一、模板系统概述

Spec Kit 的模板系统是其核心组件之一，定义了 Spec-Driven Development 工作流的各个阶段输入输出格式。

---

## 二、模板分类

### 2.1 命令模板 (Command Templates)

位于 `templates/commands/` 目录，定义 AI Agent 可执行的斜杠命令。

| 模板文件 | 对应命令 | 功能 |
|---------|---------|------|
| `constitution.md` | `/speckit.constitution` | 创建项目原则 |
| `specify.md` | `/speckit.specify` | 创建功能规格 |
| `clarify.md` | `/speckit.clarify` | 澄清规格细节 |
| `plan.md` | `/speckit.plan` | 创建实现计划 |
| `tasks.md` | `/speckit.tasks` | 生成任务列表 |
| `implement.md` | `/speckit.implement` | 执行实现 |
| `analyze.md` | `/speckit.analyze` | 一致性分析 |
| `checklist.md` | `/speckit.checklist` | 生成检查清单 |
| `taskstoissues.md` | `/speckit.taskstoissues` | 任务转 Issue |

### 2.2 工件模板 (Artifact Templates)

位于 `templates/` 根目录，定义各阶段产出文档的格式。

| 模板文件 | 用途 |
|---------|------|
| `spec-template.md` | 功能规格文档模板 |
| `plan-template.md` | 实现计划文档模板 |
| `tasks-template.md` | 任务列表文档模板 |
| `constitution-template.md` | 项目原则文档模板 |
| `checklist-template.md` | 检查清单文档模板 |
| `agent-file-template.md` | Agent 配置文件模板 |

---

## 三、命令模板格式详解

### 3.1 Frontmatter 结构

```yaml
---
description: "命令描述（显示在帮助信息中）"
handoffs:                        # 可选：下一步操作建议
  - label: "操作标签"
    agent: speckit.next_cmd      # 要执行的命令
    prompt: "执行提示"            # 自动填充的提示
    send: true                   # 是否自动发送
scripts:                         # 可选：要执行的脚本
  sh: scripts/bash/script.sh "{ARGS}"    # Bash 脚本
  ps: scripts/powershell/script.ps1 "{ARGS}"  # PowerShell 脚本
agent_scripts:                   # 可选：Agent 特定脚本
  sh: scripts/bash/update-agent.sh __AGENT__
  ps: scripts/powershell/update-agent.ps1 -AgentType __AGENT__
---
```

### 3.2 模板正文结构

```markdown
## User Input

```text
$ARGUMENTS
```

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root...
2. **Load context**: Read FEATURE_SPEC...
3. **Execute workflow**: ...

## Phases

### Phase 0: Outline & Research
...

### Phase 1: Design & Contracts
...

## Key rules

- Use absolute paths
- ERROR on gate failures
```

### 3.3 变量占位符

| 占位符 | 用途 | 示例 |
|-------|------|------|
| `$ARGUMENTS` | 用户输入参数 | 用户在命令后输入的文本 |
| `{ARGS}` | 脚本参数 | 传递给 shell 脚本 |
| `{SCRIPT}` | 脚本路径 | 解析后的完整脚本路径 |
| `{AGENT_SCRIPT}` | Agent 脚本路径 | Agent 特定的脚本 |
| `__AGENT__` | Agent 类型 | claude, gemini 等 |

---

## 四、核心命令模板分析

### 4.1 `/speckit.specify` - 创建功能规格

**执行流程**:

```
1. 解析用户输入
   ├── 提取功能描述
   └── 生成简短名称（2-4词）

2. 创建功能分支
   ├── 调用脚本: create-new-feature.sh --json --short-name "xxx"
   ├── 获取: BRANCH_NAME, SPEC_FILE
   └── 切换到新分支

3. 加载模板
   └── templates/spec-template.md

4. 生成规格内容
   ├── 提取关键概念（角色、动作、数据、约束）
   ├── 填充用户场景
   ├── 生成功能需求
   └── 定义成功标准

5. 质量验证
   ├── 创建检查清单
   ├── 验证完整性
   └── 处理 [NEEDS CLARIFICATION] 标记
```

**关键规则**:
- 最多 3 个 [NEEDS CLARIFICATION] 标记
- 聚焦于 **WHAT** 和 **WHY**，避免 **HOW**
- 成功标准必须可测量且技术无关

### 4.2 `/speckit.plan` - 创建实现计划

**执行流程**:

```
1. Setup
   └── 运行 setup-plan.sh --json

2. 加载上下文
   ├── 读取 spec.md
   └── 读取 constitution.md

3. Phase 0: 研究阶段
   ├── 提取未知项
   ├── 生成研究任务
   └── 输出 research.md

4. Phase 1: 设计阶段
   ├── 生成 data-model.md
   ├── 生成 contracts/
   ├── 生成 quickstart.md
   └── 更新 Agent 上下文
```

### 4.3 `/speckit.tasks` - 生成任务列表

**任务格式**:

```markdown
- [ ] [TaskID] [P?] [Story?] Description with file path
```

**格式组件**:

| 组件 | 说明 | 示例 |
|------|------|------|
| `- [ ]` | Markdown 复选框 | 必须 |
| TaskID | 任务编号 | T001, T002 |
| `[P]` | 可并行标记 | 可选 |
| `[US1]` | 用户故事标记 | 用户故事阶段必须 |
| 描述 | 包含文件路径 | Create User model in src/models/user.py |

**示例**:

```markdown
- [ ] T001 Create project structure per implementation plan
- [ ] T005 [P] Implement authentication middleware in src/middleware/auth.py
- [ ] T012 [P] [US1] Create User model in src/models/user.py
```

### 4.4 `/speckit.implement` - 执行实现

**执行流程**:

```
1. 前置检查
   ├── 检查扩展钩子
   └── 检查清单状态

2. 加载上下文
   ├── tasks.md (必需)
   ├── plan.md (必需)
   ├── data-model.md (可选)
   └── contracts/ (可选)

3. 项目设置验证
   ├── 检测 Git 仓库
   ├── 创建/验证 .gitignore
   └── 根据技术栈添加忽略模式

4. 任务执行
   ├── Phase-by-phase 执行
   ├── 尊重依赖关系
   ├── 并行任务 [P] 可同时运行
   └── 完成后标记 [X]

5. 完成验证
   └── 检查所有必需任务完成
```

---

## 五、工件模板分析

### 5.1 spec-template.md - 功能规格模板

**核心结构**:

```markdown
# Feature Specification: [FEATURE NAME]

## User Scenarios & Testing *(mandatory)*
### User Story 1 - [Brief Title] (Priority: P1)
- Why this priority
- Independent Test
- Acceptance Scenarios (Given/When/Then)

### Edge Cases

## Requirements *(mandatory)*
### Functional Requirements
- FR-001: System MUST [capability]

### Key Entities

## Success Criteria *(mandatory)*
### Measurable Outcomes
- SC-001: [Measurable metric]
```

**设计原则**:
- 用户故事按优先级排序 (P1, P2, P3)
- 每个故事可独立测试
- 使用 Given/When/Then 格式

### 5.2 tasks-template.md - 任务列表模板

**阶段结构**:

```
Phase 1: Setup (Shared Infrastructure)
    ↓
Phase 2: Foundational (Blocking Prerequisites)
    ↓
Phase 3+: User Stories (P1, P2, P3...)
    ↓
Phase N: Polish & Cross-Cutting Concerns
```

**依赖规则**:

```
Setup → Foundational → User Stories (可并行)
                         ↓
                       Polish
```

---

## 六、模板覆盖机制

### 6.1 覆盖优先级

```
用户模板 (.specify/templates/)
    ↓ 覆盖
预设模板 (已安装的预设)
    ↓ 覆盖
扩展模板 (已安装的扩展)
    ↓ 覆盖
核心模板 (spec-kit 内置)
```

### 6.2 自定义模板示例

```yaml
# preset.yml
provides:
  templates:
    - type: "template"
      name: "spec-template"
      file: "templates/spec-template.md"
      replaces: "spec-template"  # 声明要覆盖的模板
```

---

## 七、模板最佳实践

### 7.1 命令模板编写

1. **清晰的 Outline**: 列出执行步骤
2. **明确的 Phase**: 阶段化处理
3. **Key rules**: 列出关键规则
4. **错误处理**: 定义 ERROR 情况

### 7.2 工件模板编写

1. **必填/可选标记**: `(mandatory)` / `(optional)`
2. **占位符**: `[FEATURE NAME]`, `[DATE]`
3. **示例**: 提供填充示例
4. **注释**: 使用 `<!-- -->` 提供指导

### 7.3 格式规范

```markdown
<!-- 注释：指导 AI 如何填充 -->
<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right content.
-->

- **FR-001**: System MUST [specific capability]
*Example of marking unclear requirements:*
- **FR-006**: System MUST authenticate via [NEEDS CLARIFICATION: auth method]
```