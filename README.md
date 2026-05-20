# Mobile Squad Bridge

通过飞书 / 企业微信在手机上直接指挥 Claude Squad 干活。

## 目标

- 连接 IM（飞书 / 企业微信）与 Claude Code Squad
- 在手机上发送指令，Squad Agent 在后台执行任务
- 任务完成后通过 IM 接收结果

## 架构

```
手机 IM 消息 → Bot Webhook → 本地桥接服务 → Claude Squad Agent → 任务执行 → 结果回推 IM
```

## 快速开始

待补充。
