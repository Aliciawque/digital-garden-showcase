# Three.js WebGPU vs WebGL: Testing 1,000 Instanced Plants

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Nine measured Three.js renderer and submission paths: 1,000 plants, repeated frame samples, CPU render time, GPU probes and actual draw calls.

- Author: Alicia Lu
- Published: 2026-10-20
- Updated: 2026-10-25
- Canonical HTML: https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/

With this plant fixture, instancing reduced draw submission on every tested backend. Switching to WebGPU did not produce a meaningful FPS lead: all nine paths were close to the display's 60 Hz ceiling. The useful distinction was the submitted representation and the work inside renderer.render(), rather than the API name.

For the related asset, format and splat experiments, see the [Three.js 3D garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

This test supports the Houdini asset workflow behind [Alicia's interactive 3D garden](https://digitalgarden.si/play/). Codex operated the independent scripts and browser on 20 October 2026. It did not change the production garden or benchmark a physical phone.

![Measured submission cost and actual draw counters for all nine paths](https://digitalgarden.si/marketing/research/comparisons-20261020/plants-submission.png)

## What was kept equal

Three.js was pinned to 0.186.1. One owned procedural grape hyacinth supplied 6,056 triangles, repeated 1,000 times: 6,056,000 scene triangles. The same placement matrices, geometry, unlit vertex colours, double-sided material, camera path and bounded root-anchored sine displacement were used throughout. The classic renderer used an equivalent GLSL injection; the node renderer used TSL. This is a small teaching wind, not the garden's production deformation and normal system.

The three renderer paths were classic WebGLRenderer, WebGPURenderer on actual WebGPU, and WebGPURenderer with forceWebGL enabled. Each used separate Mesh objects, InstancedMesh, then BatchedMesh. The active backend was inspected after initialization; a WebGPURenderer constructor alone does not prove WebGPU ran.[^1]

The canvas was 1280 × 720 at renderer DPR 1, without antialiasing. The browser reported Chromium 155 on Windows and an RTX 5070 Ti. Each accepted cell ran three complete 10-second warm-ups, discarded, followed by five complete 10-second measurements along the same partial orbit. The archived results report the median of each run's summary. The slowest 1% of frame intervals define the 1% low FPS.

Frustum culling was disabled for every object; BatchedMesh per-object culling and sorting were disabled too. That keeps the submitted set equal. It also means this is not a comparison of each class's default visibility policy. Other applications remained open; these are workstation observations rather than isolated maximum-throughput measurements. GPU experiments ran sequentially. Draw representation and counters are clearer evidence than small cross-backend timing differences under background load.

The first full matrix used random Y rotations and 20-second trials. Review caught a parity problem: classic GLSL displaced before instancing, while TSL positionNode ran after the instance/batch transform. The same expression could therefore bend in different world directions. I preserved that matrix as a rejected comparison, removed rotations from all nine paths, and reran the complete matrix with 10-second trials. The chart and archived tables use only this matched-transform rerun. This shorter run does not certify long-duration frame pacing.

## Full dataset and 10k/100k runs

The current tables and machine-readable measurements live in the [Three.js plant instancing benchmark v1](https://digitalgarden.si/benchmarks/threejs-webgpu-vs-webgl-instancing/), with 1k, 10k and 100k tests, static and moving cameras, and explicit failed trials. The v1 page reruns 1k using at least five seconds and 300 warm-up frames followed by thirty seconds or 1,800 intervals, with five repeats and alternating backend order. Each sample uses a new visible test document. It does not reuse this article's shorter measurements.

This article and its chart retain the findings of the 20 October nine-path test. Its historical numbers and original protocol remain in the [original measurement archive](https://digitalgarden.si/marketing/research/comparisons-20261020/plants-tests.zip). The two protocols must not be pooled into one performance table.

The node renderer's totals include its final output pass, adding one call and one triangle. Classic WebGL's counters do not contain that extra pass. Neither counter should be relabelled as a direct browser API call count.

The BatchedMesh result is especially useful. In this pinned WebGPU path, the backend loops over the batch's draw commands. Its counter remains at 1,001 calls even though the JavaScript submission can cost much less than traversing 1,000 individual Mesh objects. Classic WebGL used the available multi-draw path and reported one call. A class called BatchedMesh does not guarantee a single draw on every backend.[^2]

InstancedMesh is a clear fit for this repeated identical geometry. BatchedMesh has a different purpose when many geometries share compatible material state; the present one-geometry case does not establish how it behaves in every mixed scene. Merging was not part of this nine-cell matrix.

## What the timers do and do not measure

CPU render time surrounds renderer.render(). It includes JavaScript and backend submission work, not completed GPU execution. A separate 30-frame GPU probe used EXT_disjoint_timer_query_webgl2 on classic WebGL and resolveTimestampsAsync on the node backends. Invalid or unavailable samples stay absent in the receipts. Repeated discrete WebGPU probe values are retained as diagnostics, not used to rank small timing differences. The probe is outside the FPS trials and is not total VRAM or end-to-end latency.

Cell setup to first submitted frame was also recorded once per cell, including compilation, after the asset had already loaded. Those values are not device-cold startup or public CDN loading times. A later [meshopt GLB loading](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/) test times the file itself across twenty WebGPU trials, separating a fresh URL, a cache hit, decoding and the first GPU frame, and it keeps the first-run compilation outlier. JavaScript heap snapshots are process observations without controlled garbage collection, so I do not rank memory efficiency from them.

The display cap hides how much headroom a fast path has. It would be misleading to call 60.00 versus 59.99 FPS a renderer victory, or to extrapolate this desktop result to a phone. This experiment also does not establish a universal instance-count threshold, transparent-foliage overdraw cost or physical-device power consumption. A different workload sits outside this one-plant matrix. The [WebGPU performance of two complete scenes](https://digitalgarden.si/blog/houdini-vs-code-procedural-threejs-slop-newyork/) profiles slop-newyork and the spring garden at 1080p, where a lower draw-call count did not by itself produce a shorter frame. A still [WebGPU plant path tracing](https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/) test with MuTracer rendered three tinted instances of one Houdini plant. The WebGL attempt lost its context and produced no finished image. Those samples are not rows in this raster matrix.

A separate [terrain placement comparison](https://digitalgarden.si/blog/threejs-scatter-objects-uneven-terrain/) measures the CPU work before rendering: downward rays, BVH, surface sampling and baked Houdini points. Its 70 placement rows are separate from this renderer matrix, and minimum spacing remains a distribution requirement.

## Reproduce the actual result

The [historical plant matrix archive](https://digitalgarden.si/marketing/research/comparisons-20261020/plants-tests.zip) preserves the pinned 20 October package, owned fixture, harness and all 72 raw trial records. For new measurements, use the [v1 harness with updated local tooling](https://digitalgarden.si/benchmarks/data/threejs-instancing-v1-harness.zip). Keep the page visible, run the suite sequentially, and inspect the actual backend and canvas dimensions in each result.

For this garden pipeline, the actionable result is to share repeated geometry, retain wind attributes, and inspect real counters. Continue with [export and compression tests](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/), [the actual garden integration](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/), or [Claude Code and Codex on a smaller wind task](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/).

[^1]: Three.js, [WebGPURenderer](https://threejs.org/docs/pages/WebGPURenderer.html), checked 20 October 2026. Pinned source is included through the package lock.
[^2]: Three.js, [BatchedMesh](https://threejs.org/docs/pages/BatchedMesh.html). The backend-specific draw behavior above is an observed result of 0.186.1, not a promise made by the class name.
