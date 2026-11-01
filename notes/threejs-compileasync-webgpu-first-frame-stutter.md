# Three.js r187 compileAsync: 40 Desktop Starts, Measured

> 40 fresh-profile Chrome starts compare Three.js r186 and r187: WebGPU compile time, first completion, long tasks, WebGL2 fallback and hidden-object limits.

- Author: Alicia Lu
- Published: 2026-11-01
- Canonical HTML: https://digitalgarden.si/blog/threejs-compileasync-webgpu-first-frame-stutter/

On my RTX 5070 Ti, Three.js r187 reduced WebGPU precompile time from a median **1,399.8 ms to 227.8 ms** in a controlled 48-tree grove. That is the useful result. It does not mean my production garden became six times faster, and it does not mean every startup hitch was shader compilation.

I ran 40 independent Chrome starts: r186.1 and r187.0, each with and without `compileAsync()`, on both WebGPU and the WebGL2 backend of `WebGPURenderer`. Each setting has five trials. The fixture uses one of my own Houdini trees and deliberately different materials, with a fixed camera and resolution.

The other result matters just as much: **both versions' precompile groups had no observed main-thread task of 50 ms or longer during the measured operation**. This experiment supports shorter waiting on r187. It does not support a claim that r187 alone made the loading screen responsive.

*Measured on 1 November 2026. Codex assisted the experiment, analysis and writing. This is an independent desktop fixture, not a production-site upgrade.*

