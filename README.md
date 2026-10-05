<img src="./assets/cover.png" alt="Cron 时区陷阱" width="100%">

<div align="center">

# Cron 时区陷阱排查

**Cron 显示 UTC 时间，任务比预期晚 8 小时——一条命令定位，一行环境变量修好。**

![Status](https://img.shields.io/badge/status-production-green)
![Pitfalls](https://img.shields.io/badge/pitfalls-4-red)
![Verified](https://img.shields.io/badge/verified-2026.08-blue)
![License](https://img.shields.io/badge/license-MIT-blue)

[它解决什么问题](#它解决什么问题) - [为什么比手动强](#为什么比手动强) - [工作流](#工作流) - [实测参数](#实测参数) - [快速开始](#快速开始)

</div>

---

## 它解决什么问题

调度任务每天凌晨悄悄"迟到"8 小时：`next_run_at` 显示 `+00:00`（UTC），`0 18 * * *` 在北京时间凌晨 2 点才跑。排查时还容易踩相邻坑——`0 */30 * * *` 并不是每 30 分钟一次、`cronjob action=update` 传 model 参数无效、整点任务并发撞网关 504。

## 为什么比手动强

| 手动排查 | 本仓库 |
|---|---|
| 翻遍配置找时区来源 | 一眼锁定 `next_run_at` 后缀 `+00:00` / `+08:00` |
| 以为 `0 */30 * * *` 是每 30 分钟 | 明确：小时字段 `*/30` 只在 0 点匹配，每 30 分钟要写分钟字段 |
| update 传 model 参数后以为生效 | 实测确认无效，必须用 `hermes cron edit --model --provider` |
| 整点并发 504 反复查配置 | 诊断 4 步 + 错峰模板，一次理清 |
| 改完凭感觉 | 3 步验证：TZ → update 重算 → 复查 `+08:00` |

## 工作流

```
症状识别（next_run_at 显示 +00:00）
   ↓
验证（cronjob action=list 看时区后缀）
   ↓
修复（TZ=Asia/Shanghai → update 重算 → 复查 +08:00）
   ↓
预防（创建后立即查后缀 + docker-compose 确保 TZ）
```

## 实测参数

- **时区错位**：`0 18 * * *` 实际在北京时间 02:00 执行（晚 8 小时）
- **2026-08-12 实测**：`cronjob action=update` 传 `model/provider` 无效，返回 job 中 model 仍为 null；模型固定必须用 CLI `hermes cron edit --model --provider`
- **2026-08-11 实测**：整点并发触发网关 `HTTP 504`；错峰方案——大 job 移到半点（如 `30 */2 * * *`），大任务间至少错开 30 分钟
- **504 诊断顺序**：① curl 测网关健康 → ② CLI 直连测主会话 → ③ 查 errors.log 的 provider/base_url → ④ 确认 fallback 链生效

## 快速开始

```bash
# 1. 查 Job 时区
cronjob action=list   # 看 next_run_at 后缀：+00:00=UTC(错) / +08:00=北京(对)

# 2. 修复：容器环境变量加 TZ=Asia/Shanghai，
#    重新 update 相同 schedule 触发时区重算

# 3. 验证
cronjob action=list   # next_run_at 变为 +08:00
```

## License

MIT
