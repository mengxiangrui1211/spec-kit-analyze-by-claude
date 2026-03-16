# Spec Kit 架构图集

本文档包含 Spec Kit 项目的各种架构图，使用 Mermaid 语法绘制。

---

## 一、系统整体架构

```mermaid
flowchart TB
    subgraph UI["用户界面层"]
        CLI["Specify CLI"]
        Agent["AI Agent<br/>(Claude/Gemini/etc.)"]
    end

    subgraph Core["核心层"]
        Init["init 命令"]
        Check["check 命令"]
    end

    subgraph Template["模板层"]
        CmdTmpl["命令模板<br/>(specify.md, plan.md...)"]
        ArtifactTmpl["工件模板<br/>(spec-template.md...)"]
    end

    subgraph Extension["扩展层"]
        ExtMgr["扩展管理器"]
        PresetMgr["预设管理器"]
        Hook["钩子系统"]
    end

    subgraph Infra["基础设施层"]
        GitHubAPI["GitHub API"]
        FileSystem["文件系统"]
        Shell["Shell 执行"]
    end

    CLI --> Init
    CLI --> Check
    Agent --> CmdTmpl

    Init --> GitHubAPI
    Init --> FileSystem

    CmdTmpl --> ExtMgr
    CmdTmpl --> PresetMgr
    ArtifactTmpl --> ExtMgr
    ArtifactTmpl --> PresetMgr

    ExtMgr --> Hook
    PresetMgr --> FileSystem

    Hook --> Shell

    style UI fill:#e3f2fd
    style Core fill:#fff3e0
    style Template fill:#f3e5f5
    style Extension fill:#e8f5e9
    style Infra fill:#fce4ec
```

---

## 二、Spec-Driven Development 工作流

```mermaid
flowchart LR
    subgraph Phase1["阶段 1: 规格定义"]
        C[Constitution<br/>项目原则]
        S[Specify<br/>功能规格]
        Cl[Clarify<br/>澄清细节]
    end

    subgraph Phase2["阶段 2: 技术规划"]
        P[Plan<br/>实现计划]
        R[Research<br/>技术研究]
        D[Data Model<br/>数据模型]
    end

    subgraph Phase3["阶段 3: 任务分解"]
        T[Tasks<br/>任务清单]
    end

    subgraph Phase4["阶段 4: 实现"]
        I[Implement<br/>代码生成]
        V[Validate<br/>验证测试]
    end

    C --> S --> Cl --> P
    P --> R
    P --> D
    P --> T --> I --> V

    style Phase1 fill:#bbdefb
    style Phase2 fill:#c8e6c9
    style Phase3 fill:#fff9c4
    style Phase4 fill:#f8bbd0
```

---

## 三、项目初始化流程

```mermaid
sequenceDiagram
    autonumber
    participant User as 用户
    participant CLI as Specify CLI
    participant GitHub as GitHub API
    participant FS as 文件系统
    participant Git as Git

    User->>CLI: specify init my-project --ai claude
    CLI->>CLI: 解析参数
    CLI->>CLI: 选择 AI Agent

    CLI->>GitHub: GET /repos/github/spec-kit/releases/latest
    GitHub-->>CLI: Release 信息

    CLI->>GitHub: GET 模板 ZIP 下载链接
    GitHub-->>CLI: ZIP 文件流

    CLI->>FS: 创建项目目录
    CLI->>FS: 解压 ZIP 文件
    CLI->>FS: 合并/写入文件

    CLI->>Git: git init
    CLI->>Git: git add .
    CLI->>Git: git commit -m "Initial commit"

    CLI-->>User: 项目初始化完成
```

---

## 四、命令执行流程

```mermaid
sequenceDiagram
    autonumber
    participant User as 用户
    participant Agent as AI Agent
    participant Cmd as 命令模板
    participant Script as Shell 脚本
    participant FS as 文件系统

    User->>Agent: /speckit.specify Build a photo album app
    Agent->>Cmd: 加载 specify.md 模板
    Cmd-->>Agent: 模板内容 + 脚本路径

    Agent->>Script: 执行 create-new-feature.sh --json
    Script->>FS: 创建分支 001-photo-albums
    Script->>FS: 创建 specs/001-photo-albums/
    Script-->>Agent: JSON {BRANCH_NAME, SPEC_FILE}

    Agent->>Cmd: 加载 spec-template.md
    Agent->>Agent: 解析用户需求
    Agent->>Agent: 生成规格内容
    Agent->>FS: 写入 spec.md

    Agent-->>User: 规格创建完成
```

