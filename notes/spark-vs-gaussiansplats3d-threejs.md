# Spark vs GaussianSplats3D vs Native Three.js: An Integration Test

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Pinned Spark, GaussianSplats3D and native Three.js tests: the same synthetic splats, actual PLY/KSPLAT/SPZ loads, mesh occlusion and backend compatibility.

- Author: Alicia Lu
- Published: 2026-10-20
- Updated: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/

A Gaussian splat viewer must be tested with the renderer and file encoding it will actually use. In this integration test, I compared Spark 2.3.1, GaussianSplats3D 0.4.7 and native Three.js 0.186.1 on the same owned synthetic fixture. The measurements separate successful rendering paths from setup and format failures; they do not rank capture quality.

This integration test is grouped with the other renderer experiments in [the garden's Three.js rendering overview](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

Codex ran the conversions and browser checks on 20 October 2026. Splats remain a research fixture alongside [Alicia's interactive garden](https://digitalgarden.si/play/), not a production garden feature.

![The owned synthetic Gaussian fixture with opaque mesh references](https://digitalgarden.si/marketing/research/comparisons-20261020/splats-native.jpg)

## A known input rather than an unknown capture

The fixture contains 12,014 synthetic splats and known outliers. Higher spherical-harmonic coefficients are zero; the KSPLAT conversion explicitly used degree zero. The PLY is 2,981,073 bytes. GaussianSplats3D's public parser and compression-level-1 SplatBufferGenerator produced a 294,024-byte KSPLAT and retained 12,014 splats. The independently converted SPZ v4 fixture is 55,323 bytes.

These are different encodings and compression choices. Their byte counts are not an objective image-quality ranking, and this tiny generated flower does not represent a room scan, reflective object, noisy phone capture or full SH scene. No scanning, training or reconstruction quality is claimed.

All paths used one deduplicated Three.js 0.186.1 installation, the same camera, opaque Houdini plant, orange depth reference, canvas and DPR. Native paths used WebGPURenderer on WebGPU or forced WebGL2. Spark and GaussianSplats3D used classic WebGLRenderer. The accepted GaussianSplats3D DropInViewer path used CPU floating-point sorting and shared-memory workers; the local server supplied COOP/COEP headers.[^1]

## Record which combinations actually load

Each listed file used up to five unique loopback URLs; a failure stops that format's attempts and is retained. Successful load timings include readiness, compilation and a submitted frame. They are warm local observations rather than public CDN throughput or completed GPU timing.

| Rendering path | Format | Result | Median local load |
| --- | --- | --- | --- |
| native-webgpu | fixture.ply | loaded, 12,014 splats | 116.3 ms |
| native-webgpu | fixture-v4.spz | loaded, 12,014 splats | 66.6 ms |
| native-fallback | fixture.ply | loaded, 12,014 splats | 83.3 ms |
| native-fallback | fixture-v4.spz | loaded, 12,014 splats | 33.4 ms |
| spark-webgl | fixture.ply | loaded, 12,014 splats | 33.2 ms |
| spark-webgl | fixture-v4.spz | failed in pinned combination | not reported |
| gs3d-webgl | fixture.ply | loaded, 12,014 splats | 152.6 ms |
| gs3d-webgl | fixture.ksplat | loaded, 12,014 splats | 141.8 ms |

Spark's WebGPURenderer compatibility was checked separately and its failure is retained. That is an observed result of these versions and this renderer integration, not a statement that future Spark versions cannot support WebGPU. Spark's documented quick-start path and renderer design should be read for the tested version.[^2]

The first GaussianSplats3D GPU/integer-sort configuration decoded the expected count but never submitted a visible splat draw. The screenshot contained only the mesh references, and the triangle counter confirmed that absence. I rejected its empty-scene FPS, added a rendering-contribution guard, initialized the DropInViewer with the host renderer before loading, and checked a CPU floating-point-sort branch. Only that branch's visible rerun is included below. This is a compatibility finding for the tested integration, not a claim that the library's GPU sorting is always broken.

Native Three.js 0.186.1 does include SPZLoader. An older assertion that native SPZ loading was absent was wrong and was corrected before this comparison. The SPZ path uses that loader explicitly. This converted Y-up fixture needed an X-axis half-turn to align with its PLY reference; that correction is specific to this file, not a universal SPZ rule.

File-format support alone does not ensure that every encoding works: SPZ versions, compression headers and library versions matter. A failed v4 file load is not turned into a successful PLY result under the same label.

Update, 21 October: a different, million-point SPZ v4 export from free PINOC loaded successfully with Spark 2.4.0 and native Three.js in [the JST-2 garden integration test](https://digitalgarden.si/blog/viggle-jst2-spz-threejs-garden/). Both the input and Spark version changed, so that result does not establish which change explains the earlier failure. Spark with WebGPURenderer still failed in the newer test. A later [Spirula Studio and Spark](https://digitalgarden.si/blog/spirula-studio-garden-splat-threejs/) test also used Spark 2.4.0, this time on a Vulkan-trained official Garden PLY and the SPZ converted from it. That sample is not this synthetic fixture.

## Orbit observations after compatibility

The browser reported Chromium 155 on Windows and an RTX 5070 Ti. Each successful path ran one discarded 20-second warm-up and three measured 20-second half-orbits at 1280 × 720, DPR 1, without antialiasing. This bounded integration protocol replaces the broader unexecuted draft benchmark plan; no three-warm-up/five-run performance claim is made here. Trials ran sequentially after the plant matrix, with visibility guards.

| Successful path | Median mean FPS | Median 1% low FPS | Median CPU render time |
| --- | --- | --- | --- |
| native-webgpu | 59.99 | 59.74 | 0.215 ms |
| native-fallback | 59.99 | 59.74 | 0.155 ms |
| spark-webgl | 59.99 | 59.72 | 0.135 ms |
| gs3d-webgl | 59.99 | 59.77 | 0.080 ms |

The scene is small and display capped. These figures cannot establish a universal fastest renderer. Gaussian sorting may involve asynchronous worker work outside the measured renderer.render() call; CPU submission time is not total application CPU cost. JavaScript heap is not total VRAM, and there is no controlled memory ranking.

## Test depth and transparency separately

The viewer supplies three fixed inspection states: an opaque plant/bar in front, the splats in front, and a translucent petal. Screenshots retain the same references and view. Opaque depth occlusion and mixed transparent compositing are different acceptance questions; a plausible opaque screenshot does not prove globally correct sorting with arbitrary transparent objects. The flower appeared darker under Spark and GaussianSplats3D's tested defaults than under the native path. I did not normalize that into a pixel-quality score or claim colour-management parity.

The fast half-orbit helps expose transient sorting changes, but the saved screenshots are representative frames, not proof that every intermediate ordering is perfect. This test does not certify a general transparency solution or high-motion capture scene. The small fixture and its simple degree-zero colours limit what can be concluded.

## Carry a verified integration into the project

The [splat comparison download](https://digitalgarden.si/marketing/research/comparisons-20261020/splats-tests.zip) contains the pinned package, known-count fixtures, KSPLAT converter, independent viewer and detailed success/failure receipts. Bundle-export gzip sizes are recorded separately from asset sizes and actual network transfer. Dependencies are installed from their official packages and retain their licenses; no external capture sample is redistributed.

For this garden's current node-renderer stack, native loading is worth testing first because it shares that renderer. Classic-WebGL libraries remain separate viable integration paths where their versions and formats pass. The correct choice follows the required format, renderer, sorting behavior and scene budget. Continue with [the earlier Gaussian asset workflow](https://digitalgarden.si/blog/gaussian-splats-threejs/) or [the plant submission measurements](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/).

[^1]: GaussianSplats3D, [official repository and DropInViewer documentation](https://github.com/mkkellogg/GaussianSplats3D), checked 20 October 2026. The tested package was 0.4.7.
[^2]: Spark, [official documentation](https://sparkjs.dev/docs/), checked 20 October 2026. The tested package was 2.3.1. Native SPZ behavior was checked against the installed Three.js 0.186.1 source and actual v4 input.
