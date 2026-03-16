# Spec Kit 项目分析报告

**分析日期**: 2026-03-16
**项目来源**: GitHub (github/spec-kit)
**分析者**: Claude Code

---

## 一、项目概述

### 1.1 项目定位

**Spec Kit** 是 GitHub 开源的一个工具包，用于实践 **Spec-Driven Development（规格驱动开发）** 方法论。核心理念是将规格文档从"一次性脚手架"转变为"可执行的实现蓝图"。

### 1.2 核心价值主张

| 传统开发 | Spec-Driven Development |
|---------|------------------------|
| 代码为王，规格是临时的 | 规格为王，代码是生成的 |
| 先写代码，文档后补 | 先定义规格，AI 生成代码 |
| 需求变更需要手动改代码 | 需求变更新规格，重新生成 |
| 技术债务随时间累积 | 规格即文档，始终同步 |

### 1.3 项目元数据

```
项目名称: spec-kit
维护者: GitHub
开源协议: MIT
编程语言: Python 3.11+
包管理: uv / pip
CLI 工具: specify-cli
```

---

## 二、项目结构概览

```
spec-kit-main/
├── src/specify_cli/          # 核心 CLI 实现
│   ├── __init__.py           # 主入口和命令定义
│   ├── agents.py             # AI Agent 注册器
│   ├── extensions.py         # 扩展管理器
│   └── presets.py            # 预设管理器
│
├── templates/                 # 核心模板文件
│   ├── commands/             # 命令模板 (specify, plan, tasks, implement 等)
│   ├── spec-template.md      # 功能规格模板
│   ├── plan-template.md      # 实现计划模板
│   ├── tasks-template.md     # 任务列表模板
│   └── constitution-template.md # 项目原则模板
│
├── presets/                   # 预设配置
│   ├── scaffold/             # 脚手架预设
│   ├── self-test/            # 自测试预设
│   ├── catalog.json          # 官方预设目录
│   └── ARCHITECTURE.md       # 预设架构文档
│
├── extensions/                # 扩展系统
│   ├── selftest/             # 自测试扩展
│   ├── template/             # 扩展模板
│   └── EXTENSION-*.md        # 扩展开发文档
│
├── docs/                      # 文档
├── tests/                     # 测试
└── scripts/                   # 辅助脚本
```

---

## 三、核心组件

### 3.1 Specify CLI (`src/specify_cli/`)

CLI 工具的核心功能：

| 命令 | 功能 |
|------|------|
| `specify init` | 初始化新项目或现有项目 |
| `specify check` | 检查系统工具是否安装 |

**关键特性**:
- 支持多种 AI Agent（Claude, Gemini, Cursor, Copilot 等 20+）
- 自动从 GitHub Release 下载模板
- 支持跨平台（Linux/macOS/Windows）
- 支持交互式 AI 选择

### 3.2 命令系统 (`templates/commands/`)

| 命令 | 描述 | 前置依赖 |
|------|------|---------|
| `/speckit.constitution` | 创建项目原则 | 无 |
| `/speckit.specify` | 创建功能规格 | constitution |
| `/speckit.clarify` | 澄清规格细节 | specify |
| `/speckit.plan` | 创建实现计划 | specify |
| `/speckit.tasks` | 生成任务列表 | plan |
| `/speckit.implement` | 执行实现 | tasks |
| `/speckit.analyze` | 一致性分析 | tasks |
| `/speckit.checklist` | 生成检查清单 | specify |

### 3.3 扩展系统 (`extensions/`)

允许第三方开发者创建自定义命令和模板：

- **扩展清单**: `extension.yml`
- **命令注册**: 自动注入到 AI Agent 目录
- **钩子系统**: before/after 钩子

### 3.4 预设系统 (`presets/`)

允许自定义模板覆盖：

- **预设清单**: `preset.yml`
- **模板覆盖**: 可以覆盖核心模板或扩展模板
- **优先级**: 预设 > 扩展 > 核心

---

## 四、支持的 AI Agent

| Agent | CLI 工具 | 命令目录 | 格式 |
|-------|---------|---------|------|
| Claude Code | `claude` | `.claude/commands/` | Markdown |
| Gemini CLI | `gemini` | `.gemini/commands/` | TOML |
| GitHub Copilot | - | `.github/agents/` | Markdown |
| Cursor | - | `.cursor/commands/` | Markdown |
| Codex CLI | `codex` | `.codex/prompts/` | Markdown |
| Windsurf | - | `.windsurf/workflows/` | Markdown |
| Qwen Code | `qwen` | `.qwen/commands/` | Markdown |
| ... | ... | ... | ... |

---

## 五、工作流程

### 5.1 Spec-Driven Development 流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    Spec-Driven Development                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Constitution    →  建立项目原则和开发规范                     │
│         ↓                                                        │
│  2. Specify         →  定义"要做什么"（功能规格）                 │
│         ↓                                                        │
│  3. Clarify         →  澄清模糊点（可选）                         │
│         ↓                                                        │
│  4. Plan            →  制定技术实现计划                           │
│         ↓                                                        │
│  5. Tasks           →  生成可执行任务清单                         │
│         ↓                                                        │
│  6. Implement       →  AI 自动实现代码                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 文件产出

每个阶段产出的文件：

```
.specify/
├── memory/
│   └── constitution.md      # 项目原则
├── specs/
│   └── 001-feature-name/
│       ├── spec.md          # 功能规格
│       ├── plan.md          # 实现计划
│       ├── tasks.md         # 任务列表
│       ├── data-model.md    # 数据模型
│       ├── research.md      # 研究笔记
│       ├── quickstart.md    # 快速验证
│       └── contracts/       # 接口契约
└── templates/               # 模板文件
```

---

## 六、技术实现要点

### 6.1 CLI 实现

- **框架**: Typer (Python CLI 框架)
- **UI**: Rich (终端美化)
- **HTTP**: httpx (GitHub API 调用)
- **交互**: readchar (键盘输入)

### 6.2 模板系统

- **格式**: Markdown + YAML Frontmatter
- **变量**: `$ARGUMENTS`, `{SCRIPT}`, `{AGENT_SCRIPT}`
- **脚本**: 支持 Bash 和 PowerShell

### 6.3 扩展机制

- **发现**: 扫描 `.specify/extensions.yml`
- **注册**: `CommandRegistrar` 类
- **钩子**: before_implement, after_tasks 等

---

## 七、项目特点总结

### 优势

1. **方法论驱动**: 提供完整的开发方法论，而非仅仅工具
2. **多 Agent 支持**: 支持几乎所有主流 AI 编程助手
3. **可扩展**: 完善的扩展和预设系统
4. **结构化**: 从规格到实现的完整链路
5. **企业级**: GitHub 官方维护，质量有保障

### 适用场景

- 从零开始的新项目（Greenfield）
- 遗留系统现代化（Brownfield）
- 多方案探索（Creative Exploration）
- 企业级应用开发

---

## 八、相关文档

- [02-architecture-analysis.md](./02-architecture-analysis.md) - 架构详细分析
- [03-cli-implementation.md](./03-cli-implementation.md) - CLI 实现详解
- [04-template-system.md](./04-template-system.md) - 模板系统分析
- [05-extension-system.md](./05-extension-system.md) - 扩展系统分析
- [06-preset-system.md](./06-preset-system.md) - 预设系统分析
- [07-workflow-analysis.md](./07-workflow-analysis.md) - 工作流程详解
- [08-architecture-diagrams.md](./08-architecture-diagrams.md) - 架构图