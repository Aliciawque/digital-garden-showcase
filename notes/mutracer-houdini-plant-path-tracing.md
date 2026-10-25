# MuTracer with a Houdini Plant: Colors, Alpha, Instances and 64 Samples

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> MuTracer 0.1.5 tested with an owned Houdini plant: vertex colors, instancing, alpha masks, OIDN downloads, fixed-sample timing and a WebGL context failure.

- Author: Alicia Lu
- Published: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/

MuTracer 0.1.5 converted my small Houdini plant into a usable WebGPU path-traced image. Vertex colors survived, three instances retained distinct color tints, and two double-sided alpha-masked cards kept their cutout silhouettes. The WebGL attempt ended with a lost context and produced no completed image.

That makes this a useful still-image experiment for the tested asset. It does not establish that my animated production garden can switch renderers unchanged: the fixture has frozen geometry, and its custom TSL wind code was not converted.

![The owned Houdini plant rendered by MuTracer at 64 samples with OIDN small](https://digitalgarden.si/marketing/research/followup-20261024/mutracer-oidn.png)

Tested locally on 23 October 2026 with Codex assistance. [Download the reproducible lab](https://digitalgarden.si/marketing/research/followup-20261024/followup-lab.zip), [measured summary](https://digitalgarden.si/marketing/research/followup-20261024/summary.json) and [individual run receipts](https://digitalgarden.si/marketing/research/followup-20261024/trace-results.json).

## What I pinned and measured

The package is scoped: `@meimu/mutracer`, version 0.1.5. I checked the [official repository](https://github.com/pissang/mutracer) at commit `daa9765b5e65db92968c5e2a05bf04da2fde5098` and used its [Three.js conversion documentation](https://github.com/pissang/mutracer/blob/daa9765b5e65db92968c5e2a05bf04da2fde5098/docs/threejs.md). This is an MIT library with a changing 0.x API.

The test used Three.js 0.186.1, Vite 8.3.4, Chromium 155 on Windows, an NVIDIA GeForce RTX 5070 Ti and driver 617.42. Every path-traced image used 640 × 480 pixels, a fixed camera, 64 samples per pixel, six bounces and bevel disabled. Adaptive sampling was off. The independent MIT fixture has 6,056 triangles and no image textures; it is separate from the private production garden assets.

I built the lab before collecting the reported runs. Earlier development work had already warmed browser and driver shader caches, so these are **warm-system measurements**. The raster image in the lab is a framing and geometry reference with its own lighting implementation; I did not rank its frame rate against path-tracing convergence.

## Compatibility results

| Check | Observed result | Boundary |
| --- | --- | --- |
| Vertex color | Green foliage and purple flowers preserved | RGB colors; vertex alpha was not tested |
| InstancedMesh | Three tinted plants rendered from shared source geometry | Three instances; no large-instance stress test |
| Alpha mask | Two elliptical cutouts rendered without their rectangular corners | A 32 × 32 generated RGBA texture and alphaTest 0.5 |
| DoubleSide | Front- and back-facing cards both visible | One controlled card material |
| OIDN small | Completed a denoised PNG | One 64-sample image; no reference-image error metric |
| WebGL | Context lost; no completed PNG | One attempted run on this system |
| Custom TSL wind | Not converted or exercised | Geometry was frozen |

![Three instances retain separate red, neutral and blue tints](https://digitalgarden.si/marketing/research/followup-20261024/mutracer-instances.png)

![Front- and back-facing alpha-masked cards preserve their elliptical silhouettes](https://digitalgarden.si/marketing/research/followup-20261024/mutracer-alpha.png)

These checks confirm visible behavior for this fixture. They do not cover every OpenPBR feature, transmission model or Three.js material extension. The upstream conversion table documents additional supported and unsupported cases.

## Separate initialization, preparation and convergence

Three consecutive production-build plant runs completed without denoising:

| Run | Engine initialization ms | Prepare ms | BVH ms reported by library | Converge wall time ms |
| --- | --- | --- | --- | --- |
| 1 | 253.2 | 29.1 | 17.1 | 144.2 |
| 2 | 0.4 | 21.8 | 14.8 | 118.3 |
| 3 | 0.4 | 20.3 | 14.4 | 115.8 |

The median convergence call took 118.3 ms, with a 115.8–144.2 ms range. These wall-clock phases are not independent GPU kernel timings. In particular, library preparation diagnostics and shader compilation can overlap phases; adding every diagnostic value would double count work.

The earlier development pilot took about 10.3 seconds to converge, with about 10.1 seconds reported for the shader pipeline. I retained that pilot separately. It illustrates why a warm rerun cannot predict a new user's first shader compile. Neither three runs nor one GPU establishes a cross-device performance claim.

## What actually downloaded for denoising

The first measured production page downloaded a 2,034,133-byte JavaScript entry as 558,622 gzip body bytes, plus the 319,648-byte plant. The BVH worker body was 5,433 gzip bytes. This entry includes the complete lab, two Three.js renderer paths and MuTracer; it is not an isolated MuTracer bundle measurement.

Selecting OIDN small added a 30,975-byte gzip network module body and a **641,480-byte small-model weight file**. The build directory also contained a larger 1,841,160-byte weight file, but the small-model run did not download it. Installed or emitted bytes therefore differ from bytes requested by this control.

The OIDN run spent 35.4 ms setting the denoiser and 194.6 ms in the convergence call, including the selected denoising stage. Those are one-run observations. I did not subtract the median of different raw runs and label the difference as isolated denoiser GPU time.

## Reproduce and decide where it fits

Unzip the lab, run `npm ci --ignore-scripts`, `npm run build`, then `node serve-production.mjs`. Open the loopback address printed by the server and use the visible scene, backend and run controls. The lockfile, original PNGs, per-run JSON and fixture license are included.

For my garden, the next useful step would be a deliberate still-image path with a clear progress state and an export result. Animated TSL deformation, camera movement during convergence, memory limits and actual phone hardware require their own tests before such a feature belongs in production.

The related [GLB loading experiment](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/) measures delivery and first-frame phases. The [Houdini and Geometry Nodes export test](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/) checks whether two editable native projects preserve the same deliverable data. The [Three.js garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/) connects these smaller experiments to the existing renderer and format notes.
