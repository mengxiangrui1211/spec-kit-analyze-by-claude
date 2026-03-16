# Spec Kit 工作流程详解

## 一、Spec-Driven Development 核心理念

### 1.1 传统开发 vs Spec-Driven Development

```
传统开发流程:
需求 → 设计 → 编码 → 测试 → 部署
        ↓
     规格文档（一次性，容易过时）

Spec-Driven Development:
需求 → 规格 → 计划 → 任务 → 实现
        ↓        ↓       ↓       ↓
    (活文档)  (技术决策) (执行清单) (AI生成代码)
        ↓
    始终与代码同步
```

### 1.2 核心原则

1. **意图驱动**: 先定义"做什么"（WHAT）和"为什么"（WHY），再考虑"怎么做"（HOW）
2. **规格可执行**: 规格文档直接驱动代码生成
3. **多步精化**: 不追求一次完成，分阶段逐步细化
4. **AI 增强**: 充分利用 AI 模型的理解能力

---

## 二、完整工作流程

### 2.1 流程概览

```
┌─────────────────────────────────────────────────────────────────┐
│                    Spec-Driven Development                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1: Constitution   →  建立项目原则                          │
│           ↓                                                      │
│  STEP 2: Specify        →  创建功能规格                          │
│           ↓                                                      │
│  STEP 3: Clarify        →  澄清模糊点（可选）                    │
│           ↓                                                      │
│  STEP 4: Plan           →  制定技术计划                          │
│           ↓                                                      │
│  STEP 5: Tasks          →  生成任务清单                          │
│           ↓                                                      │
│  STEP 6: Implement      →  AI 执行实现                           │
│           ↓                                                      │
│  STEP 7: Validate       →  验证和迭代                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 文件产出时间线

```
STEP 1: Constitution
    └── .specify/memory/constitution.md

STEP 2: Specify
    ├── .specify/specs/001-feature-name/spec.md
    └── .specify/specs/001-feature-name/checklists/requirements.md

STEP 4: Plan
    ├── .specify/specs/001-feature-name/plan.md
    ├── .specify/specs/001-feature-name/research.md
    ├── .specify/specs/001-feature-name/data-model.md
    ├── .specify/specs/001-feature-name/contracts/
    └── .specify/specs/001-feature-name/quickstart.md

STEP 5: Tasks
    └── .specify/specs/001-feature-name/tasks.md

STEP 6: Implement
    └── src/ (实际代码)
```

---

## 三、各阶段详解

### 3.1 STEP 1: Constitution（建立项目原则）

**目的**: 定义项目的开发原则和技术决策标准

**命令**:
```bash
/speckit.constitution Create principles focused on code quality, testing standards, user experience consistency, and performance requirements
```

**产出**: `.specify/memory/constitution.md`

**内容结构**:
```markdown
# Project Constitution

## Core Principles
- 原则 1: 描述
- 原则 2: 描述

## Code Quality Standards
- 编码规范
- 代码审查要求

## Testing Standards
- 测试覆盖率要求
- 测试类型分布

## Performance Requirements
- 响应时间标准
- 资源使用限制

## User Experience Guidelines
- 一致性要求
- 可访问性标准
```

**重要性**:
- 作为后续所有决策的参考依据
- AI 在生成计划时会检查宪法约束
- 确保整个项目的一致性

### 3.2 STEP 2: Specify（创建功能规格）

**目的**: 定义"要做什么"，不涉及技术实现

**命令**:
```bash
/speckit.specify Build an application that can help me organize my photos in separate photo albums. Albums are grouped by date and can be re-organized by dragging and dropping on the main page.
```

**执行流程**:

```
1. 解析功能描述
   ├── 提取关键概念
   └── 生成简短名称 (photo-albums)

2. 创建功能分支
   ├── 001-photo-albums
   └── 初始化 spec.md

3. 填充规格模板
   ├── 用户故事 (P1, P2, P3)
   ├── 功能需求 (FR-001, FR-002...)
   └── 成功标准 (SC-001, SC-002...)

4. 质量验证
   ├── 创建检查清单
   └── 处理 [NEEDS CLARIFICATION]
```

**产出**: `.specify/specs/001-photo-albums/spec.md`

**关键规则**:
- 聚焦 WHAT 和 WHY
- 避免 HOW（技术实现）
- 最多 3 个 [NEEDS CLARIFICATION]
- 成功标准可测量、技术无关

### 3.3 STEP 3: Clarify（澄清规格）

**目的**: 在技术规划前解决模糊点

**命令**:
```bash
/speckit.clarify
```

**触发场景**:
- spec.md 中存在 [NEEDS CLARIFICATION] 标记
- 规格描述有歧义
- 边界条件不明确

**输出格式**:
```markdown
## Question 1: 用户认证方式

**Context**: "用户可以登录系统"

**What we need to know**: 使用哪种认证方式？

**Suggested Answers**:

