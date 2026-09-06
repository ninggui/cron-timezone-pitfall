# cron-timezone-pitfall

![GitHub stars](https://img.shields.io/github/stars/ninggui/cron-timezone-pitfall)
![License](https://img.shields.io/github/license/ninggui/cron-timezone-pitfall)
[![SkillHub](https://img.shields.io/badge/SkillHub-在线安装-blue)](https://skillhub.cn/skills/cron-timezone-pitfall)

Cron 时区陷阱：调度器用 UTC 导致北京时间错位 8 小时的排查与修复。

## 问题现象

- 定时任务按计划时间触发，但实际执行总是差 8 小时
- 部分任务正常、部分错位（混用 UTC/本地时区配置）
- 日志时间戳与业务时间戳不一致

## 根因

调度器（cron 守护进程/平台调度）默认使用 **UTC**，而业务期望北京时间（UTC+8）。cron 表达式本身无时区概念，依赖宿主环境 `TZ` 或调度器时区配置。

## 修复步骤

1. 确认调度器时区：`date` 与 `date -u` 对比
2. 统一设置时区：`TZ=Asia/Shanghai`（或调度器时区参数）
3. 验证：`cron 表达式换算为 UTC 后对照预期执行时间`
4. 建议所有 cron 表达式按 UTC 书写并在文档标注时区，避免环境迁移后错位

## 排查命令

```bash
date; date -u                  # 对比本地与 UTC
crontab -l | head              # 查看现有表达式
grep TZ /etc/default/cron 2>/dev/null
```

## 安装

- **Hermes**: `skills/` 目录
- **Claude**: `~/.claude/skills/`
- 或 SkillHub 一键安装：https://skillhub.cn/skills/cron-timezone-pitfall

## 许可

MIT
