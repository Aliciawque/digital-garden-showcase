# Three.js Gaussian Splats: Gaussian-Splat-Lite, Spark and Radix on One Desktop

> 30 RTX 5070 Ti runs compare Gaussian-Splat-Lite, Spark and Radix with matched 500k and 2M SPZ files, first submission and browser frame pacing.

- Author: Alicia Lu
- Published: 2026-11-04
- Canonical HTML: https://digitalgarden.si/blog/threejs-gaussian-splat-renderer-comparison/

## Key takeaways

- At 2M splats Spark on Three r187 reached the first submitted-work wait in a 475.3 ms median. Radix on r186 took 1,106.2 ms.
- The warm orbit had a 16.7 ms median rAF interval across all ten groups. Browser cadence cannot rank GPU throughput.

I compared Gaussian-Splat-Lite 1.6.2, Spark 2.4.0 and Radix 0.0.1 using the same fixed SPZ file at each size, with five hardware-backed paths on an RTX 5070 Ti. Spark on r187 reached the first submitted-work wait sooner than the other paths at both sizes. The warm camera orbit paced near 60 Hz in every group. That cadence cannot rank GPU throughput.

![Gaussian-Splat-Lite WebGPU showing the 2M synthetic SH0 fixture and the same opaque cube](https://digitalgarden.si/marketing/research/splat-renderers-20261104/gsl-gpu-2000000.png)

## What was actually shared

The two SPZ v3 files contained 500,000 and 2,000,000 view-independent splats from the same seeded, synthetic flower-bed generator. File sizes were 6,443,429 and 25,759,887 bytes. Every path decoded the expected count and rendered the same opaque cube, camera, 1280 × 720 output and orbit. This is an owned stress fixture, separate from a photographic scan and the live garden.

I used Three.js 0.187.0 for Gaussian-Splat-Lite and the main Spark run. The first Radix pilot on r187 hit Three.js's compute dispatch size error. Radix 0.0.1 then ran with its compatible 0.186.1 setup. An extra Spark 2.4.0 path also ran on 0.186.1. The derived local bundles for the two r186 paths only redirect Three.js imports to the pinned alias and remove source-map references. The compatibility failure is preserved. [Gaussian-Splat-Lite repository](https://github.com/WilliamLiu-1997/Gaussian-Splat-Lite) · [Spark](https://sparkjs.dev/docs/splat-mesh/) · [Radix](https://github.com/sparkjsdev/radix).

## Setup and frame pacing

The clean repeat ran 30 fresh contexts in one Chrome 156.0.8078.80 process, with three contexts per size and path and rotating order. WebGPU reported NVIDIA Blackwell, and WebGL2 reported the RTX 5070 Ti through ANGLE D3D11. Antialiasing was disabled and the pixel ratio was 1. Each run included 45 warm-up frames, then 120 recorded requestAnimationFrame intervals during the orbit. The earlier run had an unclassified 404 console entry in one context and overlapped the start of the Unreal task. It stays separate from the clean repeat.

Startup measures module execution after the common dynamic Three.js import through the first render call and completion wait for submitted work. It includes backend setup and renderer-specific imports and loading. It stops at WebGPU queue completion or WebGL finish. Any asynchronous sort can still be pending. The later capture and full warmed orbit verify the decoded scene separately.

| Renderer path | Three.js | 0.5M first wait (ms) | 2M first wait (ms) | 0.5M warm p95 (ms) | 2M warm p95 (ms) |
| --- | --- | --- | --- | --- | --- |
| Gaussian-Splat-Lite WebGPU | 0.187.0 | 351.2 | 847.4 | 16.7 | 16.8 |
| Gaussian-Splat-Lite WebGL2 backend | 0.187.0 | 250.0 | 648.9 | 16.8 | 16.8 |
| Spark WebGL2 | 0.187.0 | 227.7 | 475.3 | 16.8 | 16.8 |
| Radix WebGPU | 0.186.1 | 767.1 | 1,106.2 | 16.7 | 16.7 |
| Spark WebGL2 control | 0.186.1 | 254.3 | 515.7 | 16.8 | 16.8 |

The cells report the median of three run-level measurements. All ten groups had a 16.7 ms median and 16.8 ms p99, rounded to one decimal. The only intervals above 25 ms were two in the 500,000-splat Gaussian-Splat-Lite WebGL2 group. At that size its first-wait values also ranged from 246.8 to 572.8 ms. Fresh contexts do not clear the browser process, GPU driver or server transform caches. The startup table is a local observation, not a cold network benchmark.

![Startup first-submitted-work wait and warm p95 rAF per renderer and file size](https://digitalgarden.si/marketing/research/splat-renderers-20261104/comparison-chart.png)

## Scope and what I would test next

These are renderer paths as configured, with different sort and upload implementations. No measured GPU timestamps, total VRAM, sort latency or pixel-accuracy ranking is inferred from rAF. All runs used fully loaded SPZ, so the table says nothing about paged RAD, SOG or LCC2 streaming. The raw renderer counters and per-frame intervals are in the archive.

![Radix WebGPU capture of the same 2M file on the r186 path](https://digitalgarden.si/marketing/research/splat-renderers-20261104/radix-2000000.png)

[Reproduction archive with deterministic fixture generator and raw evidence](https://digitalgarden.si/marketing/research/splat-renderers-20261104/splat-renderers-reproduction.zip). The [earlier Spark and Radix study](https://digitalgarden.si/blog/spark-radix-vs-spark-webgpu-benchmark/) used a separate protocol and remains a separate result. Codex implemented the harness, ran the tests and drafted this article.

Related study: [Digital Garden field note](https://digitalgarden.si/blog/houdini-procedural-gaussian-splatting/).

### Raw measurement files

- [runs.json](https://digitalgarden.si/marketing/research/splat-renderers-20261104/runs.json)
- [summary.json](https://digitalgarden.si/marketing/research/splat-renderers-20261104/summary.json)

## Questions

### Why did Radix use a different Three.js version?

Radix 0.0.1 hit a compute dispatch API error on r187. The measured Radix path and an extra Spark control used r186.1.
