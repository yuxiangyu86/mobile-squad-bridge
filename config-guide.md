# 配置手册

## 概述

Bridge 通过 JSON 配置文件启动，默认读取 `config.json`（可通过 `-config` 参数指定）。

```bash
./bin/bridge -config config.json
```

所有凭证类配置都支持通过**环境变量**覆盖，优先级高于配置文件。

---

## 完整配置示例

```json
{
  "bridge": {
    "http_addr": ":8080",
    "poll_interval_ms": 5000,
    "health_check_interval_sec": 30,
    "task_timeout_min": 30,
    "enable_rc": false,
    "rc_server_url": "",
    "stream_mode": "text"
  },
  "lark": {
    "app_id": "cli_xxxxxxxxxxxxxxxx",
    "app_secret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "encrypt_key": "",
    "identity_mode": "bot",
    "event_keys": ["im.message.receive_v1"],
    "auto_reply": true,
    "card_update_mode": "edit"
  },
  "squad": {
    "workspace": "C:/Users/yourname/squad-workspace",
    "bridge_agent_id": "bridge",
    "bridge_role": "bridge",
    "protocol_version": 2,
    "agents": ["manager", "worker", "inspector"],
    "default_agent": "manager",
    "projects": {
      "project-a": {
        "workspace": "C:/Users/yourname/projects/project-a/.squad",
        "workdir": "C:/Users/yourname/projects/project-a",
        "agents": ["manager", "worker"]
      },
      "project-b": {
        "workdir": "C:/Users/yourname/projects/project-b"
      }
    }
  },
  "tmux": {
    "session_prefix": "squad",
    "auto_recover": true,
    "capture_interval_sec": 60
  },
  "security": {
    "output_filter": true,
    "audit_log_path": "./logs/audit.log",
    "max_task_timeout_min": 30,
    "dangerous_commands": ["rm -rf /", "sudo", "DROP TABLE"]
  },
  "log": {
    "level": "info",
    "format": "json",
    "output_path": "./logs/bridge.log"
  }
}
```

---

## 字段详解

### `bridge` — Bridge 服务行为

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `http_addr` | string | `:8080` | HTTP 服务监听地址（health + webhook） |
| `poll_interval_ms` | int | 5000 | 轮询 squad 消息的间隔（毫秒） |
| `health_check_interval_sec` | int | 30 | tmux session 健康检查间隔（秒） |
| `task_timeout_min` | int | 30 | 任务超时时间（分钟） |
| `enable_rc` | bool | false | 是否启用 Claude Code Remote Control |
| `rc_server_url` | string | `""` | RC Server 地址（自托管时填写） |
| `stream_mode` | string | `text` | 流式推送模式：`text` 或 `card` |

**`stream_mode` 说明：**
- `text`：使用文本消息编辑，简单直接
- `card`：使用飞书卡片消息编辑，支持更丰富的 UI（按钮、颜色等）

### `lark` — 飞书平台接入

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `app_id` | string | `""` | 飞书应用 ID（必填） |
| `app_secret` | string | `""` | 飞书应用 Secret（必填） |
| `encrypt_key` | string | `""` | 事件加密密钥（可选，开启事件加密时填写） |
| `identity_mode` | string | `bot` | 身份模式：`bot`（机器人）或 `user`（用户） |
| `event_keys` | []string | `["im.message.receive_v1"]` | 订阅的事件类型 |
| `auto_reply` | bool | true | 是否自动回复 |
| `card_update_mode` | string | `edit` | 卡片更新模式：`edit`（编辑原消息）或 `new`（发新消息） |

