# Viggle JST-2 SPZ in Three.js: A Free PINOC Garden Test

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A free PINOC garden export tested in Three.js 0.186.1 and Spark 2.4.0: SPZ v4 loading, opaque mesh occlusion, local timings and a WebGPU incompatibility.

- Author: Alicia Lu
- Published: 2026-10-21
- Updated: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/viggle-jst2-spz-threejs-garden/

A garden exported from Viggle's free PINOC interface loaded as SPZ v4 in native Three.js 0.186.1 on WebGPU and its WebGL2 fallback, and in Spark 2.4.0 with classic WebGLRenderer. Spark did not initialize with WebGPURenderer in this test. The exported scene contains 1,048,576 Gaussian splats and occupies 17,575,822 bytes. These are measurements from one generated scene, not a claim about every JST-2 output.

Codex operated the browser and ran the local tests for Alicia on 21 October 2026. This is a research fixture beside [the interactive garden](https://digitalgarden.si/play/), not a new feature in the production garden. No paid API, subscription or purchased credits were used.

The earlier synthetic splat tests and the mesh plant notes are collected in [the Three.js 3D garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

![The PINOC garden and an opaque Houdini plant in the native Three.js viewer](https://digitalgarden.si/marketing/research/pinoc-20261021/native-front.png)

## What the free PINOC session actually produced

Viggle presents JST-2 as a world model, and PINOC is its browser interface.[^1] I used this prompt:

> Make an environment: a compact peaceful botanical garden at eye level, with a curving narrow pale gravel path through dense blue grape hyacinths, pink tulips, green leaves and soft grass. Low flower beds in the foreground, a small weathered stone border, distant leafy trees. Soft overcast daylight, natural botanical colors, realistic spatial depth, no people, no text or logos. Keep an open patch beside the path for adding a separate 3D plant mesh.

The free balance went from 60 to 43 credits: five for the initial generation and three each for four automatic filling passes. The agent kept trying to repair gaps, so I stopped that sequence after the fourth pass. This is the observed cost of this session, including those passes; it is not a quote for all future accounts or a one-pass generation cost. The changing interface quality scores did not establish that every hole had been removed.

The downloaded file was a `.spz`, not a mesh with editable faces. Its header identified SPZ version 4, 1,048,576 points, spherical-harmonic degree zero and 12 fractional bits. Its SHA-256 is `c0dfc6a38528e1324cb891b670a67d3e9dd7b1e7249e9375759e95a965a0d421`.

I did not measure the vendor's model inference time. The interactive session includes prompting, queueing, automatic repairs, viewing and export; it cannot validate a one-second model-generation claim. No PLY or character `.vsplat` result is included in this experiment. A later [Spirula Studio PLY to SPZ](https://digitalgarden.si/blog/spirula-studio-garden-splat-threejs/) test started from the official Mip-NeRF 360 Garden sample: Spirula Studio trained it on Vulkan, wrote a PLY, and Spark converted that PLY. It does not replace this generated PINOC file.

## Three working renderer paths and one recorded failure

The independent viewer combines that export with one owned Houdini grape-hyacinth mesh, containing 6,056 triangles. The mesh uses an opaque magenta inspection material, so it is easy to distinguish from generated flowers. Both libraries resolve to the same Three.js installation.

| Path | Renderer | Actual result |
| --- | --- | --- |
| Native WebGPU | WebGPURenderer, actual WebGPU backend | Loaded and rendered 1,048,576 splats |
| Native fallback | WebGPURenderer with forceWebGL | Loaded and rendered through WebGL2 |
| Spark WebGL | Spark 2.4.0 with WebGLRenderer | Loaded and rendered after sorting was ready |
| Spark WebGPU | Spark 2.4.0 with WebGPURenderer | Initialization failed |

The native branch uses `SPZLoader` and `GaussianSplat` from the installed Three.js add-ons.[^2] For this PINOC export, zero rotation aligned the scene with the camera; the half-turn used for my earlier synthetic fixture was unnecessary. Coordinate corrections must follow the actual file, not the extension alone.

```js
import { WebGPURenderer } from 'three/webgpu';
import { SPZLoader } from 'three/addons/loaders/SPZLoader.js';
import { GaussianSplat } from 'three/addons/objects/GaussianSplat.js';

const renderer = new WebGPURenderer({ forceWebGL: false });
await renderer.init();
const geometry = await new SPZLoader().loadAsync('/scene.spz');
scene.add(new GaussianSplat(geometry));
await renderer.compileAsync(scene, camera);
renderer.render(scene, camera);
```

For Spark, `await splat.initialized` alone was insufficient to prove that the first submitted frame contained splats. An initial frame reported only the mesh's 6,056 triangles. I corrected readiness to await `spark.update({ scene, camera })` after attaching the splat and updating scene matrices, then confirmed 2,103,208 submitted triangles. That guard prevents an empty-scene frame from looking like a fast successful load.[^3]

```js
const spark = new SparkRenderer({ renderer }); // classic WebGLRenderer
scene.add(spark);
const splat = new SplatMesh({ url: '/scene.spz', lod: false });
await splat.initialized;
scene.add(splat);
scene.updateMatrixWorld(true);
await spark.update({ scene, camera });
renderer.render(scene, camera);
```

With WebGPURenderer, Spark's constructor instead threw `TypeError: this.renderer.getContext(...).getExtension is not a function`. This is a version-specific integration failure, not a prediction about future Spark support. The earlier [Spark, GaussianSplats3D and native comparison](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/) used Spark 2.3.1 and a different synthetic SPZ; its format failure does not override this successful 2.4.0 PINOC-file test.

## Local timings, with the display ceiling kept visible

The desktop browser reported Chromium 155 on Windows and an NVIDIA RTX 5070 Ti. The canvas was 1280 × 720 at renderer DPR 1, with antialiasing disabled. Each working path used five unique loopback URLs for load measurements, one discarded five-second warm-up and three visible 30-second camera-path trials. The renderer stayed initialized between loads. These are warm local asset loads, not cold-device startup or public CDN download times.

| Path | Median local load | Median mean FPS | Median 1% low FPS | Median CPU render call |
| --- | --- | --- | --- | --- |
| Native WebGPU | 400.10 ms | 59.99 | 59.50 | 0.215 ms |
| Native WebGL2 fallback | 266.53 ms | 59.99 | 59.60 | 0.155 ms |
| Spark WebGL | 299.96 ms | 59.99 | 59.50 | 0.130 ms |

Load time includes decoding/readiness, compilation where available, sorting readiness for Spark, and submitted frames. It does not wait for a GPU-completion timestamp. CPU render-call duration excludes asynchronous worker work and is not total application CPU cost. The 1% low is the reciprocal of the mean of the slowest one percent of frame intervals; table values are medians across the three trials.

All three paths reached this setup's roughly 60 Hz presentation ceiling. That establishes smooth operation in this bounded test, not equal maximum throughput or a universal fastest renderer. The workstation was not a dedicated thermal or power-controlled laboratory. No physical iPhone, Android phone, mobile memory limit or mobile FPS was measured.

## Check mesh occlusion before calling it a mixed scene

Three fixed states put the opaque plant in front, behind and through the splat scene. In the inspected frames, the foreground plant remained visible, the rear plant was hidden, and the enlarged intersecting plant showed partial occlusion. This is a useful opaque-depth check; it does not prove globally correct ordering for translucent petals or every view in a large scene.

![An enlarged opaque plant partly occluded by the generated scene in native Three.js](https://digitalgarden.si/marketing/research/pinoc-20261021/native-straddle.png)

Spark and native Three.js produced visibly different brightness under their tested defaults. I did not normalize their colour pipelines or score one as higher quality. These representative views do not certify complete geometry from every camera position. It is a captured-looking spatial backdrop, not a replacement for individually authored, deformable flower meshes.

## Reproduce the integration without redistributing the generated world

The [test package](https://digitalgarden.si/marketing/research/pinoc-20261021/pinoc-integration-tests.zip) contains the pinned viewer, owned mesh fixture, prompt, header metadata and all accepted load/frame receipts, including the WebGPU failure. Supply your own exported file as `public/pinoc-garden.spz`; the vendor-generated world is not included. Screenshots document the tested result. Dependencies and the owned fixture have their own notices in the archive.

For this garden, I would test a generated splat as a bounded background around a small authored interaction area. Editable plants, wind, per-flower interaction and reusable geometry remain mesh work. Native loading fits the current node-renderer stack; Spark is a separate classic-WebGL integration when the required format and renderer pass together. Continue with [the Gaussian asset workflow](https://digitalgarden.si/blog/gaussian-splats-threejs/) or [the Houdini-to-Three.js garden pipeline](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/).

[^1]: Viggle, [JST-2 and PINOC](https://viggle.ai/jst2), checked 21 October 2026. Product claims are distinguished from this browser-session measurement.
[^2]: Three.js, [SPZLoader documentation](https://threejs.org/docs/pages/SPZLoader.html); installed and tested Three.js 0.186.1.
[^3]: Spark, [SparkRenderer documentation](https://sparkjs.dev/docs/spark-renderer/), checked 21 October 2026; tested package 2.4.0 and awaited scene/camera sorting update.
