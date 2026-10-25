---
title: "OpenCode V2 初学者指南：从安装到第一个任务"
description: "面向第一次接触 OpenCode 的开发者：安装 CLI、连接模型、使用 TUI、配置项目权限，并完成一次安全的代码修改。"
pubDate: 2026-10-25
tags: ["OpenCode", "AI 编程", "CLI", "入门", "工具"]
category: "工具"
---

# OpenCode V2 初学者指南：从安装到第一个任务

OpenCode 是一个开源的 AI 编程代理。它可以直接在你的项目目录里阅读代码、搜索文件、执行命令、修改代码并运行测试。V2 版本重新整理了客户端与服务端架构，也让配置、权限和插件体系更加清晰。

这篇文章不会罗列所有高级配置，而是带你走完一条完整的入门路径：安装、连接模型、认识 TUI、发出第一个任务，以及用配置和权限让 OpenCode 更安全地工作。

> 本文基于 OpenCode V2 官方文档整理。V1 的配置字段和插件 API 与 V2 有差异，如果你正在使用旧版本，建议先阅读官方迁移指南。

## OpenCode 是什么

### 一个终端里的 AI 编程代理

OpenCode 的默认界面是终端里的全屏 TUI。你在项目根目录运行 `opencode`，然后用自然语言描述目标，它会根据当前会话的上下文决定读取哪些文件、执行哪些命令。

它和普通的聊天窗口最大的区别是：OpenCode 能真正接触你的工作区。因此它的能力越强，权限配置就越重要。

### V2 的客户端与服务端架构

V2 把“界面”和“执行”拆成了两层：

- TUI、桌面端和 Web 端都是客户端
- 后台服务负责会话、配置、插件、权限和工具执行

这意味着你可以在终端里打开会话，之后再用其他客户端连接同一个服务，继续同一段工作。

#### 共享后台服务

默认情况下，OpenCode 会为当前用户发现或启动一个共享后台服务。所有本地客户端都连接它，所以多个终端窗口可以共享会话和配置。

查看服务状态：

```bash
opencode service status
```

配置修改通常会自动加载，不需要为了改配置而重启服务。只有在服务卡住或状态异常时，才考虑：

```bash
opencode service restart
```

#### 独立模式

如果你希望 OpenCode 使用一个私有服务，而不是共享服务，可以加上 `--standalone`：

```bash
opencode --standalone
```

也可以连接到一个明确指定的服务地址：

```bash
opencode --server http://localhost:4096
```

##### 什么时候使用独立模式

独立模式适合这些场景：

- 在 CI 中运行一次性任务，希望环境变量只作用于当前服务
- 同时调试多套配置，不想影响日常会话
- 排查共享服务的问题时做对照实验

日常本地开发通常不需要独立模式，保持默认的共享服务即可。

## 安装与首次启动

### 安装 CLI

#### macOS 与 Linux 一键安装

官方提供了一键安装脚本：

```bash
curl -fsSL https://opencode.ai/v2/install | bash
```

安装完成后，可以用下面的命令确认 CLI 是否可用：

```bash
opencode --help
```

#### 使用包管理器

如果你更习惯包管理器，可以选择 Homebrew：

```bash
brew install anomalyco/tap/opencode-v2
```

也可以使用 npm：

```bash
npm install -g @opencode/cli
```

> Windows 目前不支持通过包管理器安装，官方提供独立的二进制包下载。无论使用哪种方式，建议只保留一个安装来源，避免 PATH 中出现多个版本的 `opencode`。

### 连接模型提供商

OpenCode 本身不内置模型，需要连接一个模型提供商。

#### 在 TUI 中连接

先进入项目目录并启动 OpenCode：

```bash
cd ~/code/my-project
opencode
```

在输入框中输入：

```text
/connect
```

选择提供商，再根据提示完成 API Key 或 OAuth 登录。连接成功后，用 `/models` 选择本次会话要使用的模型。

#### 在命令行登录

如果不想打开 TUI，也可以直接用 CLI：

```bash
opencode auth login
```

指定提供商可以跳过第一步选择：

```bash
opencode auth login anthropic --method key
```

查看已经保存的账号：

```bash
opencode auth list
```

凭据保存在本地数据库中，不要把它们复制到项目文件或提交到 Git 仓库。

### 启动第一个会话

#### 基本启动

在项目根目录执行：

```bash
opencode
```

这是最常用的方式。OpenCode 会把当前目录作为工作区，并加载项目中的配置和 `AGENTS.md`。

#### 指定项目目录

也可以不切换目录，直接指定项目路径：

```bash
opencode ~/code/my-project
```

#### 一次性任务

如果只需要模型输出，不想打开交互界面，可以使用 `run`：

```bash
opencode run "解释这个仓库的构建流程"
```

这种方式适合脚本和 CI。想要一个更轻量的交互界面，可以试试：

```bash
opencode mini
```

## 认识 TUI

### 输入与多行文本

