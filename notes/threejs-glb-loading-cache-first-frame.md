# Three.js GLB Loading: Cache, Decode and the First GPU Frame

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Twenty controlled Three.js WebGPU loading trials compare raw and meshopt GLBs, separate cache, decoding and compilation, and retain the first-run outlier.

- Author: Alicia Lu
- Published: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/

The same plant shrank from 319,648 to 130,296 bytes with meshopt, a 59.2% reduction, while its decoded vertex attributes stayed identical. Across 20 local WebGPU trials, the smaller file reduced the fresh-URL first-frame median from 35.7 to 17.6 ms. Cached trials were much closer, and the first raw trial contained a large compilation cost.

These measurements answer a narrow loading question. They do not measure Internet delivery, a new device's startup or a phone's thermal behavior. A scene's steady frame rate and its first useful frame need separate budgets.

![All twenty first-frame samples, with ranges and medians, grouped by GLB variant and HTTP cache state](https://digitalgarden.si/marketing/research/followup-20261024/loading-chart.png)

Tested with Codex assistance on 23 October 2026. [Download all 20 rows as CSV](https://digitalgarden.si/marketing/research/followup-20261024/loading.csv), [full timing JSON](https://digitalgarden.si/marketing/research/followup-20261024/loading-results.json), and the [reproduction package](https://digitalgarden.si/marketing/research/followup-20261024/followup-lab.zip).

## Keep the content fixed

Both files came from the same independent MIT plant, with 6,056 triangles, vertex colors, normals, UVs and the custom wind fields `_STIFF` and `_PHASE`. glTF Transform 4.3.0 and meshoptimizer 1.3.0 encoded the second file with `EXT_meshopt_compression`. No simplification or attribute-quantization transform was applied. The script decoded both files and compared attribute hashes before any timing began.

| Asset | File body bytes | Triangles | Decoded attribute comparison |
| --- | --- | --- | --- |
| Raw GLB | 319,648 | 6,056 | Baseline |
| Meshopt GLB | 130,296 | 6,056 | Identical hashes |

The fixture has no image textures, so KTX2 compression would add no relevant texture experiment here. A textured SimReady or room scene needs a separate protocol that includes transcode, upload and texture memory.

The [asset manifest](https://digitalgarden.si/marketing/research/followup-20261024/asset-manifest.json) records the full-file SHA-256 hashes and per-attribute checks. The [validation report](https://digitalgarden.si/marketing/research/followup-20261024/asset-validation.json) records zero errors and zero warnings for both files.

## Four conditions, five repetitions each

I used Three.js 0.186.1 in a production Vite build, Chromium 155, Windows and an RTX 5070 Ti with driver 617.42. A single WebGPU renderer was initialized before the loop. Camera, lighting, resolution of 640 × 480 and geometry remained fixed.

Each of five repetitions used a new query-string cache key for both files, then immediately requested that same URL again. Raw and meshopt order alternated between repetitions. The local server sent the GLB body without HTTP compression and allowed caching. Resource Timing reported the complete file body for every fresh URL and zero transferred bytes for every immediate cached repeat.

“Fresh URL” describes **HTTP cache state only**. The JavaScript modules, device, decoder and driver caches had already been used. I did not clear the browser profile or reboot the GPU. This distinction matters more than giving the trial an ambiguous “cold start” label.

For each row I measured:

1. Fetch and read the response body.
2. Parse the GLB and decode geometry.
3. Await `compileAsync` for the fixed scene.
4. Render and await WebGPU queue completion.
5. Record two subsequent animation frames separately.

Queue completion gives a consistent API boundary, but it is not a measurement of pixels reaching the display. Canvas screenshot export was unreliable in this harness and is excluded as visual first-frame proof. The plot comes directly from the timing rows; decoded data equality is the asset-equivalence check.

## Results and the first compilation

All times below are milliseconds. Each first-frame group contains five observations; ranges include every observation.

| Condition | Fetch median | Parse/decode median | Compile median | GPU-completed first-frame median | First-frame range |
| --- | --- | --- | --- | --- | --- |
| Raw, fresh URL | 24.9 | 0.5 | 6.9 | 35.7 | 14.7–144.9 |
| Raw, cached URL | 7.1 | 0.4 | 13.5 | 24.9 | 21.4–26.0 |
| Meshopt, fresh URL | 3.6 | 0.6 | 6.8 | 17.6 | 16.3–21.9 |
| Meshopt, cached URL | 7.3 | 0.6 | 9.8 | 21.7 | 19.7–27.1 |

The first raw trial took 144.9 ms to the GPU-completed frame, including 91.2 ms in compilation. I kept that row. A five-sample median is resistant to one unusually large value, but removing it entirely would hide a real part of this run's startup behavior.

The stage medians are computed independently and do not add up to the median total. Cached fetches also happened to take longer than fresh meshopt fetches on this local server. With such small timings, browser scheduling and loopback overhead matter; this is not evidence that downloading from a network is intrinsically faster than cache access.

After loading, I recorded ten render-and-queue-completion calls per row. Group medians fell between 3.2 and 3.5 ms. Those calls include CPU submission and waiting for the GPU and are paced by animation frames; inverting them into an advertised FPS number would be misleading. Meshopt changes storage and decoding while the decoded geometry remains the same.

## How I would use the result

The byte reduction is directly useful for this fixture. The observed first-frame reduction is specific to this computer, localhost server and protocol. A deployment test should add the actual CDN, bandwidth and latency, test fresh and repeat visits, and retain shader compilation and texture work as separate phases.

The existing [240-sample instancing benchmark](https://digitalgarden.si/benchmarks/threejs-webgpu-vs-webgl-instancing/) remains its own immutable v1 dataset. These 20 loading rows do not replace it. Real phone testing is still pending; desktop emulation cannot fill that gap.

To reproduce this run, use the included lockfile, run the production server and select “Run 5 paired loading trials.” `summarize.mjs` recomputes the table from the saved rows and `plot-loading.mjs` produces the vector plot. Earlier captures and the failed screenshot attempt remain separately identified in the package.

For related boundaries, see the [MuTracer fixed-sample experiment](https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/), [SimReady USD-to-GLB export checks](https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/), and the [Three.js rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).
