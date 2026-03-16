# Spec Kit 架构分析

## 一、整体架构

Spec Kit 采用**分层架构**设计，从底层到上层依次为：

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interface Layer                     │
│                    (CLI Commands & Output)                   │
├─────────────────────────────────────────────────────────────┤
│                    Orchestration Layer                       │
│              (Workflow & Command Handlers)                   │
├─────────────────────────────────────────────────────────────┤
│                    Template Layer                            │
│          (Command Templates, Artifact Templates)            │
├─────────────────────────────────────────────────────────────┤
│                    Extension Layer                           │
│            (Extensions, Presets, Hooks)                      │
├─────────────────────────────────────────────────────────────┤
│                    Core Layer                                │
│         (Agent Registration, File Management)               │
├─────────────────────────────────────────────────────────────┤
│                    Infrastructure Layer                      │
│      (GitHub API, File System, Shell Execution)             │
└─────────────────────────────────────────────────────────────┘
```

---

## 二、核心模块设计

### 2.1 CLI 入口模块 (`__init__.py`)

**职责**：
- 命令行解析和路由
- 用户交互界面
- 模板下载和项目初始化

**核心类**：

```python
class BannerGroup(TyperGroup):
    """自定义命令组，显示 Banner"""

class StepTracker:
    """进度跟踪器，用于显示层级步骤状态"""
    - add(key, label)      # 添加步骤
    - start(key, detail)   # 开始步骤
    - complete(key)        # 完成步骤
    - error(key)           # 错误状态
    - render()             # 渲染树形显示
```

**关键配置**：

```python
AGENT_CONFIG = {
    "claude": {
        "name": "Claude Code",
        "folder": ".claude/",
        "commands_subdir": "commands",
        "install_url": "...",
        "requires_cli": True,
    },
    # ... 其他 Agent 配置
}
```

### 2.2 Agent 注册模块 (`agents.py`)

**职责**：
- 统一的命令注册接口
- 支持多种 Agent 格式（Markdown, TOML）
- 参数占位符转换

**核心类**：

```python
class CommandRegistrar:
    """命令注册器"""

    AGENT_CONFIGS = {
        "claude": {"dir": ".claude/commands", "format": "markdown", ...},
        "gemini": {"dir": ".gemini/commands", "format": "toml", ...},
        # ...
    }

    def register_commands(agent_name, commands, source_id, ...):
        """为指定 Agent 注册命令"""

    def parse_frontmatter(content) -> tuple[dict, str]:
        """解析 YAML Frontmatter"""

    def render_markdown_command(...) -> str:
        """渲染 Markdown 格式命令"""

    def render_toml_command(...) -> str:
        """渲染 TOML 格式命令"""
```

### 2.3 扩展管理模块 (`extensions.py`)

**职责**：
- 扩展的安装、卸载、验证
- 扩展清单解析
- 命令和模板注册

**核心类**：

```python
class ExtensionManifest:
    """扩展清单验证器"""
    SCHEMA_VERSION = "1.0"
    REQUIRED_FIELDS = ["schema_version", "extension", "requires", "provides"]

class ExtensionManager:
    """扩展管理器"""
    - install(source, project_root)      # 安装扩展
    - uninstall(extension_id, ...)        # 卸载扩展
    - list_installed(project_root)        # 列出已安装
    - validate_compatibility(...)         # 验证兼容性
```

### 2.4 预设管理模块 (`presets.py`)

**职责**：
- 预设的安装、卸载、管理
- 模板覆盖机制
- 版本兼容性检查

**核心类**：

```python
class PresetManifest:
    """预设清单验证器"""
    SCHEMA_VERSION = "1.0"

class PresetManager:
    """预设管理器"""
    - install(source, project_root)       # 安装预设
    - uninstall(preset_id, ...)           # 卸载预设
    - get_template_resolution_stack(...)  # 获取模板解析栈
```

---

## 三、模板系统架构

### 3.1 模板层次结构

```
┌─────────────────────────────────────────┐
│            User's Templates             │  ← 最高优先级
│         (.specify/templates/)           │
├─────────────────────────────────────────┤
│            Preset Templates             │
│         (已安装的预设模板)               │
├─────────────────────────────────────────┤
│          Extension Templates            │
│         (已安装的扩展模板)               │
├─────────────────────────────────────────┤
│            Core Templates               │  ← 最低优先级
│         (spec-kit 核心模板)             │
└─────────────────────────────────────────┘
```

### 3.2 命令模板格式

```markdown
---
description: "命令描述"
handoffs:
  - label: "下一步操作"
    agent: speckit.next
    prompt: "执行提示"
scripts:
  sh: scripts/bash/script.sh "{ARGS}"
  ps: scripts/powershell/script.ps1 "{ARGS}"
---

