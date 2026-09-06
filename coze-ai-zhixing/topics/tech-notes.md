# 技术笔记

> 时间跨度：2026 年 7 月初 – 8 月底，2026-08-31 统一归档。学习笔记，原话照录。

## CLI / TUI / GUI 三种界面

- **CLI 命令行**：纯输入输出，没有交互面板，只能敲指令。例：原始 ping、ls
- **TUI 文本界面**：终端内带菜单、光标选择、分割边框。例：htop、nano、neofetch、ranger
- **GUI 图形界面**：窗口、图标、图片、鼠标拖拽。例：Chrome、微信、Windows 桌面
**#概念卡 #CLI #TUI #GUI**

## agent 微信主动发消息方法

当需要主动给用户的微信发送消息时，按以下步骤操作：

1. 先调用 `sessions_list` 工具，找到 channel 为 wechat 的 session（通常有 type=main 和 type=cron 两个，优先用 type=main 的那个）
2. 记下该 session 的 session_id
3. 调用 `sessions_send` 工具发送消息：
   - session_id：填上面找到的微信 main session 的 id
   - message：在消息开头加上「请将以下内容原样回复给用户，不要改写：」这句话，再拼上要发的内容
   - 示例 message：`请将以下内容原样回复给用户，不要改写：\n\n你要发的内容在这里`
   - timeout_seconds 设为 10 即可

**注意事项：**
- `sessions_send` 是在给另一个 agent session 发消息，不是直接给用户发消息。开头那段「请将以下内容原样回复给用户」是必须的，否则接收方 agent 会二次加工你的内容
- 微信 session 状态要是 running 或 idle 才能发
- 发完 sessions_send 返回 ok 就代表消息已经投递到微信 session 了，微信端会立即推送给用户

**#agent #微信 #sessions_send #笔记**

## 相关深潜（豆包讲解，见对话实录批次二）

- AGI 核心技术挑战 → 对话十八
- Agent 是什么 → 对话十九
- agent web search 起源时间线（WebGPT → Perplexity → Operator/computer use）→ 对话二十
- 搜索引擎 API 原理 → 对话二十一
- 豆包 vs DeepSeek 快速模式（CDN 提速 vs 架构提速）→ 对话二十二
- MoE 混合专家模型 → 对话二十三
**#索引 #豆包课堂**

## 2026-09-02 | 网页大改版军规：注入代码别动 script 标签本身

- 事故：用 `replace('<script>', 注入片段)` 会把原脚本的开标签吞掉——整段 JS 变正文、页面全灭，且**无任何 JS 报错**（代码根本没执行）；回退组装时同一手法复发，"回退版也坏"是假象，排查被带偏半小时
- 正确姿势：JS 内容插到 `<script>` 标签**内部**，不动标签本身
- 大改版流程：模板 → 一次组装（禁止多轮补丁叠加）→ `node --check` → 本地真跑 → 推送 → 线上字节核对 → 线上复测；每步失败就地停下，不带病前进
- 出问题先 git log 找最后确认正常的 commit 对比 diff，别在污染版本上继续排查

**#前端 #注入 #大改版 #事故复盘**

## 2026-09-02 | 域名经济学：小众后缀的首年贴钱陷阱

- 小众后缀（.xyz/.icu 等）= "首年贴钱冲量 + 续费高价收割"模式——aibingworld.top 首年 15 续费 35
- 规则：买域名看续费价，不看首年价；华而不实的后缀一律当营销价格看待

**#域名 #经济学 #建站**

## 2026-09-03 | MCP vs HTTP：一个入口统一

- 翻车现场：MCP 工具参数是 itemName/count，SKILL.md 写的是 itemId/qty——两套接口各写各的，MCP 玩家照文档读必踩坑
- 拍板：取消 MCP 通道，全员走 HTTP API，一个入口统一——参数不一致问题随之消失
- 通用启示：双通道并行 = 双倍文档维护 + 口径漂移风险；HTTP = 所有 agent 的普通话，协议统一的收益大于兼容的收益

**#MCP #HTTP #接口设计 #agent协议**

## 2026-09-03 | Skill = 渐进式披露上下文

- 南浊的洞察：**skill=渐进式披露上下文**——给豆包只需发 skill 名，世界观/接口文档/策略全在包里，用时展开
- 封装价值：一次沉淀、任意 agent 复用；对挂机场景 = 把"世界规则书"压缩成一个可分发文件

**#skill #上下文管理 #agent工程**

## 2026-09-04 | 验证层级必须与故障层级对齐（badge 永动机事故）

- **铁律**：curl/Node 只覆盖传输层；JS/运行时故障必须真浏览器跑（15s 内 readyState=complete）；最终验收 = 用户真机肉眼确认
- 事故链：MutationObserver 监听全 body + 回调里无条件写 badge → 写入触发 mutation → 再写入，微任务风暴打满主线程；curl 却 200 正常（传输层没坏 ≠ 页面活着）
- **MutationObserver 三守则**（适用于一切自反馈结构）：写入前先比较 / 防重入标志 / 缩小观察范围——三选一即可防死循环
- 修复版一行之差：值缓存守卫（`textContent !== val` 先比较再写入）

**#前端 #MutationObserver #验收 #自反馈结构**
