/opt/backups # 踩坑：GitHub 备份静默失败 15 天——cron exec 需要 x 位（2026-09-18）

## 现场还原
agent-world 世界数据的三层备份里，GitHub 层（/opt/github_backup.sh 每日 00:10 git 快照推 world-data-backup 仓库）从 9/3 首推后整整 15 天零提交。journalctl 显示 cron 每天都记了 CMD，脚本手动跑永远好使，git log 却始终只有 9/3 一个 commit。当日为修它做全套法医式排查（cron 邮件、journal 触发史、git 内部时间戳、OOM、VM 停顿假设全部排除），最后真凶藏在第一天的 ls -l 里：

```
-rwxr-xr-x root root  269 /opt/backup.sh        ← 有 x 位，cron 天天跑成
-rw------- root root 1192 /opt/github_backup.sh ← 没有 x 位！
```

cron 用 exec 直接拉起命令，无执行权限 → 进程从未启动。"Permission denied" 本该进 cron 邮件，但服务器没装 MTA，journald 只留下一句 "No MTA installed, discarding output"——报错被系统整个吞掉。

## 教训
1. **journalctl 的 "(root) CMD (...)" 只代表 cron 尝试拉起，不代表脚本运行过**——两者之间隔着 exec 权限这一关
2. **无 MTA 的服务器上，cron 任务的 stdout/stderr 是单程票**：报错即蒸发。定时任务必须自证存活（状态文件/心跳），不能依赖输出
3. 手动 `bash script.sh` 好使 ≠ cron 能跑——显式指定解释器会绕过 x 位检查，这正是"手动永远好使"错觉的来源
4. 排障时先复查第一天拿到的原始证据（ls -l 早就拍在脸上），再上重型侦查

## 防再犯
新脚本上 cron 前三件套：chmod 755、脚本自带日志+状态文件、有一个每日任务读状态做看门狗。v2 已落地：显式 PATH / flock 防重叠 / ERR trap 带行号 / push 重试 3 次 / OK-FAIL 状态文件 / 由每日备份任务做看门狗。
