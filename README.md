# Mobile Squad Bridge

通过手机飞书 App 以自然语言对话方式指挥本地 Claude Code Squad 多 Agent 协作工作。

## 核心能力

- **实时对话**：与主 Agent（默认 manager）自然语言对话，流式返回思考过程
- **随时切换**：发送 `/agent worker` 即可切换到其他 Agent 直接对话
- **流式输出**：Agent 回复通过飞书消息编辑逐步展现，秒级延迟
- **任务指派**：发送 `/task` 创建正式 squad 任务，支持生命周期追踪
- **危险命令审批**：检测到 `rm -rf /`、`DROP TABLE` 等危险操作时发送审批卡片到飞书
- **多项目隔离**：不同代码库独立 workspace、独立 Agent 团队

## 前置依赖

- [Go 1.23+](https://go.dev/dl/)（仅编译需要，运行不需要）
- [squad](https://github.com/squad-sh/squad)（Rust CLI，多 Agent 协作）
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)（`npm install -g @anthropic-ai/claude-code`）
- [lark-cli](https://www.npmjs.com/package/@larksuite/cli)（`npm install -g @larksuite/cli`）
- [tmux](https://github.com/tmux/tmux/wiki)（Windows 需 WSL2）

## 快速开始

### 1. 获取对应平台的二进制

从 `bin/` 目录选择你的平台：

| 平台 | 文件 |
|------|------|
| Windows | `bin/bridge-windows-amd64.exe` |
| Linux x86_64 | `bin/bridge-linux-amd64` |
| Linux ARM64 | `bin/bridge-linux-arm64` |
| macOS Intel | `bin/bridge-darwin-amd64` |
| macOS Apple Silicon | `bin/bridge-darwin-arm64` |

### 2. 配置

复制配置示例并填写你的飞书应用凭证：

```bash
cp config.example.json config.json
```

最少需要填写：
- `lark.app_id` / `lark.app_secret`：在[飞书开放平台](https://open.feishu.cn/app)创建企业自建应用获取
- `squad.workspace`：本地 squad workspace 目录路径（需提前存在）

完整配置说明见 [config-guide.md](config-guide.md)。

**环境变量覆盖（推荐用于凭证）：**
```bash
export LARK_APP_ID="cli_xxx"
export LARK_APP_SECRET="xxx"
export SQUAD_WORKSPACE="/home/user/squad"
```

### 3. 启动

```bash
# Linux / macOS
./bin/bridge-linux-amd64 -config config.json

# Windows
.\bin\bridge-windows-amd64.exe -config config.json
```

首次启动会自动：
1. 初始化 squad workspace
2. 写入角色模板到 `.squad/roles/`
3. 创建 tmux sessions 并启动 Claude Code Agent
4. 连接飞书事件总线，开始接收消息

### 4. 使用

在飞书中找到你的机器人，发送消息即可开始对话。

| 命令 | 说明 |
|------|------|
| `/agents` | 查看所有 Agent 状态 |
| `/agent {name}` | 切换到指定 Agent 对话 |
| `/project {name}` | 切换到指定项目 |
| `/task "标题" "内容"` | 发送正式 squad 任务 |
| `/approve {id}` | 批准危险命令 |
| `/reject {id}` | 拒绝危险命令并中断 Agent |

## 核心交互示例

```
[用户] 给项目加一个 JWT 认证模块

[Bridge] 🤔 manager 正在思考...
         ↓（流式更新）
         manager: 我来分析项目结构...
         manager: 发现使用 Gin 框架，我来实现 JWT middleware
         ✅ 回复完成

[manager 内部分配]
[worker 执行中...]

[Bridge] 💬 与 manager 对话中
         manager: JWT middleware 实现已分配给 worker
         ✅ worker: 已完成实现

[用户] /agent worker
[Bridge] 已切换到 worker

[用户] 你用了哪个 JWT 库？
[Bridge] 💬 与 worker 对话中
         worker: 使用了 golang-jwt/jwt v5...
```

## 从源码编译

```bash
# 克隆源码（私有仓库，需权限）
git clone ...
cd source

# 当前平台
go build -o ../bin/bridge ./cmd/bridge

# 交叉编译
GOOS=linux   GOARCH=amd64 go build -o ../bin/bridge-linux-amd64 ./cmd/bridge
GOOS=linux   GOARCH=arm64 go build -o ../bin/bridge-linux-arm64 ./cmd/bridge
GOOS=darwin  GOARCH=amd64 go build -o ../bin/bridge-darwin-amd64 ./cmd/bridge
GOOS=darwin  GOARCH=arm64 go build -o ../bin/bridge-darwin-arm64 ./cmd/bridge
GOOS=windows GOARCH=amd64 go build -o ../bin/bridge-windows-amd64.exe ./cmd/bridge
```

## 技术栈

- **Go 1.23**：Bridge 服务主语言
- **squad**：Rust 编写的本地多 Agent 协作 CLI
- **Claude Code**：Anthropic 终端 AI 编程助手（Node.js）
- **lark-cli**：飞书官方 CLI（Node.js），Bridge 通过调用它与飞书交互
- **tmux**：Agent 运行容器，保持 Claude Code 在后台持续运行

## 文档索引

- [config-guide.md](config-guide.md) — 每个配置字段的详细说明

## 安全

- 所有凭证通过环境变量注入，不硬编码
- 输出自动脱敏（API Key、Token、内网 IP）
- 危险命令触发飞书审批，用户确认后才执行
- 审计日志记录所有消息发送、Agent 切换、审批决议

## License

Private
