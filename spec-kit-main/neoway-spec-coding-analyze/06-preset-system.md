# Spec Kit 预设系统分析

## 一、预设系统概述

预设 (Preset) 是 Spec Kit 中用于定制模板和行为的高级机制，允许组织或个人覆盖核心模板和命令，实现开发流程的标准化。

---

## 二、预设 vs 扩展

| 维度 | 预设 (Preset) | 扩展 (Extension) |
|------|--------------|-----------------|
| **目的** | 定制现有行为 | 添加新功能 |
| **命令** | 可以覆盖 | 可以添加和覆盖 |
| **模板** | 可以覆盖 | 可以添加和覆盖 |
| **钩子** | 不支持 | 支持 |
| **优先级** | 高于扩展 | 低于预设 |
| **典型场景** | 企业标准、团队规范 | 新功能、集成 |

**核心区别**：
- **扩展** = 添加新能力（做加法）
- **预设** = 覆盖默认行为（做定制）

---

## 三、预设目录结构

```
presets/
├── catalog.json              # 官方预设目录
├── catalog.community.json    # 社区预设目录
├── ARCHITECTURE.md           # 预设架构文档
├── PUBLISHING.md             # 发布指南
├── README.md                 # 预设系统说明
│
├── scaffold/                 # 脚手架预设（示例）
│   ├── preset.yml            # 预设清单
│   ├── README.md             # 预设说明
│   ├── commands/             # 命令覆盖
│   │   ├── speckit.specify.md
│   │   └── speckit.myext.myextcmd.md
│   └── templates/            # 模板覆盖
│       ├── spec-template.md
│       └── myext-template.md
│
└── self-test/                # 自测试预设
    ├── preset.yml
    ├── commands/
    └── templates/
```

---

## 四、预设清单格式

### 4.1 preset.yml 完整结构

```yaml
schema_version: "1.0"            # 必填：清单版本

preset:
  id: "my-preset"                # 必填：唯一标识符
  name: "My Preset"              # 必填：显示名称
  version: "1.0.0"               # 必填：语义版本号
  description: "预设描述"         # 必填：简短描述
  author: "Your Name"            # 必填：作者
  repository: "https://..."      # 必填：代码仓库
  license: "MIT"                 # 必填：许可证

requires:                        # 依赖要求
  speckit_version: ">=0.1.0"     # Spec Kit 版本要求

provides:                        # 提供的内容
  templates:                     # 模板覆盖列表
    - type: "template"           # 类型：template/command/script
      name: "spec-template"      # 模板名称
      file: "templates/spec-template.md"  # 文件路径
      description: "自定义规格模板"
      replaces: "spec-template"  # 要覆盖的目标

    - type: "command"
      name: "speckit.specify"
      file: "commands/speckit.specify.md"
      description: "自定义规格命令"
      replaces: "speckit.specify"

tags:                            # 标签（用于发现）
  - "enterprise"
  - "custom"
```

### 4.2 字段详解

#### `provides.templates`

```yaml
- type: "template"               # 类型
  name: "spec-template"          # 模板标识符
  file: "templates/spec.md"      # 相对于预设根目录
  description: "描述"            # 可选：说明
  replaces: "spec-template"      # 要覆盖的目标模板
```

**类型说明**：
- `template`: 工件模板 (spec-template.md, plan-template.md 等)
- `command`: 命令模板 (speckit.specify.md 等)
- `script`: 脚本模板 (保留用于未来)

#### `replaces` 字段

指定要覆盖的目标：
- 核心模板: `spec-template`, `plan-template` 等
- 扩展模板: `myext-template` (扩展提供的模板)

---

## 五、模板解析栈

### 5.1 优先级顺序

```
┌─────────────────────────────────────────┐
│      用户模板 (最高优先级)               │
│      .specify/templates/                 │
├─────────────────────────────────────────┤
│      预设模板                            │
│      已安装的预设提供的模板              │
├─────────────────────────────────────────┤
│      扩展模板                            │
│      已安装的扩展提供的模板              │
├─────────────────────────────────────────┤
│      核心模板 (最低优先级)               │
│      spec-kit 内置模板                   │
└─────────────────────────────────────────┘
```

### 5.2 解析流程

```python
def resolve_template(name: str) -> Path:
    """解析模板位置"""

    # 1. 检查用户自定义模板
    user_template = project_root / ".specify/templates" / f"{name}.md"
    if user_template.exists():
        return user_template

    # 2. 检查预设模板
    for preset in installed_presets:
        preset_template = preset.path / "templates" / f"{name}.md"
        if preset_template.exists():
            return preset_template

    # 3. 检查扩展模板
    for extension in installed_extensions:
        ext_template = extension.path / "templates" / f"{name}.md"
        if ext_template.exists():
            return ext_template

    # 4. 使用核心模板
    return core_templates / f"{name}.md"
```

---

## 六、预设管理器 API

### 6.1 PresetManager 类

