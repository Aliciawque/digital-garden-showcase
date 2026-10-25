# Gaussian Splats in Three.js: Native Renderer and Spark Tested

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Test native Three.js Gaussian splats and Spark 2.3.1 with owned fixtures: WebGPU compatibility, SPZ v4 loading, GSOPs cropping and transparency limits.

- Author: Alicia Lu
- Published: 2026-10-19
- Updated: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/gaussian-splats-threejs/

Native Three.js Gaussian splats worked on both WebGPU and WebGL2 in this test, including direct SPZ v4 loading with SPZLoader. Spark 2.3.1 worked with WebGLRenderer, but the tested integration failed with WebGPURenderer. The same real SPZ v4 file failed Spark’s reader with an “Invalid gzip header” error.

For how this splat study relates to the mesh plant notes, see [the garden's Three.js rendering overview](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

**Correction, 20 October 2026:** the original article incorrectly stated that the pinned Three.js 0.186.1 package had no SPZLoader. It does include that loader. I have now directly loaded and rendered the original SPZ v4 fixture on native WebGPU and WebGL2. The download viewer and receipts below include this correction; the original 19 October PLY measurements retain their original scope and date.

These findings come from an **owned synthetic fixture**, tested on 19 October 2026. It contains 12,014 Gaussian points derived from the procedural plant in the previous article, including 500 intentionally placed outliers. It is not a trained capture or a photogrammetry quality benchmark. Its higher spherical harmonic coefficients are all zero, so the results do not validate view dependent appearance.

![Native Three.js Gaussian splats showing the synthetic flower, outliers and opaque mesh references](https://digitalgarden.si/marketing/research/webgpu-series-20261019/splat-native.png)

## Pin the renderer and the data

The native path used Three.js 0.186.1 with its GaussianSplat mesh and PLY loader under WebGPURenderer. The fallback used the same renderer with the WebGL backend forced. The Spark path used Spark 2.3.1 with WebGLRenderer, matching its documented setup.[^1] All tests ran in a visible Chromium 155 in app browser on an NVIDIA RTX 5070 Ti, at 1280 × 720 and DPR 1.

The PLY generator emitted position, color, opacity, scale, rotation and the expected spherical harmonic fields. The underlying plant produced 11,514 jittered points; the separate 500 red outliers made crop failures easy to detect. Encoding a repeated synthetic shape is much easier than compressing a complex real capture, so this dataset must not be used to advertise a general compression ratio.

## What loaded, and what failed

| Path | Input | Result |
| --- | --- | --- |
| Native WebGPURenderer, WebGPU | PLY, 12,014 points | Rendered |
| Native WebGPURenderer, WebGL2 | Same PLY | Rendered |
| Spark + WebGLRenderer | Same PLY | Rendered |
| Spark + WebGPURenderer | Same PLY | TypeError at renderer context extension access |
| Spark + WebGLRenderer | Actual SPZ v4 | Invalid gzip header |
| Native WebGPU | SPZ v4 decoded to PLY | Rendered, 12,014 points |
| Native WebGPU, checked 20 October | Direct SPZ v4 via SPZLoader | Rendered, 12,014 points |
| Native WebGL2, checked 20 October | Same direct SPZ v4 | Rendered, 12,014 points |
| Native WebGPU | 32 byte per point SPLAT | Rendered, 12,014 points |
| Native WebGPU | GSOPs cropped PLY | Rendered, 11,514 points |

The Spark WebGPU failure was `this.renderer.getContext(...).getExtension is not a function`. It is evidence about this integration and version, not proof that every future Spark path or adapter must fail. The working WebGLRenderer route remains useful.

Three.js 0.186.1 includes SPZLoader alongside the PLY, SPLAT and KSPLAT loaders. For v4, parsing can be asynchronous; `loadAsync` handles that path.[^4] The earlier SPZ-to-PLY conversion worked, but it is not required to use this file with the native renderer.

```js
import { SPZLoader } from 'three/addons/loaders/SPZLoader.js';
import { GaussianSplat } from 'three/addons/objects/GaussianSplat.js';

const geometry = await new SPZLoader().loadAsync('fixture-v4.spz');
const splat = new GaussianSplat(geometry);
// Align this converted fixture with its original Y-up PLY scene.
splat.rotation.x = Math.PI;
scene.add(splat);
```

The Rx180 alignment above is specific to this file’s conversion and our reference scene, not a universal SPZ requirement. After that transform, corresponding decoded centers differed from the original PLY by at most 0.0001221 scene units per component. Both visible browser backends rendered 12,014 points at a 1280 × 720 canvas and renderer pixel ratio 1. Five local load checks per backend passed; this follow-up did not rerun the FPS benchmark or validate nonzero SH appearance.

![Direct SPZ v4 loaded by the pinned native Three.js SPZLoader on WebGPU, aligned with the original fixture and opaque mesh references](https://digitalgarden.si/marketing/research/webgpu-series-20261019/native-spz-v4.png)

## SPZ v4 is not the old gzip container

I uploaded the owned PLY into Niantic’s official browser utility, downloaded SPZ, and inspected the result. It had an `NGSP` header, version 4, 12,014 points and 55,323 bytes. Reopening it in the utility showed six streams. The utility then decoded it back to PLY for the native renderer test. No external scan was uploaded or republished.

The official format documentation describes v4’s plain header and Zstandard streams, while earlier versions used gzip.[^2] That explains why a reader expecting the older container can reject a valid v4 file before rendering starts. It does not mean the SPZ download was corrupt. The exact utility build commit was not exposed in its UI; the record keeps the test date, downloaded file hash and inspected header instead of inventing one.

The unusually small SPZ is a property of this simple synthetic dataset with zero higher SH coefficients. Real captures need separate file size, decoder time and appearance measurements.

## Crop in Houdini, then reopen

I used the public GSOPs 3.0.0 HDAs, pinned to [commit 46f1756c65d4](https://github.com/cgnomads/GSOPs/tree/46f1756c65d40315fa977af78dfc7af860117894), in an isolated Houdini 22.0.429 subprocess. Loading only the apparent import/crop/export HDAs initially left missing dependencies. Loading the public dependency set resolved that problem without changing the user’s global Houdini package configuration.

The crop box removed the 500 known outliers. The exported PLY was then imported again: **12,014 input points → 11,514 cropped points → 11,514 reopened points**, with no node errors. Some no matching group/attribute export warnings remained and are recorded. This proves the known count crop and round trip, not arbitrary scan cleanup or attribute equivalence for nonzero SH data.[^3]

![The cropped synthetic plant in the native renderer after the known outliers were removed](https://digitalgarden.si/marketing/research/webgpu-series-20261019/splat-cropped.png)

GSOPs is a third party tool with its own license. The download links its official source and pinned version; it does not redistribute the plugin. An external sample marked noncommercial was considered and rejected for this publication. All published splat imagery instead comes from the owned fixture.

## Performance and compositing boundaries

Each primary renderer ran a 10 second static view after one second of warmup. All three working paths measured about 59.99 FPS, with 95th percentile frame intervals of 16.7–16.8 ms. Native totals were four render calls and Spark totals were three. At only 12,014 points and a 60 Hz display ceiling, this is a compatibility check, not a large scene throughput ranking.

Five warm local load trials had medians of 116.7 ms for native WebGPU PLY, 76.9 ms for native WebGL2 PLY and 16.7 ms for Spark WebGL PLY. Each includes fetch, parsing, initialization and the first submitted frame. Their pipelines and decoder paths differ; these values do not isolate decoder speed or predict CDN loading.

I also placed opaque red and green mesh references before and behind the splats, then introduced a translucent pink plane. The working paths rendered the opaque controls, but the translucent views differed in brightness and opacity. They did not achieve pixel parity. A scene with layered petals, glass or overlapping translucent materials needs a dedicated sorting and art direction acceptance pass. “It loads” is a much narrower claim than “it composites correctly for every asset.”

## Use the test as a compatibility receipt

Codex executed the native scripts, official utility workflow and browser checks. The supplied receipts distinguish a successful render, a known setup error and a load only check.

The [experiment download](https://digitalgarden.si/marketing/research/webgpu-series-20261019/webgpu-garden-tests.zip) includes owned Gaussian fixtures, generation code, compact receipts and a standalone viewer, updated 20 October. It includes the actual SPZ v4 file, the corrected native loader path and the recorded Spark container failure. The [machine-readable results](https://digitalgarden.si/marketing/research/webgpu-series-20261019/results-2026-10-19.json) preserve the original measurements and add a separately dated correction. No training run, real capture reconstruction, large splat scene or physical phone was tested. A later [Spark and GaussianSplats3D comparison](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/) loads this same fixture through native Three.js, Spark and GaussianSplats3D, including KSPLAT and direct SPZ v4, and keeps the combinations that failed to draw.

For mesh plants, start with [Houdini procedural geometry and wind](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/) and [glTF compression](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/). The [main garden article](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/) explains where those meshes fit. The current production garden remains a mesh based scene; this splat study does not silently replace it.

For a trained sample rather than this synthetic flower, [Spirula Studio to Three.js](https://digitalgarden.si/blog/spirula-studio-garden-splat-threejs/) uses the official Mip-NeRF 360 Garden images with supplied camera poses, Vulkan training, a PLY checkpoint and Spark. It documents 100,000 splats and is not a phone capture. For an actual generated environment rather than this synthetic flower, the [free PINOC / JST-2 SPZ test](https://digitalgarden.si/blog/viggle-jst2-spz-threejs-garden/) uses a million-point garden export. It records native and Spark 2.4.0 loading, opaque plant occlusion and the WebGPURenderer combination that failed. Different input files and package versions are kept separate.

[^1]: [Spark documentation](https://sparkjs.dev/docs/), including its WebGLRenderer quick start, checked 19 October 2026.
[^2]: Niantic, [SPZ format and official utility](https://github.com/nianticlabs/spz).
[^3]: [GSOPs official repository](https://github.com/cgnomads/GSOPs), version 3.0.0 public HDAs at the pinned commit above.
[^4]: Three.js, [SPZLoader documentation](https://threejs.org/docs/pages/SPZLoader.html), checked against the installed 0.186.1 source and the original fixture on 20 October 2026.
