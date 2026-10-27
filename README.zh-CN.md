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

## 统一风场实测 · 2026-10-27

新增54组草地与树木风场试验，包含原始数据、图表、动画、复现包和Codex辅助披露；共31篇英文笔记。

- [Three.js WebGPU Grass Wind: One Field, 54 Desktop Trials](notes/threejs-webgpu-grass-wind-field.md) · [官网原文](https://digitalgarden.si/blog/threejs-webgpu-grass-wind-field/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

## 三篇桌面实测 · 2026-10-26

新增 132 次加载、21 个 Houdini 进程和 24 组植被试验。英文全文包含原始结果、复现下载、Codex 辅助披露及未验证范围；该次发布后共 30 篇公开英文笔记。

- [Draco vs Meshopt and KTX2: 132 Loading Trials in Three.js](notes/draco-vs-meshopt-ktx2-benchmark.md) · [官网原文](https://digitalgarden.si/blog/draco-vs-meshopt-ktx2-benchmark/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [RTX 5070 Ti in Houdini 22: Karma, Vellum and a Small Dense Pyro Test](notes/rtx-5070-ti-houdini-benchmark.md) · [官网原文](https://digitalgarden.si/blog/rtx-5070-ti-houdini-benchmark/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [1,600 Trees in Three.js: Mesh LOD and 8/16-View Impostors Measured](notes/threejs-webgpu-vegetation-lod.md) · [官网原文](https://digitalgarden.si/blog/threejs-webgpu-vegetation-lod/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

## 完整技术文章 · 2026-10-25

以下 27 篇为数字花园已公开英文文章的文档副本，保留原始日期、测量、AI 辅助披露和实验限制。每篇注明官网原文，图片及复现资料继续从官网读取。花园运行代码与原始资产的权利边界保持不变。

| 原始日期 | 文章 | 官网 |
| --- | --- | --- |
| 2026-10-25 | [Three.js Terrain Scatter: Fix Floating, Then Fix Clumping](notes/threejs-scatter-objects-uneven-terrain.md) | [原文](https://digitalgarden.si/blog/threejs-scatter-objects-uneven-terrain/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-24 | [Spirula Studio to Three.js: An Official Garden Sample Test](notes/spirula-studio-garden-splat-threejs.md) | [原文](https://digitalgarden.si/blog/spirula-studio-garden-splat-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-24 | [Three.js GLB Loading: Cache, Decode and the First GPU Frame](notes/threejs-glb-loading-cache-first-frame.md) | [原文](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-24 | [MuTracer with a Houdini Plant: Colors, Alpha, Instances and 64 Samples](notes/mutracer-houdini-plant-path-tracing.md) | [原文](https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-24 | [Houdini vs Geometry Nodes: Exporting the Same Plant Data](notes/houdini-vs-geometry-nodes-plant-export.md) | [原文](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-23 | [SimReady USD to GLB: Blender and Houdini Export Traps, Tested in WebGPU](notes/simready-usd-to-glb-threejs-webgpu.md) | [原文](https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-23 | [Houdini Assets vs Runtime Geometry: slop-newyork and My Garden, Profiled](notes/houdini-vs-code-procedural-threejs-slop-newyork.md) | [原文](https://digitalgarden.si/blog/houdini-vs-code-procedural-threejs-slop-newyork/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-22 | [I Tested img2interior with Codex: A Room and a 95 MB Runtime](notes/img2interior-codex-threejs-garden.md) | [原文](https://digitalgarden.si/blog/img2interior-codex-threejs-garden/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-21 | [Rendering an Interactive 3D Garden in Three.js: WebGPU, Plants, Formats and Splats](notes/threejs-3d-garden-rendering.md) | [原文](https://digitalgarden.si/blog/threejs-3d-garden-rendering/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-21 | [Viggle JST-2 SPZ in Three.js: A Free PINOC Garden Test](notes/viggle-jst2-spz-threejs-garden.md) | [原文](https://digitalgarden.si/blog/viggle-jst2-spz-threejs-garden/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-20 | [Houdini USD vs glTF: A Two-Plant Three.js Test](notes/houdini-usd-vs-gltf-threejs.md) | [原文](https://digitalgarden.si/blog/houdini-usd-vs-gltf-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-20 | [Three.js WebGPU vs WebGL: Testing 1,000 Instanced Plants](notes/threejs-webgpu-vs-webgl-instanced-plants.md) | [原文](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-20 | [Spark vs GaussianSplats3D vs Native Three.js: An Integration Test](notes/spark-vs-gaussiansplats3d-threejs.md) | [原文](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-20 | [Claude Code vs Codex: One Reproducible Three.js Wind Task](notes/claude-code-vs-codex-threejs.md) | [原文](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-20 | [Meshy Free to Houdini: Testing the Download and Cleanup Path](notes/meshy-3d-agent-houdini-cleanup.md) | [原文](https://digitalgarden.si/blog/meshy-3d-agent-houdini-cleanup/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-19 | [How I Built a WebGPU 3D Garden with Houdini and Three.js](notes/webgpu-garden-houdini-threejs.md) | [原文](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-19 | [Gaussian Splats in Three.js: Native Renderer and Spark Tested](notes/gaussian-splats-threejs.md) | [原文](https://digitalgarden.si/blog/gaussian-splats-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-19 | [Houdini to Unreal with USD: A Tested Stage, HDA and Material Pipeline](notes/houdini-unreal-usd.md) | [原文](https://digitalgarden.si/blog/houdini-unreal-usd/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-19 | [Procedural Plants for the Web: Houdini Geometry and Wind Data](notes/houdini-procedural-plants-realtime-web.md) | [原文](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-19 | [Houdini to Three.js: Testing glTF Compression and Instancing](notes/houdini-gltf-threejs-optimization.md) | [原文](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-19 | [ComfyUI 3D Workflow: From a Million-Triangle Draft to a DCC-Tested Prop](notes/comfyui-3d-workflow.md) | [原文](https://digitalgarden.si/blog/comfyui-3d-workflow/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-19 | [OpenUSD for Artists: A First Stage Tested in Houdini Solaris](notes/openusd-pipeline-beginners.md) | [原文](https://digitalgarden.si/blog/openusd-pipeline-beginners/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-19 | [Houdini ML Volume Upres for Pyro: Pretrained Models and a Real Training Run](notes/houdini-ml-volume-upres-pyro.md) | [原文](https://digitalgarden.si/blog/houdini-ml-volume-upres-pyro/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-18 | [I Tested machine-skill with Codex: Two Three.js Pump Simulations](notes/codex-machine-skill-threejs.md) | [原文](https://digitalgarden.si/blog/codex-machine-skill-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-16 | [OpenUSD in Three.js: A Houdini Solaris Test of USD Web Viewer](notes/openusd-threejs-usd-web-viewer.md) | [原文](https://digitalgarden.si/blog/openusd-threejs-usd-web-viewer/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-14 | [Google Gemini 4 Argon for Three.js: What Community Tests Actually Show](notes/gemini-4-argon-threejs.md) | [原文](https://digitalgarden.si/blog/gemini-4-argon-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| 2026-10-14 | [Claude Code for 3D Artists: A Real Houdini Pipeline Setup](notes/claude-code-for-3d.md) | [原文](https://digitalgarden.si/blog/claude-code-for-3d/?utm_source=github&utm_medium=referral&utm_campaign=showcase) |

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