| Option | Answer | Implications |
|--------|--------|--------------|
| A      | Email/Password | 需要密码重置流程 |
| B      | OAuth2 | 需要第三方集成 |
| C      | SSO | 依赖企业系统 |

**Your choice**: _
```

### 3.4 STEP 4: Plan（制定技术计划）

**目的**: 将功能需求转化为技术方案

**命令**:
```bash
/speckit.plan The application uses Vite with React. Images are not uploaded anywhere and metadata is stored in a local SQLite database.
```

**执行流程**:

```
Phase 0: 研究
├── 提取技术未知项
├── 生成研究任务
└── 输出 research.md

Phase 1: 设计
├── 提取数据实体 → data-model.md
├── 定义接口契约 → contracts/
├── 生成快速验证 → quickstart.md
└── 更新 Agent 上下文
```

**产出**:
```
.specify/specs/001-photo-albums/
├── plan.md           # 实现计划（技术栈、架构）
├── research.md       # 技术决策和原因
├── data-model.md     # 数据模型
├── contracts/        # API 契约
│   └── api-spec.json
└── quickstart.md     # 快速验证场景
```

### 3.5 STEP 5: Tasks（生成任务清单）

**目的**: 将计划分解为可执行的任务

**命令**:
```bash
/speckit.tasks
```

**任务组织原则**:
- 按用户故事分组
- 每个故事可独立测试
- 明确依赖关系
- 标记可并行任务

**产出**: `.specify/specs/001-photo-albums/tasks.md`

**结构**:
```markdown
## Phase 1: Setup
- [ ] T001 Create project structure
- [ ] T002 Initialize React project with Vite

## Phase 2: Foundational
- [ ] T003 Setup SQLite database
- [ ] T004 [P] Configure routing

## Phase 3: User Story 1 - Browse Albums (P1)
- [ ] T005 [P] [US1] Create Album model
- [ ] T006 [US1] Implement AlbumService
- [ ] T007 [US1] Create AlbumList component

## Phase 4: User Story 2 - Create Album (P2)
- [ ] T008 [P] [US2] Create AlbumForm component
- [ ] T009 [US2] Implement album creation

## Dependencies
- Phase 2 depends on Phase 1
- Phase 3+ depend on Phase 2
- User stories can proceed in parallel after foundation
```

### 3.6 STEP 6: Implement（执行实现）

**目的**: AI 按任务清单生成代码

**命令**:
```bash
/speckit.implement
```

**执行流程**:

```
1. 前置检查
   ├── 检查扩展钩子
   └── 检查清单状态

2. 加载上下文
   ├── tasks.md
   ├── plan.md
   └── data-model.md

3. 项目设置验证
   └── 创建/验证 .gitignore

4. 阶段执行
   ├── Phase 1: Setup
   ├── Phase 2: Foundational
   ├── Phase 3+: User Stories
   └── Phase N: Polish

5. 完成验证
   └── 所有任务标记 [X]
```

**任务执行规则**:
- 顺序任务按序执行
- `[P]` 标记的可并行执行
- `[US1]` 标记属于特定用户故事
- 完成后标记 `[X]`

---

## 四、开发场景适配

### 4.1 Greenfield（从零开始）

```
完整流程:
Constitution → Specify → Clarify → Plan → Tasks → Implement

特点:
- 完整体验 Spec-Driven Development
- 所有阶段都需要
- 输出完整的项目结构
```

### 4.2 Brownfield（遗留系统扩展）

```
简化流程:
Specify → Plan (结合现有架构) → Tasks → Implement

特点:
- constitution 可能已隐含在现有代码中
- plan 阶段需要分析现有架构
- tasks 需要考虑与现有代码的集成
```

### 4.3 Creative Exploration（多方案探索）

```
分支流程:
Specify → Plan (方案A) → Tasks → Implement
       → Plan (方案B) → Tasks → Implement
       → Plan (方案C) → Tasks → Implement

特点:
- 同一规格，多种技术实现
- 用于技术选型验证
- 最终选择最优方案
```

---

## 五、最佳实践

### 5.1 规格编写

1. **使用业务语言**: 避免技术术语
2. **具体而非抽象**: "用户可以上传 5MB 以内的图片" 而非 "用户可以上传文件"
3. **可测试**: 每个需求都应有明确的验证方式
4. **边界清晰**: 明确说明功能范围

### 5.2 计划制定

1. **技术无关的规格 → 技术相关的计划**: 保持关注点分离
2. **研究先行**: 不确定的技术选型先研究
3. **契约优先**: 先定义接口，再实现

### 5.3 任务分解

1. **用户故事优先级**: P1 是 MVP
2. **独立测试**: 每个故事完成后可独立验证
3. **并行标记**: 充分利用 `[P]` 标记
4. **文件路径**: 每个任务指定明确的文件位置

### 5.4 实现执行

1. **检查清单先行**: 确保 tasks.md 完整
2. **阶段验证**: 每个 Phase 完成后检查
3. **增量提交**: 每个任务或任务组完成后提交
4. **错误处理**: 遇到问题及时调整 tasks.md