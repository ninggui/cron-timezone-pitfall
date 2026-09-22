<div align="center">


![cover](assets/cover.png)

# cron-timezone-pitfall

**Cron 显示 UTC 时间，你的任务比预期晚了 8 小时——一行 TZ 环境变量修好。**

<p>
  <a href="#"><img src="https://img.shields.io/badge/symptom-next_run_at%20%2B00%3A00-red" alt="Symptom" /></a>
</p>

[症状](#症状) · [修复](#修复)

</div>

---

## 症状

`next_run_at` 显示 `+00:00`（UTC），`0 18 * * *` 实际在北京时间凌晨 2 点跑。

## 修复

1. 容器环境变量加 `TZ=Asia/Shanghai`
2. 重新 update cron job 触发时区重算
3. 验证 `next_run_at` 变成 `+08:00`

## 预防

创建 cron 后立即检查 `next_run_at` 后缀。docker-compose 里确保 `- TZ=Asia/Shanghai`。

## License

MIT
