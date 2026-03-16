# Spec Kit 扩展系统分析

## 一、扩展系统概述

Spec Kit 的扩展系统允许第三方开发者创建自定义命令和模板，在不修改核心代码的情况下扩展功能。

---

## 二、扩展目录结构

```
extensions/
├── selftest/                    # 自测试扩展示例
│   ├── extension.yml            # 扩展清单
│   ├── commands/                # 自定义命令
│   │   └── speckit.selftest.mycommand.md
│   └── templates/               # 自定义模板
│       └── my-template.md
│
├── template/                    # 扩展开发模板
│   ├── extension.yml
│   ├── commands/
│   └── templates/
│
├── EXTENSION-API-REFERENCE.md   # API 参考
├── EXTENSION-DEVELOPMENT-GUIDE.md # 开发指南
├── EXTENSION-PUBLISHING-GUIDE.md # 发布指南
└── EXTENSION-USER-GUIDE.md      # 用户指南
```

---

## 三、扩展清单格式

### 3.1 extension.yml 完整结构

```yaml
schema_version: "1.0"            # 必填：清单版本

extension:
  id: "my-extension"             # 必填：唯一标识符（小写、连字符）
  name: "My Extension"           # 必填：显示名称
  version: "1.0.0"               # 必填：语义版本号
  description: "扩展描述"         # 必填：简短描述（<200字符）
  author: "Your Name"            # 必填：作者名称
  repository: "https://github.com/..." # 必填：代码仓库 URL
  license: "MIT"                 # 必填：开源许可证

requires:                        # 依赖要求
  speckit_version: ">=0.1.0"     # Spec Kit 版本要求

provides:                        # 提供的内容
  commands:                      # 命令列表
    - name: "speckit.myext.cmd"
      file: "commands/cmd.md"
      description: "命令描述"
      aliases:                   # 可选：命令别名
        - "speckit.myext.c"

  templates:                     # 模板列表
    - name: "my-template"
      file: "templates/my-template.md"
      description: "模板描述"

  hooks:                         # 钩子列表
    before_implement:
      - command: "speckit.myext.validate"
        optional: true           # 是否可选
        condition: "has_tests"   # 执行条件
        prompt: "验证实现前状态"
```

### 3.2 字段验证规则

```python
class ExtensionManifest:
    SCHEMA_VERSION = "1.0"
    REQUIRED_FIELDS = ["schema_version", "extension", "requires", "provides"]

    def __init__(self, manifest_path: Path):
        # 1. 解析 YAML
        # 2. 验证必需字段
        # 3. 验证版本兼容性
        # 4. 验证命令命名规范
```

---

## 四、命令注册机制

### 4.1 命令命名规范

```
speckit.<extension-id>.<command-name>

示例:
- speckit.selftest.run
- speckit.myext.validate
- speckit.analytics.report
```

### 4.2 注册流程

```python
# extensions.py
class ExtensionManager:
    def _register_commands(self, extension_id: str, commands: list, ...):
        registrar = CommandRegistrar()

        for cmd in commands:
            # 1. 读取命令模板文件
            content = (source_dir / cmd["file"]).read_text()

            # 2. 解析 frontmatter
            frontmatter, body = registrar.parse_frontmatter(content)

            # 3. 转换参数占位符
            body = body.replace("$ARGUMENTS", agent_config["args"])

            # 4. 写入目标目录
            dest_file.write_text(output)
```

### 4.3 目标目录映射

| Agent | 命令目录 | 文件扩展名 |
|-------|---------|-----------|
| Claude | `.claude/commands/` | `.md` |
| Gemini | `.gemini/commands/` | `.toml` |
| Copilot | `.github/agents/` | `.agent.md` |
| Cursor | `.cursor/commands/` | `.md` |

---

## 五、钩子系统

### 5.1 可用钩子点

| 钩子点 | 触发时机 | 用途 |
|-------|---------|------|
| `before_tasks` | 任务生成前 | 预验证、环境检查 |
| `after_tasks` | 任务生成后 | 后处理、通知 |
| `before_implement` | 实现执行前 | 代码检查、安全扫描 |
| `after_implement` | 实现执行后 | 测试、部署 |

### 5.2 钩子配置

```yaml
hooks:
  before_implement:
    - command: "speckit.security.scan"
      optional: true           # 可选钩子（用户可选择执行）
      condition: "has_api"     # 执行条件
      prompt: "运行安全扫描"

    - command: "speckit.lint.check"
      optional: false          # 必须执行的钩子
      prompt: "检查代码规范"
```