**获取方式：**
- `app_id` / `app_secret`：在[飞书开放平台](https://open.feishu.cn/app)创建企业自建应用后获取
- `encrypt_key`：在应用「事件订阅」页面开启加密后获取

**环境变量覆盖：**
- `LARK_APP_ID` → 覆盖 `app_id`
- `LARK_APP_SECRET` → 覆盖 `app_secret`
- `LARK_ENCRYPT_KEY` → 覆盖 `encrypt_key`
- `LARK_IDENTITY_MODE` → 覆盖 `identity_mode`

### `squad` — Squad 协作网络

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `workspace` | string | `""` | 默认 squad workspace 路径（必填，目录必须存在） |
| `bridge_agent_id` | string | `bridge` | Bridge 在 squad 中的 Agent ID |
| `bridge_role` | string | `bridge` | Bridge 的角色名 |
| `protocol_version` | int | 2 | squad 协议版本 |
| `agents` | []string | `["manager","worker","inspector"]` | 全局默认 Agent 列表 |
| `default_agent` | string | `manager` | 默认活跃 Agent |
| `projects` | map | `{}` | 多项目隔离配置（可选） |

#### `projects` 多项目配置

用于隔离不同代码库，每个项目有独立的 workspace 和 Agent。

```json
"projects": {
  "project-a": {
    "workspace": "C:/Users/yourname/projects/project-a/.squad",
    "workdir": "C:/Users/yourname/projects/project-a",
    "agents": ["manager", "worker"]
  }
}
```

| 字段 | 说明 |
|------|------|
| `workspace` | 项目独立的 squad workspace（为空则自动创建在默认 workspace 下的 `projects/{name}/`） |
| `workdir` | Claude Code 启动时的工作目录（为空则等于 workspace） |
| `agents` | 项目专属 Agent 列表（为空则使用全局 `agents`） |

**环境变量覆盖：**
- `SQUAD_WORKSPACE` → 覆盖 `workspace`

### `tmux` — 终端复用器

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `session_prefix` | string | `squad` | tmux session 名称前缀 |
| `auto_recover` | bool | true | 是否自动恢复崩溃/退出的 session |
| `capture_interval_sec` | int | 60 | Pane Capture 兜底策略的捕获间隔（秒） |

**session 命名规则：**
- 默认项目：`{prefix}-{agent}`，如 `squad-manager`
- 多项目：`{prefix}-{project}-{agent}`，如 `squad-project-a-manager`

### `security` — 安全与审计

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `output_filter` | bool | true | 是否启用输出过滤（脱敏 API Key、Token、内网 IP） |
| `audit_log_path` | string | `./logs/audit.log` | 审计日志文件路径 |
| `max_task_timeout_min` | int | 30 | 最大任务超时（分钟） |
| `dangerous_commands` | []string | 见示例 | 危险命令关键字列表，触发审批流程 |

**输出过滤规则：**
- 自动匹配并替换 `sk-xxx`（OpenAI API Key）、`Bearer xxx`（Token）、内网 IP（`10.x.x.x`、`192.168.x.x`、`172.16-31.x.x`）

**危险命令检测：**
- 当 Agent 输出包含 `dangerous_commands` 中的关键字时，Bridge 会向飞书发送审批卡片
- 用户点击「批准」后继续执行，点击「拒绝」则发送 `Ctrl+C` 中断对应 Agent 的 tmux session

### `log` — 日志

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `level` | string | `info` | 日志级别：`debug`、`info`、`warn`、`error` |
| `format` | string | `json` | 格式：`json` 或 `text` |
| `output_path` | string | `./logs/bridge.log` | 输出路径（为空则输出到 stdout） |

**环境变量覆盖：**
- `LOG_LEVEL` → 覆盖 `level`
- `LOG_FORMAT` → 覆盖 `format`

---

## 环境变量总览

| 环境变量 | 覆盖配置 | 示例 |
|----------|----------|------|
| `LARK_APP_ID` | `lark.app_id` | `cli_xxx` |
| `LARK_APP_SECRET` | `lark.app_secret` | `xxx` |
| `LARK_ENCRYPT_KEY` | `lark.encrypt_key` | `xxx` |
| `LARK_IDENTITY_MODE` | `lark.identity_mode` | `bot` |
| `SQUAD_WORKSPACE` | `squad.workspace` | `/home/user/squad` |
| `BRIDGE_HTTP_ADDR` | `bridge.http_addr` | `:8080` |
| `LOG_LEVEL` | `log.level` | `debug` |
| `LOG_FORMAT` | `log.format` | `text` |
| `BRIDGE_ENABLE_RC` | `bridge.enable_rc` | `true` |
| `BRIDGE_RC_SERVER_URL` | `bridge.rc_server_url` | `ws://localhost:8081` |
| `BRIDGE_POLL_INTERVAL_MS` | `bridge.poll_interval_ms` | `5000` |

---

## 配置校验规则

启动时 Bridge 会自动校验配置，以下情况会报错退出：

1. `lark.app_id` 为空
2. `lark.app_secret` 为空
3. `squad.workspace` 为空
4. `squad.workspace` 指向的目录不存在

---

## 最小可用配置

如果只连接一个项目、使用默认参数，最小配置如下：

```json
{
  "lark": {
    "app_id": "cli_xxxxxxxxxxxxxxxx",
    "app_secret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  },
  "squad": {
    "workspace": "C:/Users/yourname/squad-workspace"
  }
}
```

其余字段全部采用默认值即可启动。