## 命令内容

$ARGUMENTS  # 用户输入参数
{SCRIPT}    # 脚本路径占位符
```

### 3.3 工件模板类型

| 模板文件 | 用途 |
|---------|------|
| `spec-template.md` | 功能规格模板 |
| `plan-template.md` | 实现计划模板 |
| `tasks-template.md` | 任务列表模板 |
| `constitution-template.md` | 项目原则模板 |
| `checklist-template.md` | 检查清单模板 |
| `agent-file-template.md` | Agent 配置模板 |

---

## 四、扩展系统架构

### 4.1 扩展清单结构

```yaml
# extension.yml
schema_version: "1.0"

extension:
  id: "my-extension"
  name: "My Extension"
  version: "1.0.0"
  description: "扩展描述"
  author: "作者"
  repository: "Git 仓库 URL"

requires:
  speckit_version: ">=0.1.0"

provides:
  commands:
    - name: "speckit.myext.cmd"
      file: "commands/cmd.md"
      description: "命令描述"

  templates:
    - name: "my-template"
      file: "templates/my-template.md"
      description: "模板描述"

hooks:
  before_implement:
    - command: "speckit.myext.validate"
      optional: true
      prompt: "验证实现前状态"
```

### 4.2 钩子系统

```
┌─────────────────────────────────────────────────────────────┐
│                        Hook Points                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  before_tasks      →  任务生成前                            │
│  after_tasks       →  任务生成后                            │
│  before_implement  →  实现执行前                            │
│  after_implement   →  实现执行后                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 扩展发现机制

```
1. 扫描 .specify/extensions.yml
2. 解析已安装扩展列表
3. 加载每个扩展的 extension.yml
4. 注册命令和模板
5. 设置钩子
```

---

## 五、预设系统架构

### 5.1 预设 vs 扩展

| 特性 | 预设 (Preset) | 扩展 (Extension) |
|------|--------------|-----------------|
| 目的 | 定制模板和行为 | 添加新功能 |
| 命令 | 可以覆盖 | 可以添加和覆盖 |
| 模板 | 可以覆盖 | 可以添加和覆盖 |
| 钩子 | 不支持 | 支持 |
| 优先级 | 高于扩展 | 低于预设 |

### 5.2 预设清单结构

```yaml
# preset.yml
schema_version: "1.0"

preset:
  id: "my-preset"
  name: "My Preset"
  version: "1.0.0"

requires:
  speckit_version: ">=0.1.0"

provides:
  templates:
    - type: "template"
      name: "spec-template"
      file: "templates/spec-template.md"
      replaces: "spec-template"  # 覆盖核心模板
```

---

## 六、数据流分析

### 6.1 项目初始化流程

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ specify init │ ──▶ │ GitHub API   │ ──▶ │ Download ZIP │
└──────────────┘     └──────────────┘     └──────────────┘
                                                │
                                                ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Git Init     │ ◀── │ Merge/Write  │ ◀── │ Extract ZIP  │
└──────────────┘     └──────────────┘     └──────────────┘
```

### 6.2 命令执行流程

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ /speckit.cmd │ ──▶ │ Load Template│ ──▶ │ Run Script   │
└──────────────┘     └──────────────┘     └──────────────┘
                                                │
                                                ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Write Output │ ◀── │ AI Process   │ ◀── │ Parse JSON   │
└──────────────┘     └──────────────┘     └──────────────┘
```

---

## 七、设计模式应用

### 7.1 策略模式

用于支持多种 AI Agent 的命令格式：

```python
# 不同 Agent 使用不同的渲染策略
if agent_config["format"] == "markdown":
    output = self.render_markdown_command(...)
elif agent_config["format"] == "toml":
    output = self.render_toml_command(...)
```

### 7.2 模板方法模式

命令模板定义骨架，具体内容由 AI 填充：

```markdown
## Outline
1. Setup: Run `{SCRIPT}`
2. Load context: Read FEATURE_SPEC
3. Execute workflow: ...
```

### 7.3 责任链模式

模板解析栈形成责任链：

```
User Template → Preset Template → Extension Template → Core Template
     ↓              ↓                  ↓                   ↓
  找到了？ ──────▶ 没有？ ──────────▶ 没有？ ──────────▶ 使用核心模板
```

### 7.4 观察者模式

钩子系统实现观察者模式：

```python
# 扩展可以注册为观察者
hooks:
  before_implement:
    - command: "speckit.myext.validate"
```

---

## 八、架构优势

1. **可扩展性**: 通过扩展系统轻松添加新功能
2. **可定制性**: 预设系统允许深度定制
3. **多 Agent 支持**: 统一接口支持多种 AI Agent
4. **关注点分离**: 清晰的层次划分
5. **向后兼容**: 版本检查和兼容性验证