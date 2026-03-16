# Spec Kit 项目分析文档索引

**项目**: spec-kit (GitHub 官方开源项目)
**分析日期**: 2026-03-16
**文档位置**: `neoway-spec-coding-analyze/`

---

## 文档列表

| 编号 | 文档名称 | 内容概述 |
|------|---------|---------|
| 01 | [项目概述](./01-project-overview.md) | 项目定位、结构、核心组件、支持 Agent |
| 02 | [架构分析](./02-architecture-analysis.md) | 分层架构、模块设计、设计模式 |
| 03 | [CLI 实现](./03-cli-implementation.md) | 命令实现、Agent 配置、GitHub API 集成 |
| 04 | [模板系统](./04-template-system.md) | 命令模板、工件模板、覆盖机制 |
| 05 | [扩展系统](./05-extension-system.md) | 扩展清单、钩子系统、命令注册 |
| 06 | [预设系统](./06-preset-system.md) | 预设清单、模板覆盖、解析栈 |
| 07 | [工作流程](./07-workflow-analysis.md) | SDD 流程、各阶段详解、最佳实践 |
| 08 | [架构图集](./08-architecture-diagrams.md) | Mermaid 图表（流程图、序列图等） |

---

## 快速导航

### 想要了解...

- **项目是什么？** → [01-项目概述](./01-project-overview.md)
- **整体架构如何？** → [02-架构分析](./02-architecture-analysis.md) + [08-架构图集](./08-architecture-diagrams.md)
- **CLI 如何实现？** → [03-CLI 实现](./03-cli-implementation.md)
- **模板如何工作？** → [04-模板系统](./04-template-system.md)
- **如何扩展功能？** → [05-扩展系统](./05-extension-system.md)
- **如何自定义模板？** → [06-预设系统](./06-preset-system.md)
- **如何使用项目？** → [07-工作流程](./07-workflow-analysis.md)

---

## 项目核心概念

```
┌─────────────────────────────────────────────────────────────┐
│                  Spec-Driven Development                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Constitution → Specify → Plan → Tasks → Implement         │
│       ↓             ↓         ↓        ↓         ↓          │
│    项目原则     功能规格   技术计划  任务清单   AI实现       │
│                                                              │
│   核心理念: 规格可执行，AI 生成代码                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 项目技术栈

| 类别 | 技术 |
|------|------|
| 编程语言 | Python 3.11+ |
| CLI 框架 | Typer |
| 终端美化 | Rich |
| HTTP 客户端 | httpx |
| 包管理 | uv |
| 配置格式 | YAML, JSON |

---

## 关键文件路径

```
spec-kit-main/
├── src/specify_cli/          # CLI 源码
│   ├── __init__.py           # 主入口 (~4000 行)
│   ├── agents.py             # Agent 注册器
│   ├── extensions.py         # 扩展管理器
│   └── presets.py            # 预设管理器
│
├── templates/                 # 模板文件
│   ├── commands/             # 命令模板 (9 个)
│   └── *-template.md         # 工件模板 (6 个)
│
├── presets/                   # 预设配置
│   ├── scaffold/             # 脚手架预设
│   └── self-test/            # 自测试预设
│
└── extensions/                # 扩展系统
    ├── selftest/             # 自测试扩展
    └── template/             # 扩展模板
```

---

## 支持的 AI Agent (20+)

| 类型 | Agent |
|------|-------|
| CLI 工具 | Claude Code, Gemini CLI, Qwen Code, Codex CLI, Kiro CLI, Amp, SHAI, Auggie CLI, Tabnine CLI, CodeBuddy, Qoder CLI, Mistral Vibe, Kimi Code, opencode |
| IDE 集成 | GitHub Copilot, Cursor, Windsurf, Roo Code, IBM Bob, Antigravity, Kilo Code |
| 自定义 | Generic (BYOA) |

---

## 学习建议

### 初学者路径

1. 阅读 [01-项目概述](./01-project-overview.md) 了解项目定位
2. 阅读 [07-工作流程](./07-workflow-analysis.md) 理解 SDD 方法论
3. 实践：安装 specify-cli 并初始化一个项目
4. 按流程执行：constitution → specify → plan → tasks → implement

### 开发者路径

1. 阅读 [02-架构分析](./02-architecture-analysis.md) 理解整体设计
2. 阅读 [03-CLI 实现](./03-cli-implementation.md) 了解实现细节
3. 阅读 [04-模板系统](./04-template-system.md) 理解模板机制
4. 阅读 [05-扩展系统](./05-extension-system.md) 学习扩展开发
5. 阅读 [06-预设系统](./06-preset-system.md) 学习模板定制

### 架构师路径

1. 阅读 [02-架构分析](./02-architecture-analysis.md) 的设计模式部分
2. 阅读 [08-架构图集](./08-architecture-diagrams.md) 查看各种视图
3. 分析扩展和预设系统的优先级机制
4. 评估在企业环境中的适用性

---

## 总结

Spec Kit 是一个实践 **Spec-Driven Development** 方法论的成熟工具包，由 GitHub 官方维护。其核心价值在于：

1. **方法论驱动** - 提供完整的从规格到代码的流程
2. **多 Agent 支持** - 几乎覆盖所有主流 AI 编程助手
3. **高度可扩展** - 完善的扩展和预设系统
4. **企业级质量** - GitHub 官方维护，文档完善

适合用于：
- 新项目快速启动（Greenfield）
- 遗留系统现代化（Brownfield）
- 技术方案探索（Creative Exploration）
- 企业级应用开发