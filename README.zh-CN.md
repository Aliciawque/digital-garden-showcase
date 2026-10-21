# 数字花园 · Digital Garden by Alicia

Alicia 创作的免费 3D 互动花卉作品。打开浏览器，探索四季与昼夜，聆听自然声，调整光线和植物颜色，再拍一张花卉照片或一段短片。

**[走进数字花园](https://digitalgarden.si/zh/?utm_source=github&utm_medium=referral&utm_campaign=showcase)** · [English](README.md) · [体验介绍](https://digitalgarden.si/zh/virtual-garden/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

![数字花园中金色的秋日麦穗与花卉](https://digitalgarden.si/marketing/autumn.jpg)

## 可以做什么

- 探索春夏秋冬，切换白天与夜晚。
- 靠近一朵花，改变风力、颜色与光线。
- 选择画幅与焦点，拍摄照片或录制最长 10 秒的短片。
- 保存文件，或在支持的浏览器里通过系统面板分享文件。
- 无需注册或安装。偏好保存在当前设备，拍摄文件由你决定何时分享。

这里的“数字花园”是一件可探索的植物艺术作品，与同名的知识库、笔记网站概念不同。

## 植物如何来到网页

植物由 Alicia 在 Houdini 中制作，再以 GLB 资源交给 Three.js 场景，通过实例化组成花野。风摆、光线和镜头可实时交互，渲染提供 WebGPU 与 WebGL2 兼容模式。

手机上呈现一小片圆形花丛，单指水平环绕 360 度，双指在 1–5 倍间缩放，平移限制在花丛附近。加载与流畅度取决于设备和浏览器，声音需要手势开启。

[阅读双语创作手记](https://digitalgarden.si/zh/about/?utm_source=github&utm_medium=referral&utm_campaign=showcase#making-of) · [手势、声音与拍摄指南](https://digitalgarden.si/zh/guide/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

## 最新实测 · 2026-10-21

- [植物实例化基准：1k、10k、100k](https://digitalgarden.si/benchmarks/threejs-webgpu-vs-webgl-instancing/?utm_source=github&utm_medium=referral&utm_campaign=showcase)：每组五次的桌面测试，提供 CSV/JSON、明确失败记录和复现包。测量数据与图表采用 CC BY 4.0；没有声称已完成手机真机测试。
- [免费 PINOC SPZ 接入 Three.js](https://digitalgarden.si/blog/viggle-jst2-spz-threejs-garden/?utm_source=github&utm_medium=referral&utm_campaign=showcase)：同一生成花园实测原生 WebGPU、原生 WebGL2 fallback 与 Spark，包含加载、遮挡检查及 Spark/WebGPURenderer 的实际初始化失败。

[机器可读的基准汇总](https://digitalgarden.si/benchmarks/data/threejs-instancing-v1-summary.json)。以上正文为英文，旧实验与原有记录保留。

## 管线实测笔记 · 2026-10-20

五篇新实测附原始测量、可复现下载包及失败与兼容性边界。文章正文为英文；结果只适用于本次工作站和指定任务，不作通用工具排名。

- [Three.js WebGPU 与 WebGL：1,000 株植物](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [Houdini USD 与 glTF：浏览器功能检查](https://digitalgarden.si/blog/houdini-usd-vs-gltf-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [Claude Code 与 Codex：同一道 Three.js 任务](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [Meshy 免费版到 Houdini：真实下载与清理](https://digitalgarden.si/blog/meshy-3d-agent-houdini-cleanup/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [Spark、GaussianSplats3D 与 Three.js 原生加载](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

[测量与复现包](https://digitalgarden.si/marketing/research/comparisons-20261020/results-2026-10-20.json)

## AI 助手的 MCP 笔记入口

公开只读 MCP 服务使用 Streamable HTTP，无需鉴权：`https://mcp.digitalgarden.si/mcp`。提供 `search_articles`、`get_article`、`list_topics`、`list_recent_articles`，可检索和阅读 Houdini、Three.js/WebGPU、glTF、OpenUSD 与 AI 工作流的中英文笔记，并返回原文规范链接。

[连接指南与示例](https://digitalgarden.si/zh/agents/?utm_source=github&utm_medium=referral&utm_campaign=showcase) · [技术博客](https://digitalgarden.si/blog/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

本仓库仅保存展示说明，托管 MCP 是远程服务，此处不含可本地安装的服务端包。

## 分享与使用

个人参观免费。分享自己的拍摄作品时，请署名 **Digital Garden by Alicia** 并附 [digitalgarden.si](https://digitalgarden.si/)。包含冬季雪声音频的短片还需保留额外署名，详见[使用说明](https://digitalgarden.si/zh/terms/)与[素材许可](https://digitalgarden.si/zh/credits/)。

[网站商业授权](https://digitalgarden.si/zh/licensing/?utm_source=github&utm_medium=referral&utm_campaign=showcase)和[品牌花园定制](https://digitalgarden.si/zh/custom-gardens/?utm_source=github&utm_medium=referral&utm_campaign=showcase)采用询价与书面约定。

此仓库用于展示作品及官网入口，不分发花园运行源码、植物原始工程，也不为这些素材提供开源许可。[权利与来源说明](RIGHTS.md)