![The actual controlled grove: 48 copies of an owned Houdini tree, rendered on WebGPU at 1280 by 720](https://digitalgarden.si/marketing/research/compileasync-20261101/grove.png)

*Actual browser capture. The deliberately varied shaders make pipeline preparation measurable; they are not a claim about how many materials a production garden needs. Fixture and figures by Alicia, CC BY 4.0.*

## What changed, and what I checked

[Three.js PR #34816](https://github.com/mrdoob/three.js/pull/34816) changes the shared renderer's asynchronous compilation scheduler. The pinned r187 source permits up to 16 pipelines in flight and cooperatively yields after roughly 8 ms of work. That is a scheduling budget, not a guarantee that every task or displayed frame lasts at most 8 ms.

The PR author, sunag, reported approximately 12.8 seconds to 1.5 seconds on an i9-13900K. Those are the author's measurements, not mine. I checked the [r187 release](https://github.com/mrdoob/three.js/releases/tag/r187) and the installed npm source before testing.

My instrumented WebGPU calls reached **9–10 concurrent asynchronous pipeline creations on r187**, against **one on r186**. They did not reach the source's cap of 16. The completed grove eventually created 50 render pipelines: 49 for the grove and ground, plus a separate output pipeline at the first render.

This is a whole-version comparison. r187 contains other changes, so these timings cannot isolate the causal contribution of one PR. The live Digital Garden remains on its accepted production dependencies; I did not upgrade its rendering engine to manufacture a before-and-after claim.

## The fixture and the meaning of a fresh start

The study runs on Windows 11, a Ryzen 9 9950X, 64 GB RAM and an NVIDIA GeForce RTX 5070 Ti with driver 617.42. Chrome reports version 156; the WebGPU adapter reports NVIDIA/Blackwell. Each trial saves its own browser and backend receipt.

The scene contains 48 copies of an owned 400-triangle Houdini tree, a ground plane, two lights and **49 material instances**. Different literal TSL expressions intentionally produce distinct programs. Every formal first render records **50 draw calls and 19,203 submitted triangles**, including the output pass. The fixture has no production wind scheduler, LOD transitions, shadows, environment-map preprocessing or postprocessing stack.

The viewport and renderer output are 1280 × 720 with renderer pixel ratio 1. The camera and lighting stay fixed. I used the same fixture bytes and application code for both versions, changing the import map to the pinned npm package.

Each trial launches a dedicated, fresh Chrome user-data directory with the GPU shader disk cache disabled. Treatment order rotates between rounds. That gives independent browser profiles; it **does not prove cold OS, driver or hardware caches**. I therefore describe these as fresh-profile starts rather than claiming completely cold GPU compilations.

Eight earlier pilots remain local and are excluded. During those pilots, automatic renderer-info reset could clear WebGPU counters while I waited for the GPU. The formal harness disables automatic reset and checks the actual draws and triangles. The raw receipts also retain an incidental missing-favicon 404; no application or shader errors were observed in the formal dataset.

## Three clocks that should not be combined

I separate these measurements:

1. **Precompile:** wall time around the awaited `renderer.compileAsync(scene, camera)`.
2. **First render to completion:** the first `render()` call through a completion fence. WebGPU uses `device.queue.onSubmittedWorkDone()`; WebGL2 uses `gl.finish()`.
3. **Operation to completion:** from immediately before optional precompile through that first completion fence. This is the relevant wait when deciding whether precompile adds or removes startup delay.

The operation clock begins after renderer initialization, the local GLB load/parse and fixture construction. Raw receipts also preserve navigation-to-completion and setup marks, but the headline is not a network-loading benchmark.

A GPU queue fence is **not a monitor presentation timestamp**. Nor is its entire duration a main-thread stall. In the WebGPU no-precompile trials, substantial time elapsed after the synchronous `render()` call returned while the GPU/driver completed the submitted work. Reporting all that time as a frozen JavaScript task would be wrong.

I also collected `PerformanceObserver` long-task entries and a DOM animation's RAF timestamps during preparation. Long-task entries only cover tasks at least 50 ms long. A zero below means none above that threshold, not zero JavaScript work. I did not manually measure orbit-input latency or inspect a DevTools Performance trace, so neither appears as a completed acceptance claim.

## WebGPU: shorter preparation, similar prepared first render

All numbers below are medians of five trials. Durations are milliseconds. The downloadable JSON and CSV retain every trial and range.

| Setting | Precompile | First render to completion | Operation to completion | Longest observed operation task |
| --- | --- | --- | --- | --- |
| r186.1, no precompile | — | 1263.6 | 1263.6 | 143 |
| r186.1, precompile | 1399.8 | 23.8 | 1424.6 | 0, none ≥50 ms |
| r187.0, no precompile | — | 905.9 | 906.0 | 107 |
| r187.0, precompile | 227.8 | 23.5 | 251.3 | 0, none ≥50 ms |

![Five-trial median and full range for preparation through the first completed render, on WebGPU and the forced WebGL2 fallback](https://digitalgarden.si/marketing/research/compileasync-20261101/startup.png)

The r187 WebGPU precompile range was **226.0–244.6 ms**; r186 was **1394.9–1456.4 ms**. Operation-to-completion moved from **1424.6 ms to 251.3 ms** between the two precompiled configurations, about 5.7 times shorter in this fixture.

The already-prepared first render stayed around 24 ms in both versions. The improvement was predominantly preparation, not a dramatic reduction in the cost of rendering the prepared grove.

There is also an inconvenient r186 result worth keeping: precompile made total waiting slightly longer than the no-precompile setting, even though it removed the observed ≥50 ms operation tasks. Moving work behind a loading screen and shortening overall startup are different goals.

![Precompile duration and full five-run ranges; WebGPU fell from about 1400 ms to 228 ms in this fixture](https://digitalgarden.si/marketing/research/compileasync-20261101/compile.png)

## The WebGL2 fallback is a separate result

This group uses `new WebGPURenderer({ forceWebGL: true })`. It is **not** the older `WebGLRenderer`. The common renderer scheduler is still involved, so I tested it instead of assuming that the WebGPU result applied unchanged.

| Setting | Precompile | First render to completion | Operation to completion | Longest observed operation task |
| --- | --- | --- | --- | --- |
| r186.1, no precompile | — | 1178.7 | 1178.7 | 1179 |
| r186.1, precompile | 1650.2 | 14.5 | 1664.8 | 0, none ≥50 ms |
| r187.0, no precompile | — | 991.6 | 991.7 | 992 |
| r187.0, precompile | 228.6 | 14.8 | 243.2 | 0, none ≥50 ms |

Here the synchronous no-precompile path produced a much longer main-thread task than WebGPU. Precompiling again changed where the work happened; the r187 precompile group also waited much less overall.

I did not instrument WebGL driver compilation concurrency and did not test the legacy renderer's `KHR_parallel_shader_compile` behavior. A zero WebGPU-pipeline count in these WebGL2 receipts means that WebGPU API was not used, not that shaders needed no work.

## A smooth post-ready loop does not describe startup

Every trial records the next 120 RAF intervals **after** the first completion fence: 4,800 intervals across the 40 formal starts. All eight settings had zero intervals over 50 ms in that window.

That result does not cancel the startup waits in the tables. Starting a frame-time chart after expensive work can make every configuration look smooth. I retain the startup heartbeat and long-task records separately so that the measurement window is explicit.

![Neither version's precompile group produced an observed operation task at least 50 ms long; zero is a thresholded observation](https://digitalgarden.si/marketing/research/compileasync-20261101/longtasks.png)

## Hidden trees still needed pipelines when revealed

I ran four additional diagnostic starts, outside the 40-trial timing denominator: each version with a visibility/frustum control and a detached-group/target-scene control.

For the visibility control, one tree had `visible = false` and another was positioned outside the camera frustum during precompile. Both versions prepared 47 pipelines. The first render added the output pipeline. Revealing the two trees then created **two additional pipelines**.

So I would not tell an artist that calling `compileAsync()` once prepares everything that might ever appear. In this fixture it follows visibility and camera coverage. Streaming, alternate materials and later scene changes still need explicit planning.

The target-scene control moved the tree meshes into a detached group, compiled that group with the scene's lighting context and then attached it:

```javascript
// Verified on both pinned versions in this controlled fixture.
await renderer.compileAsync(nextGroup, camera, scene);
scene.add(nextGroup);
```

Both versions prepared 48 pipelines for the detached trees; the ground and output pipelines were added when rendering the full scene. The three-argument signature is supported by the [pinned renderer source](https://github.com/mrdoob/three.js/blob/r187/src/renderers/common/Renderer.js), rather than being copied from a different renderer's API.

This does not verify arbitrary moving lights, shadows, render targets or a postprocessing pass graph. Match the compilation context to the eventual render and measure new pipeline creation when the object appears.

## Where I would use this in a garden loader

First finish the scene's intended materials, lights and camera, then await precompile while an honest loading indicator is visible. Keep first-render completion and the total preparation wait in the same log. For incoming groups, supply the destination scene so that compiling without its lights does not become a misleading warmup.

Avoid automatically making every invisible object visible in the production scene just to warm it. That can change what the user sees or submit an unexpectedly large workload. A controlled offscreen warmup is its own integration task, with the eventual render format, material state and coverage verified.

This fixture deliberately avoids large image textures. It cannot show that `compileAsync()` removes texture decode or upload cost, GLB download, Draco/Meshopt decode, or instance-buffer preparation. Those belong to the separate [GLB loading and first-frame study](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/). Mesh-detail tradeoffs are covered in the [vegetation LOD measurements](https://digitalgarden.si/blog/threejs-webgpu-vegetation-lod/), and backend comparisons in the [instanced plant study](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/). Those older articles and their dates are unchanged.

Before a production upgrade, read the [r186-to-r187 migration guide](https://github.com/mrdoob/three.js/wiki/Migration-Guide#186--187). This test does not accept every migration change, PMREM output difference or application-specific shader. Phones, Safari/Firefox, a second GPU, shadows, texture-heavy scenes and the production garden remain untested.

## Data and reproduction

[Study ZIP](https://digitalgarden.si/marketing/research/compileasync-20261101/compileasync-study-20261101.zip) · [Package size and SHA-256](https://digitalgarden.si/marketing/research/compileasync-20261101/reproduction-receipt.json) · [Results JSON and ranges](https://digitalgarden.si/marketing/research/compileasync-20261101/analysis.json) · [Trial CSV](https://digitalgarden.si/marketing/research/compileasync-20261101/results.csv) · [Source hashes](https://digitalgarden.si/marketing/research/compileasync-20261101/source-manifest.json) · [Environment and limits](https://digitalgarden.si/marketing/research/compileasync-20261101/ENVIRONMENT.md).

The package includes the pinned dependency manifest, owned GLB, measurement harness, all 40 formal receipts, four final diagnostic receipts, selected browser captures, figures and four saved diagnostic recordings. It excludes browser profiles, account information, production assets and the eight local pilots. An initial control runner used the old entrypoint before a local server restart, so its target-scene claims were rejected and all four controls rerun with an execution marker. One earlier video export also failed. Both issues remain documented rather than being counted as successful final controls.

Code, fixture and figures are by Alicia, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/); Three.js retains its MIT license. The result is a measured reason to test r187's preparation path on this hardware, with explicit clocks and coverage limits. It is not a promise that a version bump removes every garden loading hitch.

[Explore the Three.js rendering field guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/). The ZIP preserves the measured manuscript snapshot; this canonical page also carries the current navigation links.
