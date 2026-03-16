# Spec Kit CLI 实现详解

## 一、CLI 入口分析

### 1.1 主入口定义

文件位置: `src/specify_cli/__init__.py`

```python
#!/usr/bin/env python3
# /// script
# requires-python = ">=3.11"
# dependencies = [
#     "typer",
#     "rich",
#     "platformdirs",
#     "readchar",
#     "httpx",
# ]
# ///

app = typer.Typer(
    name="specify",
    help="Setup tool for Specify spec-driven development projects",
    add_completion=False,
    invoke_without_command=True,
    cls=BannerGroup,
)
```

### 1.2 依赖说明

| 依赖 | 用途 |
|------|------|
| `typer` | CLI 框架，命令解析 |
| `rich` | 终端美化输出 |
| `platformdirs` | 跨平台路径处理 |
| `readchar` | 键盘输入处理 |
| `httpx` | HTTP 客户端，GitHub API 调用 |

---

## 二、核心命令实现

### 2.1 `specify init` 命令

**功能**: 初始化 Spec-Driven Development 项目

**参数列表**:

```python
def init(
    project_name: str = typer.Argument(None, help="项目名称"),
    ai: str = typer.Option(None, "--ai", help="AI 助手选择"),
    ai_commands_dir: str = typer.Option(None, "--ai-commands-dir"),
    script: str = typer.Option("sh", "--script"),
    ignore_agent_tools: bool = typer.Option(False, "--ignore-agent-tools"),
    no_git: bool = typer.Option(False, "--no-git"),
    here: bool = typer.Option(False, "--here"),
    force: bool = typer.Option(False, "--force"),
    skip_tls: bool = typer.Option(False, "--skip-tls"),
    debug: bool = typer.Option(False, "--debug"),
    github_token: str = typer.Option(None, "--github-token"),
    ai_skills: bool = typer.Option(False, "--ai-skills"),
):
```

**执行流程**:

```
1. 参数验证和处理
   ├── 处理项目名称（当前目录 or 新目录）
   ├── 处理 AI 助手选择（交互式 or 命令行指定）
   └── 处理别名（如 kiro -> kiro-cli）

2. 模板下载
   ├── 调用 GitHub API 获取最新 Release
   ├── 查找匹配的模板 ZIP 文件
   └── 下载到临时目录

3. 解压和合并
   ├── 解压 ZIP 文件
   ├── 处理嵌套目录结构
   ├── 合并到目标目录
   └── 处理 .vscode/settings.json 特殊合并

4. Git 初始化
   ├── git init
   ├── git add .
   └── git commit -m "Initial commit"

5. 清理临时文件
```

### 2.2 `specify check` 命令

**功能**: 检查系统工具是否安装

**检查的工具**:

```python
TOOLS_TO_CHECK = [
    "git",
    "claude",      # Claude Code CLI
    "gemini",      # Gemini CLI
    "code",        # VS Code
    "cursor-agent",
    "windsurf",
    "qwen",
    "opencode",
    "codex",
    "kiro-cli",
    "shai",
    "qodercli",
    "vibe",
    "kimi",
]
```

---

## 三、Agent 配置系统

### 3.1 Agent 配置结构

```python
AGENT_CONFIG = {
    "claude": {
        "name": "Claude Code",           # 显示名称
        "folder": ".claude/",            # 配置目录
        "commands_subdir": "commands",   # 命令子目录
        "install_url": "https://...",    # 安装文档 URL
        "requires_cli": True,            # 是否需要 CLI 工具
    },
    "copilot": {
        "name": "GitHub Copilot",
        "folder": ".github/",
        "commands_subdir": "agents",     # 特殊: 使用 agents/ 而非 commands/
        "install_url": None,             # IDE 内置，无独立安装
        "requires_cli": False,
    },
    "codex": {
        "name": "Codex CLI",
        "folder": ".codex/",
        "commands_subdir": "prompts",    # 特殊: 使用 prompts/
        "install_url": "https://github.com/openai/codex",
        "requires_cli": True,
    },
    # ... 更多 Agent
}
```