### 5.3 钩子执行逻辑

```python
# 命令模板中的钩子处理逻辑
## Pre-Execution Checks

**Check for extension hooks (before implementation)**:
- Check if `.specify/extensions.yml` exists
- Filter to only hooks where `enabled: true`
- For each executable hook:
  - **Optional hook** (`optional: true`):
    ```
    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
```

---

## 六、扩展管理器 API

### 6.1 ExtensionManager 类

```python
class ExtensionManager:
    def __init__(self, project_root: Path):
        self.project_root = project_root
        self.extensions_file = project_root / ".specify" / "extensions.yml"

    def install(self, source: str, project_root: Path = None) -> dict:
        """
        安装扩展

        Args:
            source: 扩展源（本地路径、ZIP URL、Git URL）
            project_root: 项目根目录

        Returns:
            安装信息字典
        """

    def uninstall(self, extension_id: str, project_root: Path = None) -> bool:
        """卸载扩展"""

    def list_installed(self, project_root: Path = None) -> list[dict]:
        """列出已安装扩展"""

    def get_extension(self, extension_id: str) -> Optional[dict]:
        """获取扩展信息"""

    def validate_compatibility(self, manifest: ExtensionManifest) -> bool:
        """验证兼容性"""
```

### 6.2 安装流程

```
1. 解析源地址
   ├── 本地目录: 直接使用
   ├── ZIP URL: 下载解压
   └── Git URL: 克隆仓库

2. 验证清单
   ├── 检查 extension.yml 存在
   ├── 验证必需字段
   └── 验证版本兼容性

3. 安装文件
   ├── 复制到 .specify/extensions/<extension-id>/
   └── 更新 extensions.yml

4. 注册命令
   └── 为每个检测到的 Agent 注册命令

5. 清理临时文件
```

---

## 七、扩展发现机制

### 7.1 目录扫描

```python
def _discover_extensions(self) -> list[str]:
    """扫描已安装扩展"""
    extensions_dir = self.project_root / ".specify" / "extensions"
    if not extensions_dir.exists():
        return []

    return [
        d.name for d in extensions_dir.iterdir()
        if d.is_dir() and (d / "extension.yml").exists()
    ]
```

### 7.2 扩展状态文件

```yaml
# .specify/extensions.yml
extensions:
  - id: "my-extension"
    version: "1.0.0"
    installed_at: "2024-01-15T10:30:00Z"
    source: "https://github.com/user/spec-kit-ext-myextension"
    enabled: true
```

---

## 八、扩展目录（Catalog）

### 8.1 目录结构

```json
// catalog.json
{
  "schema_version": "1.0",
  "extensions": [
    {
      "id": "security-scanner",
      "name": "Security Scanner",
      "version": "1.0.0",
      "description": "安全扫描扩展",
      "repository": "https://github.com/user/spec-kit-ext-security",
      "author": "Security Team",
      "tags": ["security", "scanning"]
    }
  ]
}
```

### 8.2 目录栈

```python
class CatalogStack:
    """管理多个扩展目录"""

    def __init__(self):
        self.catalogs = [
            # 优先级从高到低
            CatalogEntry("project-catalog.json", priority=100),
            CatalogEntry("team-catalog.json", priority=50),
            CatalogEntry("official-catalog.json", priority=10),
        ]
```

---

## 九、扩展开发最佳实践

### 9.1 命名规范

- 扩展 ID: 小写、连字符分隔 (`my-extension`)
- 命令: `speckit.<ext-id>.<cmd>` (`speckit.myext.run`)

### 9.2 版本管理

- 使用语义版本号 (MAJOR.MINOR.PATCH)
- 声明 Spec Kit 版本要求

### 9.3 文档

- 提供 README.md
- 描述命令用途和使用方法
- 列出配置选项

### 9.4 测试

- 使用 `presets/self-test/` 作为参考
- 测试命令在不同 Agent 下的执行

---

## 十、扩展 vs 预设

| 特性 | 扩展 (Extension) | 预设 (Preset) |
|------|-----------------|---------------|
| 主要用途 | 添加新功能 | 自制模板和行为 |
| 命令 | 添加和覆盖 | 仅覆盖 |
| 模板 | 添加和覆盖 | 仅覆盖 |
| 钩子 | 支持 | 不支持 |
| 发现机制 | 扩展目录 | 预设目录 |
| 安装位置 | `.specify/extensions/` | `.specify/presets/` |
| 优先级 | 低 | 高 |