---

## 五、模板解析优先级

```mermaid
flowchart TD
    Request["请求模板: spec-template.md"]

    subgraph Stack["解析栈 (优先级从高到低)"]
        L1["用户模板<br/>.specify/templates/spec-template.md"]
        L2["预设模板<br/>已安装预设提供的模板"]
        L3["扩展模板<br/>已安装扩展提供的模板"]
        L4["核心模板<br/>spec-kit 内置模板"]
    end

    Request --> Check1{L1 存在?}
    Check1 -->|是| ReturnL1["返回用户模板"]
    Check1 -->|否| Check2{L2 存在?}
    Check2 -->|是| ReturnL2["返回预设模板"]
    Check2 -->|否| Check3{L3 存在?}
    Check3 -->|是| ReturnL3["返回扩展模板"]
    Check3 -->|否| ReturnL4["返回核心模板"]

    style L1 fill:#c8e6c9
    style L2 fill:#fff9c4
    style L3 fill:#bbdefb
    style L4 fill:#e0e0e0
```

---

## 六、扩展系统架构

```mermaid
flowchart TB
    subgraph Extension["扩展"]
        Manifest["extension.yml<br/>清单文件"]
        Commands["commands/<br/>命令模板"]
        Templates["templates/<br/>工件模板"]
        Hooks["hooks<br/>钩子配置"]
    end

    subgraph Manager["扩展管理器"]
        Install["install()"]
        Uninstall["uninstall()"]
        List["list_installed()"]
        Validate["validate_compatibility()"]
    end

    subgraph Registrar["命令注册器"]
        Parse["parse_frontmatter()"]
        Render["render_command()"]
        Write["写入 Agent 目录"]
    end

    subgraph AgentDirs["Agent 目录"]
        Claude[".claude/commands/"]
        Gemini[".gemini/commands/"]
        Copilot[".github/agents/"]
    end

    Manifest --> Manager
    Commands --> Manager
    Manager --> Registrar

    Registrar --> Parse --> Render --> Write
    Write --> Claude
    Write --> Gemini
    Write --> Copilot

    style Extension fill:#e8f5e9
    style Manager fill:#fff3e0
    style Registrar fill:#f3e5f5
    style AgentDirs fill:#e3f2fd
```

---

## 七、钩子执行机制

```mermaid
flowchart TD
    Start["命令开始执行"]

    subgraph BeforeHooks["Before 钩子"]
        B1["before_tasks"]
        B2["before_implement"]
    end

    subgraph Command["命令执行"]
        Exec["执行命令逻辑"]
    end

    subgraph AfterHooks["After 钩子"]
        A1["after_tasks"]
        A2["after_implement"]
    end

    End["命令执行完成"]

    Start --> CheckBefore{有 Before 钩子?}
    CheckBefore -->|是| RunBefore["运行 Before 钩子"]
    CheckBefore -->|否| Exec
    RunBefore --> CheckOptional1{钩子可选?}
    CheckOptional1 -->|是| AskUser["提示用户执行"]
    CheckOptional1 -->|否| AutoRun["自动执行"]
    AskUser --> Exec
    AutoRun --> Exec

    Exec --> CheckAfter{有 After 钩子?}
    CheckAfter -->|是| RunAfter["运行 After 钩子"]
    CheckAfter -->|否| End
    RunAfter --> End

    style BeforeHooks fill:#ffcdd2
    style Command fill:#c8e6c9
    style AfterHooks fill:#bbdefb
```

---

## 八、支持的 AI Agent 生态

```mermaid
mindmap
    root((AI Agent 支持))
        IDE 集成
            GitHub Copilot
            Cursor
            Windsurf
            Roo Code
            IBM Bob
            Antigravity
        CLI 工具
            Claude Code
            Gemini CLI
            Qwen Code
            Codex CLI
            Kiro CLI
            Auggie CLI
            Amp
            SHAI
            Tabnine CLI
            CodeBuddy
            Qoder CLI
            Mistral Vibe
            Kimi Code
            opencode
            Kilo Code
        自定义
            Generic Agent
            BYOA
```

