# How I Built a WebGPU 3D Garden with Houdini and Three.js

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Inside Alicia’s interactive 3D garden: Houdini assets, Three.js TSL, measured WebGPU and WebGL2 results, and the limits of this desktop test.

- Author: Alicia Lu
- Published: 2026-10-19
- Updated: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/

The [interactive garden](https://digitalgarden.si/play/) starts in Houdini and runs in Three.js. Houdini makes the plant geometry and its deformation attributes; the browser places, lights and animates those plants. On 19 October 2026, I tested the current garden code on both WebGPU and WebGL2, then built a separate small plant to check the export pipeline. The useful result is a working authoring boundary, not a claim that switching APIs makes every scene faster.

This note is part of the [Three.js 3D garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/), which groups the renderer, plant, format and splat tests in reading order.

![The actual spring garden running through Three.js WebGPURenderer, with the active backend visible in the test harness](https://digitalgarden.si/marketing/research/webgpu-series-20261019/garden-webgpu.png)

## What the browser actually renders

The garden has four seasonal compositions, day and night transitions, shader driven wind, plant interaction and a photography mode. It uses Three.js 0.186.1 and TSL materials. The renderer is WebGPURenderer, which can use its WebGPU backend or its WebGL2 fallback. The active backend is checked after renderer initialization; the constructor name alone cannot establish which one ran.[^1]

Houdini exports reusable plant assets rather than a giant frozen scene. The browser owns placement, camera movement, screen space detail decisions and interactions. That keeps artistic geometry work in the DCC while allowing the website to respond without a Houdini installation. A stem is authored once and placed many times. Its wind attributes describe how it may deform; they do not contain a frame by frame animation cache.

The production wind system keeps roots fixed and bends a shared stem while preserving coherent flower heads. It also deforms normals. The small downloadable experiment uses a much simpler sine displacement to make attribute loss easy to see. Its shader is a teaching fixture, not the production wind implementation. A later [MuTracer WebGPU path tracer](https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/) test rendered a frozen Houdini plant as a still: vertex colors, three instances and alpha masks survived on WebGPU, and the WebGL attempt lost its context. That image does not convert the animated wind in this garden.

## A measured comparison of the current garden

I imported the existing Experience module from source commit `9c25ecc6ab2e` into a local measurement harness. Each backend ran the same 30 second partial orbit after a 1.5 second warmup. The canvas was 1920 × 1080 at device pixel ratio 1; the browser viewport was 1280 × 900 so the desktop scene policy stayed active. The spring scene contained 162,420 plant placements.

The machine used an NVIDIA RTX 5070 Ti. The in app browser reported Chromium 155; WebGPU reported NVIDIA Blackwell, and the WebGL2 fallback reported ANGLE over Direct3D 11. Both tabs were visible. Other applications were not closed, and this was one sample per backend.

| Active backend | Mean FPS | 95th percentile frame time | Mean render calls | Mean triangles, including passes |
| --- | --- | --- | --- | --- |
| WebGPU | 54.66 | 33.4 ms | 67.22 | 9.49 million |
| WebGL2 fallback | 58.40 | 16.8 ms | 67.24 | 9.48 million |

WebGL2 was faster in this particular run. The display was close to a 60 Hz ceiling, the detail scheduler was active, and the triangle totals vary slightly as detail transitions settle. These measurements do not isolate API overhead or establish a universal winner. They do establish that both backends render the current garden and that this scene does not justify an unconditional “WebGPU is faster” headline.

Both runs reused 36 assets from the browser’s garden cache, with zero model downloads. First frame observations were 1.59 seconds for WebGPU and 10.81 seconds for WebGL2. Those are single initialization observations with different backend pipeline work, not controlled cold load or CDN benchmarks. They should not be used to predict a visitor’s loading time. A later [Three.js GLB cache benchmark](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/) published twenty controlled rows for one texture-free plant, separating a fresh URL from an immediate cache hit. It does not replace these single garden initialization observations. A later desktop profile counted the garden’s full transfer, including seasons loaded after the first frame, and set the spring scene beside a [procedural city built at runtime](https://digitalgarden.si/blog/houdini-vs-code-procedural-threejs-slop-newyork/). Both ran at 1920×1080 on WebGPU; they are different scenes, not a controlled pipeline comparison.

## Where the scene budget goes

Instancing shares a plant geometry across many placements. It does not eliminate triangle processing, alpha testing or pixel shading. In the separate 1,000 plant fixture, 6,056 triangles per plant still produced about 6.06 million scene triangles. Reducing the plant to 2,116 triangles brought that to about 2.12 million. Both configurations remained near 60 FPS on this desktop, so the geometry reduction is clearer evidence than a rounded FPS comparison. A later [WebGPU and WebGL plant matrix](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/) repeated one plant as separate meshes, InstancedMesh and BatchedMesh on three renderer paths, and compared submission cost with the actual draw counters.

The garden’s current desktop policy keeps two decoded seasons resident and uses a bounded 160 MiB model cache on disk. Background preparation pauses around activity and photography. The ordinary scene has a scene pass and bloom; depth of field belongs to photography mode. These are deliberate budgets and lifecycle choices, not benefits supplied automatically by WebGPU.

Reversed depth is enabled only for the actual WebGPU backend in this project. Earlier fallback testing produced a blank post processing result with the reversed path. That is a project compatibility decision, not a general statement that WebGL2 cannot support any reversed depth technique.

## Test the boundary before scaling the garden

The companion export test caught a practical failure: native Houdini GLB and the tested glTF Transform Meshopt path preserved `_STIFF` and `_PHASE`, while the tested gltfpack paths dropped them. A smaller file that loses the data required by wind is not an accepted replacement. The production garden assets were preserved during this experiment.

The most useful sequence is to validate one plant, reopen its editable Houdini scene, inspect the exported attributes, load the compressed candidate with the real browser decoders, and only then test repeated placements. A renderer screenshot alone cannot prove that deformation data survived compression.

Read the measured details in [Houdini glTF optimization](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/), [procedural plants and wind attributes](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/), and [Gaussian splats in Three.js](https://digitalgarden.si/blog/gaussian-splats-threejs/). The Gaussian article uses a separate synthetic fixture; splats are not part of the production garden. A later [Spark versus GaussianSplats3D integration](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/) ran that fixture on native Three.js, Spark and GaussianSplats3D, and recorded which encodings actually drew.

## Reproduce the experiment and understand its limits

A later [free PINOC / JST-2 garden test](https://digitalgarden.si/blog/viggle-jst2-spz-threejs-garden/) places an actual million-point SPZ environment beside an owned Houdini plant. It reports native WebGPU, native WebGL2 and Spark results, with a separate renderer-integration failure. That backdrop remains an independent research fixture.

Codex executed the native scripts and browser checks. The [compact measurement receipts](https://digitalgarden.si/marketing/research/webgpu-series-20261019/results-2026-10-19.json) preserve versions, counts and methods alongside the rendered evidence.

The [experiment download](https://digitalgarden.si/marketing/research/webgpu-series-20261019/webgpu-garden-tests.zip) contains the original small plant generator, compact results, owned fixtures and independent asset/splat viewers. It excludes the private garden engine and production models. The full garden measurements are supplied as dated results with their method, not as a promise that the private application is included in the download.

No physical phone was tested. Responsive layout checks are not phone GPU tests. I also did not record historical TSL migration hours, measure total driver VRAM, or test every browser version. Those remain unmeasured rather than being filled with estimates. The concrete outcome is a working current garden on both backends, and a set of smaller tests that expose where its asset pipeline can fail.

[^1]: Three.js, [WebGPURenderer documentation](https://threejs.org/docs/pages/WebGPURenderer.html), checked 19 October 2026.