在 TUI 里输入内容后按 **Enter** 发送。需要换行时，可以使用 **Shift+Enter**、**Ctrl+Enter**、**Alt+Enter** 或 **Ctrl+J**。

OpenCode 正在工作时，**Enter** 会向当前会话补充指令；如果想把新提示排到后面执行，可以按 **Ctrl+X**，再按 **Enter**。

### 用 @ 引用文件

输入 `@` 会打开文件搜索。选中文件后，它会作为上下文附加到当前提示中：

```text
检查 @src/auth.ts 的错误处理
```

还可以指定行号范围：

```text
解释 @src/auth.ts#20-45 这段逻辑
```

### 用 ! 执行 Shell 命令

在空输入框中输入 `!` 会进入 Shell 模式：

```text
!git status
```

命令会在当前会话的工作目录中执行，输出会保留在会话里。按 **Esc** 可以退出 Shell 模式而不执行命令。

### 常用斜杠命令

输入 `/` 会列出所有可用命令，继续输入可以过滤。

| 操作 | 快捷键 | 命令 |
| --- | --- | --- |
| 新建会话 | Ctrl+X，然后 N | `/new` |
| 切换会话 | Ctrl+X，然后 L | `/sessions` |
| 选择模型 | Ctrl+X，然后 M | `/models` |
| 选择代理 | Ctrl+X，然后 A | `/agents` |
| 撤销 | Ctrl+X，然后 U | `/undo` |
| 重做 | Ctrl+X，然后 R | `/redo` |
| 用外部编辑器写提示 | Ctrl+X，然后 E | `/editor` |

按 **Ctrl+P** 可以打开命令面板，查看当前界面支持的全部操作。

#### 会话管理

每个会话都有自己的上下文。开始一个互不相关的新任务时，优先使用 `/new`，而不是在旧会话里不断追加内容。

按 **Ctrl+O** 可以同时查看最近的会话和项目。

#### 模型与代理

按 **F2** 可以在最近使用过的模型之间切换。按 **Shift+Tab** 可以循环切换代理。

模型决定“由谁来思考和回答”，代理决定“它能做什么”。例如 Plan 代理偏向分析和规划，Build 代理则可以修改代码。

#### 撤销与重做

`/undo` 会撤销最近一条用户消息以及它之后产生的工作，并把那条提示放回输入框，方便你修改后重新发送。

`/redo` 用来恢复刚才撤销的内容。如果你撤销后直接发送了新提示，就相当于接受了这次回退并从此处继续。

### 使用 /btw 提问

当你想问一个和当前任务有关、但不想污染会话上下文的问题时，可以使用：

```text
/btw 为什么第二步的测试失败了？
```

`/btw` 会使用当前会话的上下文和模型来回答，但答案不会加入后续对话。

## 第一个任务：读懂现有项目

### 描述目标

对 AI 编程代理来说，模糊的指令通常得到模糊的结果。对比下面两种说法：

```text
帮我改一下
```

```text
先阅读 README、package.json 和 src/index.ts，告诉我这个项目如何启动、
如何运行测试，以及入口文件负责什么。暂时不要修改任何文件。
```

第二种说法明确了范围、目标和限制，OpenCode 更容易给出可验证的结果。

### 附加文件与范围

如果已经知道相关文件，用 `@` 直接附加可以减少搜索成本：

```text
阅读 @README.md 和 @package.json，总结本地开发需要哪些步骤。
```

如果只想看某个函数的上下文，可以附加行号范围。对于大型仓库，这比一次性让它扫描整个项目更高效。

### 先计划再执行

#### 使用 Plan 代理

面对不熟悉的仓库，先让 Plan 代理做只读分析：

```text
切换到 Plan 代理，阅读项目结构，给出修改方案和风险点，不要编辑文件。
```

可以通过 `/agents` 选择 Plan，也可以按 **Shift+Tab** 循环切换。Plan 默认不会修改普通项目文件，适合在动手前确认思路。

#### 切换到 Build 代理

确认计划后，再切换到 Build 代理执行：

```text
按刚才的计划修改代码，一次只做一步，每步完成后告诉我改了什么。
```

把任务拆小，既方便你审查 diff，也能在出错时快速回退。

### 验证结果

#### 查看变更

不要只看模型的总结，要看实际改动：

```bash
git diff
```

在 TUI 中也可以直接输入 `!git diff`，让命令输出留在会话里。

#### 运行测试

让 OpenCode 运行项目已有的测试命令：

```text
运行项目的测试，并解释失败原因。不要为了让测试通过而删除测试用例。
```

如果测试需要安装依赖或访问网络，OpenCode 可能会发起权限请求。确认命令内容后再批准。

## 让 OpenCode 更懂你的项目

### 配置 opencode.jsonc

在项目根目录创建 `opencode.jsonc`，或者放到 `.opencode/opencode.jsonc`。建议保留 `$schema`，这样编辑器可以提供补全和校验：

```jsonc
{
  "$schema": "https://opencode.ai/config.json"
}
```

#### 设置默认模型