### 3.2 Agent 别名处理

```python
AI_ASSISTANT_ALIASES = {
    "kiro": "kiro-cli",  # kiro 是 kiro-cli 的别名
}

def resolve_alias(ai: str) -> str:
    return AI_ASSISTANT_ALIASES.get(ai, ai)
```

---

## 四、进度跟踪系统

### 4.1 StepTracker 类

**设计目的**: 提供层级化的进度显示，类似树形结构

```python
class StepTracker:
    def __init__(self, title: str):
        self.title = title
        self.steps = []  # [{key, label, status, detail}]

    def add(self, key: str, label: str):
        """添加新步骤（pending 状态）"""

    def start(self, key: str, detail: str = ""):
        """标记步骤为运行中"""

    def complete(self, key: str, detail: str = ""):
        """标记步骤为完成"""

    def error(self, key: str, detail: str = ""):
        """标记步骤为错误"""

    def skip(self, key: str, detail: str = ""):
        """标记步骤为跳过"""

    def render(self) -> Tree:
        """渲染为 Rich Tree"""
```

### 4.2 状态显示

```
● 已完成 (done)        [green]
○ 待处理 (pending)     [bright_black]
○ 运行中 (running)     [cyan]
● 错误 (error)         [red]
○ 跳过 (skipped)       [yellow]
```

---

## 五、交互式选择器

### 5.1 箭头键选择

```python
def select_with_arrows(
    options: dict,
    prompt_text: str = "Select an option",
    default_key: str = None
) -> str:
    """
    使用箭头键进行交互式选择

    ┌─────────────────────────────┐
    │     Select AI Assistant     │
    ├─────────────────────────────┤
    │ ▶ claude   (Claude Code)    │
    │   gemini   (Gemini CLI)     │
    │   cursor   (Cursor)         │
    ├─────────────────────────────┤
    │ Use ↑/↓ to navigate, Enter  │
    │ to select, Esc to cancel    │
    └─────────────────────────────┘
    """
```

### 5.2 键盘事件处理

```python
def get_key():
    """获取单个按键"""
    key = readchar.readkey()

    if key == readchar.key.UP or key == readchar.key.CTRL_P:
        return 'up'
    if key == readchar.key.DOWN or key == readchar.key.CTRL_N:
        return 'down'
    if key == readchar.key.ENTER:
        return 'enter'
    if key == readchar.key.ESC:
        return 'escape'
    if key == readchar.key.CTRL_C:
        raise KeyboardInterrupt

    return key
```

---

## 六、GitHub API 集成

### 6.1 认证处理

```python
def _github_token(cli_token: str | None = None) -> str | None:
    """获取 GitHub Token（优先级：CLI 参数 > GH_TOKEN > GITHUB_TOKEN）"""
    return (cli_token or os.getenv("GH_TOKEN") or os.getenv("GITHUB_TOKEN") or "").strip()

def _github_auth_headers(cli_token: str | None = None) -> dict:
    """生成认证请求头"""
    token = _github_token(cli_token)
    return {"Authorization": f"Bearer {token}"} if token else {}
```

### 6.2 速率限制处理

```python
def _parse_rate_limit_headers(headers: httpx.Headers) -> dict:
    """解析 GitHub API 速率限制头"""
    info = {}

    if "X-RateLimit-Limit" in headers:
        info["limit"] = headers.get("X-RateLimit-Limit")
    if "X-RateLimit-Remaining" in headers:
        info["remaining"] = headers.get("X-RateLimit-Remaining")
    if "X-RateLimit-Reset" in headers:
        reset_epoch = int(headers.get("X-RateLimit-Reset", "0"))
        reset_time = datetime.fromtimestamp(reset_epoch, tz=timezone.utc)
        info["reset_time"] = reset_time

    return info
```

