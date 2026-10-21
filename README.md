# Digital Garden by Alicia

A free interactive 3D flower garden in your browser. Explore four seasons, change the light and colours, listen to nature sounds, and make a flower photograph or a short clip.

**[Visit the garden](https://digitalgarden.si/?utm_source=github&utm_medium=referral&utm_campaign=showcase)** · [中文介绍](README.zh-CN.md) · [What you can do](https://digitalgarden.si/virtual-garden/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

![Spring lily-of-the-valley flowers in Digital Garden by Alicia](https://digitalgarden.si/marketing/spring.jpg)

## Explore and photograph

- Choose spring, summer, autumn or winter, by day or night.
- Move closer to a flower, adjust the wind and recolour plants and light.
- Choose a frame and focus for a still photo, or record a clip up to 10 seconds.
- Save your capture or use native file sharing on a supported browser.
- No account or installation is required. Preferences and captures stay on your device until you choose to share them.

This is an explorable botanical artwork. It is distinct from the personal knowledge-base meaning of “digital garden.”

## Pick a season

| Scene | Plants to look for | Open |
| --- | --- | --- |
| Spring meadow | Lily of the valley, grape hyacinth and fern | [Day](https://digitalgarden.si/play/?season=spring&night=0&lang=en&utm_source=github&utm_medium=referral&utm_campaign=showcase) · [Night](https://digitalgarden.si/play/?season=spring&night=1&lang=en&utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| Summer garden | Lavender, iris and dandelion | [Day](https://digitalgarden.si/play/?season=summer&night=0&lang=en&utm_source=github&utm_medium=referral&utm_campaign=showcase) · [Night](https://digitalgarden.si/play/?season=summer&night=1&lang=en&utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| Autumn field | Wheat, sunflower and cosmos | [Day](https://digitalgarden.si/play/?season=autumn&night=0&lang=en&utm_source=github&utm_medium=referral&utm_campaign=showcase) · [Night](https://digitalgarden.si/play/?season=autumn&night=1&lang=en&utm_source=github&utm_medium=referral&utm_campaign=showcase) |
| Winter snowfield | Snowdrop, hellebore and camellia | [Day](https://digitalgarden.si/play/?season=winter&night=0&lang=en&utm_source=github&utm_medium=referral&utm_campaign=showcase) · [Night](https://digitalgarden.si/play/?season=winter&night=1&lang=en&utm_source=github&utm_medium=referral&utm_campaign=showcase) |

## Made with Houdini and Three.js

Alicia authored the plants in Houdini. GLB assets become an instanced flower scene in Three.js, with live wind, lighting and camera controls. WebGPU rendering has a WebGL2 compatibility mode.

On a phone, a smaller circular patch keeps nearby flowers available for a full horizontal orbit. Pinch to zoom from 1× to 5×; movement across the ground is limited. Device and browser performance varies. Sound starts after a user gesture.

[Read the making-of notes](https://digitalgarden.si/about/?utm_source=github&utm_medium=referral&utm_campaign=showcase#making-of) · [Controls, audio and capture guide](https://digitalgarden.si/guide/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

## New measurements — 21 October 2026

- [Plant instancing benchmark: 1k, 10k and 100k plants](https://digitalgarden.si/benchmarks/threejs-webgpu-vs-webgl-instancing/?utm_source=github&utm_medium=referral&utm_campaign=showcase): five-repeat desktop matrix with CSV/JSON, explicit failures and a reproduction harness. The measurement data and chart are CC BY 4.0; no physical-phone results are claimed.
- [Free PINOC SPZ in Three.js](https://digitalgarden.si/blog/viggle-jst2-spz-threejs-garden/?utm_source=github&utm_medium=referral&utm_campaign=showcase): one generated garden tested with native WebGPU, native WebGL2 fallback and Spark. Includes actual loading/occlusion checks and the recorded Spark/WebGPURenderer initialization failure.

[Machine-readable benchmark summary](https://digitalgarden.si/benchmarks/data/threejs-instancing-v1-summary.json)

## Tested pipeline notes — 20 October 2026

Five new tests include original measurements, reproducible downloads and explicit failure/compatibility limits. These are bounded workstation and pipeline cases, not universal tool rankings.

- [Three.js WebGPU vs WebGL: 1,000 plants](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [Houdini USD vs glTF: browser feature checks](https://digitalgarden.si/blog/houdini-usd-vs-gltf-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [Claude Code vs Codex: one paired Three.js task](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [Meshy Free to Houdini: the actual download and cleanup](https://digitalgarden.si/blog/meshy-3d-agent-houdini-cleanup/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
- [Spark vs GaussianSplats3D vs native Three.js](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

[Data and reproduction packages](https://digitalgarden.si/marketing/research/comparisons-20261020/results-2026-10-20.json)

## MCP notes for AI assistants

Digital Garden also provides a public, read-only MCP endpoint (Streamable HTTP, no authentication):

`https://mcp.digitalgarden.si/mcp`

Tools: `search_articles`, `get_article`, `list_topics`, `list_recent_articles`. Search or read Alicia’s English and Chinese notes on Houdini, three.js/WebGPU, glTF, OpenUSD and AI workflows, with canonical source URLs for citation.

[Connection guide and examples](https://digitalgarden.si/agents/?utm_source=github&utm_medium=referral&utm_campaign=showcase) · [Technical blog](https://digitalgarden.si/blog/?utm_source=github&utm_medium=referral&utm_campaign=showcase)

This repository contains showcase documentation. The hosted MCP service is remote-only; this repository does not contain an installable server package.

## Sharing and commercial use

Personal exploration is free. For sharing your captures, credit **Digital Garden by Alicia** and link to **https://digitalgarden.si/**. Winter clips containing the snow recording also need its third-party attribution; see the [usage guide](https://digitalgarden.si/terms/) and [credits](https://digitalgarden.si/credits/).

Commercial website licensing and custom garden commissions are available by inquiry. Scope and third-party rights are described on the [licensing](https://digitalgarden.si/licensing/?utm_source=github&utm_medium=referral&utm_campaign=showcase) and [custom garden](https://digitalgarden.si/custom-gardens/?utm_source=github&utm_medium=referral&utm_campaign=showcase) pages.

This repository presents the artwork and links to the live experience. It does not distribute the garden runtime, plant source files or an open-source license for those materials. See [rights and sources](RIGHTS.md).

**Official website:** [digitalgarden.si](https://digitalgarden.si/?utm_source=github&utm_medium=referral&utm_campaign=showcase) · **Artist:** [Alicia](https://digitalgarden.si/about/?utm_source=github&utm_medium=referral&utm_campaign=showcase)