```python
class PresetManager:
    def __init__(self, project_root: Path):
        self.project_root = project_root
        self.presets_file = project_root / ".specify" / "presets.yml"

    def install(self, source: str, project_root: Path = None) -> dict:
        """
        安装预设

        Args:
            source: 预设源（本地路径、ZIP URL、Git URL）

        Returns:
            安装信息字典
        """

    def uninstall(self, preset_id: str, project_root: Path = None) -> bool:
        """卸载预设"""

    def list_installed(self, project_root: Path = None) -> list[dict]:
        """列出已安装预设"""

    def get_template_resolution_stack(self) -> list[tuple[str, Path]]:
        """获取模板解析栈"""

    def resolve_template(self, name: str) -> Path:
        """解析模板位置"""
```

### 6.2 安装流程

```
1. 解析源地址
   ├── 本地目录: 直接使用
   ├── ZIP URL: 下载解压
   └── Git URL: 克隆仓库

2. 验证清单
   ├── 检查 preset.yml 存在
   ├── 验证必需字段
   └── 验证版本兼容性

3. 安装文件
   ├── 复制到 .specify/presets/<preset-id>/
   └── 更新 presets.yml

4. 验证覆盖
   └── 确认 replaces 目标存在

5. 清理临时文件
```

---

## 七、预设状态文件

### 7.1 presets.yml

```yaml
# .specify/presets.yml
presets:
  - id: "enterprise-standards"
    version: "1.0.0"
    installed_at: "2024-01-15T10:30:00Z"
    source: "https://github.com/company/spec-kit-preset-enterprise"
    enabled: true

  - id: "team-workflow"
    version: "2.1.0"
    installed_at: "2024-02-20T14:00:00Z"
    source: "./local/presets/team-workflow"
    enabled: true
```

---

## 八、预设目录（Catalog）

### 8.1 目录结构

```json
// catalog.json
{
  "schema_version": "1.0",
  "presets": [
    {
      "id": "enterprise-dotnet",
      "name": "Enterprise .NET Preset",
      "version": "1.0.0",
      "description": "企业级 .NET 开发预设",
      "repository": "https://github.com/company/spec-kit-preset-dotnet",
      "author": "Enterprise Team",
      "tags": ["enterprise", "dotnet", "c#"]
    }
  ]
}
```

### 8.2 目录栈优先级

```python
class PresetCatalogStack:
    def __init__(self):
        self.catalogs = [
            # 优先级从高到低
            PresetCatalogEntry(
                url="project-catalog.json",
                name="项目预设目录",
                priority=100,
                install_allowed=True
            ),
            PresetCatalogEntry(
                url="team-catalog.json",
                name="团队预设目录",
                priority=50,
                install_allowed=True
            ),
            PresetCatalogEntry(
                url="https://spec-kit.dev/catalog.json",
                name="官方预设目录",
                priority=10,
                install_allowed=True
            ),
        ]
```

---

## 九、典型使用场景

### 9.1 企业标准化

**场景**: 企业有统一的开发规范和模板

```yaml
# enterprise-preset/preset.yml
preset:
  id: "enterprise-standards"
  name: "Enterprise Standards"

provides:
  templates:
    # 覆盖规格模板，添加企业必需章节
    - type: "template"
      name: "spec-template"
      replaces: "spec-template"
      file: "templates/enterprise-spec-template.md"

    # 覆盖宪法模板，添加企业原则
    - type: "template"
      name: "constitution-template"
      replaces: "constitution-template"
      file: "templates/enterprise-constitution.md"
```

### 9.2 团队定制

**场景**: 特定团队有独特的工作流程

```yaml
# team-preset/preset.yml
preset:
  id: "frontend-team"
  name: "Frontend Team Preset"

provides:
  commands:
    # 覆盖实现命令，添加前端特定检查
    - type: "command"
      name: "speckit.implement"
      replaces: "speckit.implement"
      file: "commands/frontend-implement.md"
```

### 9.3 技术栈专用

**场景**: 针对特定技术栈的模板

```yaml
# react-preset/preset.yml
preset:
  id: "react-stack"
  name: "React Stack Preset"

provides:
  templates:
    - type: "template"
      name: "plan-template"
      replaces: "plan-template"
      file: "templates/react-plan-template.md"

  commands:
    - type: "command"
      name: "speckit.tasks"
      replaces: "speckit.tasks"
      file: "commands/react-tasks.md"
```

---

## 十、预设开发最佳实践

### 10.1 命名规范

- 预设 ID: 小写、连字符分隔
- 反映预设用途: `enterprise-standards`, `react-stack`

### 10.2 版本管理

- 使用语义版本号
- 声明 Spec Kit 版本要求
- 更新时检查兼容性

### 10.3 文档

```markdown
# My Preset

## 用途
描述预设的目标和适用场景

## 包含内容
- 覆盖的模板列表
- 覆盖的命令列表

## 安装
```bash
specify preset install my-preset
```

## 使用说明
如何使用预设提供的功能
```

### 10.4 测试

- 确保覆盖的模板能正常工作
- 测试与核心命令的兼容性
- 验证模板解析顺序