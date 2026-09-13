# 踩坑：移动端地图/全屏页三连坑——updateWhenIdle、100vh、backdrop-filter

**日期**：2026-09-08~09 ｜ **背景**：旅游地图上线当晚 TA 手机实测报拖拽卡顿、顶部按钮被裁

## 1. Leaflet 手机默认 updateWhenIdle=true
移动端浏览器拖动地图时，Leaflet 默认等 idle 才加载新瓦片（省流量设计）——体感就是"拖不动/白屏跟手慢"。改 updateWhenIdle: false + keepBuffer: 4，跟手度立变。

## 2. height:100vh 在 iOS 上是"大视口"
地址栏收起后 100vh 大于实际可视高度，顶部控件被裁出屏。双写兜底：height 先写 100vh 再写 100dvh，支持 dvh 的浏览器用后者。

## 3. 手机禁 backdrop-filter
毛玻璃在移动端 GPU 开销大，拖拽时掉帧明显。用 hover:none 媒体查询关掉。

## 泛化
"桌面丝滑手机卡"的性能问题，先查这三个：组件库的移动端默认值、视口单位、特效滤镜。