### 6.3 模板下载

```python
def download_template_from_github(
    ai_assistant: str,
    download_dir: Path,
    script_type: str = "sh",
    verbose: bool = True,
    show_progress: bool = True,
    client: httpx.Client = None,
    debug: bool = False,
    github_token: str = None
) -> Tuple[Path, dict]:
    """
    从 GitHub Release 下载模板

    返回: (zip_path, metadata)
    metadata 包含: filename, size, release, asset_url
    """
    # 1. 获取最新 Release 信息
    api_url = f"https://api.github.com/repos/github/spec-kit/releases/latest"
    response = client.get(api_url, headers=_github_auth_headers(github_token))

    # 2. 查找匹配的模板文件
    pattern = f"spec-kit-template-{ai_assistant}-{script_type}"
    matching_assets = [a for a in assets if pattern in a["name"]]

    # 3. 下载文件
    with client.stream("GET", download_url) as response:
        for chunk in response.iter_bytes(chunk_size=8192):
            f.write(chunk)
```

---

## 七、文件处理工具

### 7.1 JSON 合并

```python
def merge_json_files(existing_path: Path, new_content: dict, verbose: bool = False) -> dict:
    """
    深度合并 JSON 文件

    规则:
    - 新键添加
    - 现有键保留（除非被覆盖）
    - 嵌套字典递归合并
    - 列表和其他值直接替换
    """
    def deep_merge(base: dict, update: dict) -> dict:
        result = base.copy()
        for key, value in update.items():
            if key in result and isinstance(result[key], dict) and isinstance(value, dict):
                result[key] = deep_merge(result[key], value)
            else:
                result[key] = value
        return result

    return deep_merge(existing_content, new_content)
```

### 7.2 VSCode Settings 处理

```python
def handle_vscode_settings(sub_item, dest_file, rel_path, verbose=False, tracker=None):
    """
    特殊处理 .vscode/settings.json 文件

    不直接覆盖，而是合并配置
    """
    with open(sub_item, 'r') as f:
        new_settings = json.load(f)

    if dest_file.exists():
        merged = merge_json_files(dest_file, new_settings)
        with open(dest_file, 'w') as f:
            json.dump(merged, f, indent=4)
    else:
        shutil.copy2(sub_item, dest_file)
```

---

## 八、错误处理和调试

### 8.1 错误消息格式化

```python
def _format_rate_limit_error(status_code: int, headers: httpx.Headers, url: str) -> str:
    """格式化用户友好的错误消息"""
    lines = [f"GitHub API returned status {status_code} for {url}"]

    lines.append("[bold]Rate Limit Information:[/bold]")
    lines.append(f"  • Rate Limit: {rate_info['limit']} requests/hour")
    lines.append(f"  • Remaining: {rate_info['remaining']}")
    lines.append(f"  • Resets at: {reset_time}")

    lines.append("[bold]Troubleshooting Tips:[/bold]")
    lines.append("  • Consider using a GitHub token via --github-token")
    lines.append("  • Authenticated requests: 5,000/hour vs 60/hour")

    return "\n".join(lines)
```

### 8.2 调试模式

```python
if debug:
    error_msg += f"\n\n[dim]Response body (truncated 500):[/dim]\n{response.text[:500]}"
```

---

## 九、关键设计决策

### 9.1 为什么使用 Typer？

- 声明式命令定义
- 自动生成帮助信息
- 类型提示友好
- 与 Rich 无缝集成

### 9.2 为什么支持多脚本类型？

```python
SCRIPT_TYPE_CHOICES = {
    "sh": "POSIX Shell (bash/zsh)",
    "ps": "PowerShell"
}
```

跨平台支持：
- Linux/macOS 默认使用 bash
- Windows 默认使用 PowerShell

### 9.3 为什么使用 Truststore？

```python
import truststore
ssl_context = truststore.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
```

解决企业环境中的 SSL 证书问题，使用系统信任的证书存储。