---

## 九、文件产出关系

```mermaid
erDiagram
    PROJECT ||--o{ FEATURE : contains
    PROJECT ||--o| CONSTITUTION : has

    FEATURE ||--|{ SPEC : "定义于"
    FEATURE ||--o| PLAN : "规划于"
    FEATURE ||--o| TASKS : "分解于"
    FEATURE ||--o| RESEARCH : "研究于"
    FEATURE ||--o{ DATA_MODEL : "建模于"
    FEATURE ||--o{ CONTRACT : "契约于"
    FEATURE ||--o| QUICKSTART : "验证于"
    FEATURE ||--o{ CHECKLIST : "检查于"

    PROJECT {
        string name
        string ai_agent
    }

    CONSTITUTION {
        text principles
        text standards
        text guidelines
    }

    FEATURE {
        int number
        string name
        string branch
    }

    SPEC {
        text user_stories
        text requirements
        text success_criteria
    }

    PLAN {
        text tech_stack
        text architecture
        text structure
    }

    TASKS {
        text phase_1_setup
        text phase_2_foundation
        text phase_3_user_stories
        text phase_n_polish
    }

    DATA_MODEL {
        text entities
        text relationships
        text validation
    }

    CONTRACT {
        text api_spec
        text interface_def
    }

    RESEARCH {
        text decisions
        text rationale
        text alternatives
    }

    QUICKSTART {
        text scenarios
        text validation_steps
    }

    CHECKLIST {
        text items
        boolean completed
    }
```

---

## 十、项目目录结构

```mermaid
flowchart TB
    subgraph Root["spec-kit-main/"]
        SRC["src/specify_cli/"]
        TPL["templates/"]
        PRE["presets/"]
        EXT["extensions/"]
        DOC["docs/"]
        TST["tests/"]
    end

    subgraph SRC_Detail["src/specify_cli/"]
        INIT["__init__.py<br/>CLI 入口"]
        AGENTS["agents.py<br/>Agent 注册"]
        EXT_MGR["extensions.py<br/>扩展管理"]
        PRE_MGR["presets.py<br/>预设管理"]
    end

    subgraph TPL_Detail["templates/"]
        CMDS["commands/<br/>命令模板"]
        SPECS["spec-template.md"]
        PLANS["plan-template.md"]
        TASKS["tasks-template.md"]
        CONST["constitution-template.md"]
    end

    SRC --> SRC_Detail
    TPL --> TPL_Detail

    style Root fill:#f5f5f5
    style SRC fill:#e3f2fd
    style TPL fill:#f3e5f5
    style PRE fill:#e8f5e9
    style EXT fill:#fff3e0
```

---

## 十一、版本兼容性检查流程

```mermaid
flowchart TD
    Start["安装扩展/预设"]

    Start --> LoadManifest["加载清单文件"]
    LoadManifest --> ParseVersion["解析版本要求"]

    ParseVersion --> CheckSchema{Schema 版本<br/>兼容?}
    CheckSchema -->|否| Error1["错误: 不兼容的清单格式"]
    CheckSchema -->|是| CheckSpeckit{Spec Kit<br/>版本满足?}

    CheckSpeckit -->|否| Error2["错误: 需要 Spec Kit >= X.Y.Z"]
    CheckSpeckit -->|是| CheckDeps{依赖满足?}

    CheckDeps -->|否| Error3["错误: 缺少依赖"]
    CheckDeps -->|是| ValidateContent{内容验证}

    ValidateContent -->|失败| Error4["错误: 无效的内容"]
    ValidateContent -->|通过| Install["执行安装"]

    Install --> Success["安装成功"]

    style Start fill:#e3f2fd
    style Success fill:#c8e6c9
    style Error1 fill:#ffcdd2
    style Error2 fill:#ffcdd2
    style Error3 fill:#ffcdd2
    style Error4 fill:#ffcdd2
```

---

## 图表使用说明

以上架构图可以在支持 Mermaid 的 Markdown 渲染器中直接显示，包括：

- GitHub
- GitLab
- VS Code (with Mermaid extension)
- Notion
- Obsidian
- Typora

如需导出为图片，可使用：
- [Mermaid Live Editor](https://mermaid.live/)
- VS Code Mermaid 插件的导出功能