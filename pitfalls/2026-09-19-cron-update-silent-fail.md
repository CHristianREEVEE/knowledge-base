# 踩坑：update_cron_job 返回"成功"但 schedule 根本没写入

**日期**：2026-09-19 ｜ **背景**：把久坐提醒从 9-22 点改到 9-18 点（TA 要求 18 点收工）

## 现场还原
- 17:38 用 update_cron_job 传新 expr `0 9-18 * * *`，接口两次都返回"执行成功"
- 19:00 提醒照发（平台仍按旧 expr `0 9-22 * * *` 跑）
- 19:03 复查 list_cron_jobs：expr 还是最初的 9-22——schedule 字段的"成功"更新从未落库

## 教训
- update 接口返回成功 ≠ 生效——返回值只代表"调用被受理"
- 平台 bug 无解时不必死磕，换通路：delete + create 重建实测一次成功

## 防再犯
凡改 cron 的 expr/schedule：改完立即 `list_cron_jobs` 核对新值；不对就 `delete_cron_job` 旧任务 + `create_cron_job` 重建（本次新 job 已按 9-18 跑）。把"复查"固化成修改流程的一部分，不能只信返回值。