使用 `provider/model` 格式指定默认模型：

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-5-5"
}
```

#### 配置权限

权限规则由 `action`、`resource`、`effect` 三个字段组成。一个常见的保守配置是：Shell 命令默认询问，允许常规 Git 查看命令，禁止推送。

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "permissions": [
    { "action": "shell", "resource": "*", "effect": "ask" },
    { "action": "shell", "resource": "git status *", "effect": "allow" },
    { "action": "shell", "resource": "git diff *", "effect": "allow" },
    { "action": "shell", "resource": "git push *", "effect": "deny" }
  ]
}
```

规则按顺序匹配，最后一条匹配的规则生效。因此要把宽泛的规则放在前面，把例外放在后面。

### 编写 AGENTS.md

`AGENTS.md` 用来告诉 OpenCode 这个项目的长期约定。把它放在项目根目录，并提交到仓库，团队里的每个人都会得到同样的指导。

#### 应该写什么

优先写那些“每次都要重复说明”的信息：

- 如何安装依赖、启动开发服务器、运行测试
- 代码目录的职责划分
- 命名、错误处理、日志等约定
- 哪些文件是自动生成的，不能直接修改
- 提交前必须通过的检查

示例：

```md
# Project instructions

- 包管理器使用 bun，不要混用 npm。
- 修改 TypeScript 后运行 `bunx tsc --noEmit`。
- 数据库查询集中在 `src/database`。
- 不要直接编辑 `dist/` 下的生成文件。
```

> OpenCode V2 只识别 `AGENTS.md`，不会把 `CLAUDE.md` 当作后备文件。嵌套目录中的 `AGENTS.md` 会在代理读到对应区域时加载。

### 自定义斜杠命令

把重复使用的提示保存成命令，可以显著减少输入成本。

#### Markdown 命令

在 `.opencode/commands/review.md` 中写入：

```md
---
description: 审查当前改动
agent: plan
---

审查 $ARGUMENTS 的正确性和缺失的测试，先按严重程度列出问题，再给修改建议。
```

之后在 TUI 中运行：

```text
/review src/auth.ts
```

`$ARGUMENTS` 会被替换成完整的参数文本。命令文件保存后会自动加载，不需要重启。

#### JSON 配置

也可以把命令写进 `opencode.jsonc`：

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "commands": {
    "review": {
      "description": "审查当前改动",
      "template": "审查 $ARGUMENTS 的正确性和缺失的测试。",
      "agent": "plan"
    }
  }
}
```

项目命令会覆盖同名的全局命令；嵌套目录中的命令会以 `/team/review` 这样的形式出现。

### 配置热更新

保存 `opencode.jsonc` 后，OpenCode 会监听变化并重新加载代理、命令、技能、MCP 服务器和插件。如果怀疑配置没有完全生效，可以在 TUI 中运行：

```text
/reload
```

注意：终端外观和快捷键属于 CLI 设置，位于 `~/.config/opencode/cli.json`，不要写进项目配置。

## 安全与排障

### 权限审批

当某条规则的结果是 `ask` 时，客户端会弹出审批。通常有三个选择：

- **Allow once**：只批准这一次
- **Allow always**：批准，并把这次操作产生的模式保存为项目级规则
- **Reject**：拒绝，并拒绝当前会话中其他待处理的权限请求

Shell 命令拥有当前用户的文件系统、进程和网络权限。不要因为嫌麻烦就无脑选择“Always”，尤其是涉及删除、推送、安装全局包或访问敏感目录的命令。

默认策略会允许大多数操作，但对 `.env` 和 `.env.*` 的读取会询问。你可以按项目需要收紧规则，但不要让 `deny` 规则被后面的宽泛规则意外覆盖。

### 后台服务排障

#### 检查服务状态

先确认问题出在客户端、共享服务还是项目配置：

```bash
opencode service status
opencode api get /api/info
```

如果共享服务表现异常，可以用独立模式做对照：

```bash
opencode --standalone
```

如果独立模式正常，问题更可能在共享服务或它的环境中。

#### 查看日志

日志默认位于：

```text
~/.local/share/opencode/log/opencode.log
```

普通日志不够时，可以用一次调试级别的复现来收集信息：

```bash
OPENCODE_LOG_LEVEL=DEBUG opencode
```

分享日志前，记得删除 API Key、授权头、完整提示词和文件内容等敏感信息。

## 下一步

如果你已经完成上面的流程，可以继续尝试这些方向：

- 为你常用的仓库写一份 `AGENTS.md`
- 用 Plan 代理分析不熟悉的代码，再用 Build 代理小步修改
- 把重复的提示沉淀成 `/review`、`/test` 等自定义命令
- 根据团队习惯收紧 Shell 和外部目录权限
- 阅读 OpenCode V2 的配置、权限、代理和插件文档，按需扩展

OpenCode 的价值不只是“让模型写代码”，而是把阅读、执行、修改和验证串成一个可审查的工作流。先用好默认配置，再逐步加入自己的规则，通常比一开始就堆满插件更稳妥。
