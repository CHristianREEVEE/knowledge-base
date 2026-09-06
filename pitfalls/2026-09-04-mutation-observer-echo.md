# 踩坑：MutationObserver 自触发死循环（badge 永动机事故）

**日期**：2026-09-04 ｜ **现场**：world 观测台全站卡死，curl 却 200

## 事故链
badge 永动机脚本（MutationObserver 监听 document.body + subtree）回调里无条件 `textContent=` 写 badge——写入本身产生 mutation → 触发自己 → 无限循环，主线程被微任务风暴打满。

三个致命细节：
1. `textContent = x` 即使值不变也触碰 DOM，必触发 mutation
2. observer 观察 body+subtree，badge 自己在观察范围内
3. 回调无任何退出条件

## 修复（一行之差，死循环↔正常）

```js
if (badge.textContent !== val) badge.textContent = val;  // 值缓存守卫
```

## 三守则（任选其一）
1. 写入前比较——值不变就不写
2. 防重入标志
3. 缩小观察范围——只 observe 目标节点，别监全 body

同理适用于一切自反馈结构：interval 写 DOM、事件触发自己、rAF 无停止条件。

## 附带教训
- agent-browser 一开就超时 = 页面主线程死循环的最好报警器，别归因成别的（当天被我误读成自己的 resize 改动，白修半小时）
- curl 200 正常 + 页面卡死 = JS 层故障，传输层工具测不出来
