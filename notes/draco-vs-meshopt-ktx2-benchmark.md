# Draco vs Meshopt and KTX2: 132 Loading Trials in Three.js

> I measured 132 Three.js loads across Draco, Meshopt and KTX2: file sizes, parse time, first GPU frame, cache evidence and reproducible assets.

- Author: Alicia Lu
- Published: 2026-10-26
- Canonical HTML: https://digitalgarden.si/blog/draco-vs-meshopt-ktx2-benchmark/

Meshopt produced the smaller geometry files in two fixtures, while Draco was slightly smaller in the texture-heavy leaf. ETC1S gave that leaf the smallest complete GLB and the quickest measured fresh-URL first frame. UASTC reduced file size too, but its longer transcode made it slower to load here. Those outcomes came from three different fixtures, not one compression preset applied to a production garden.

Tested 26 October 2026. Hardware: Ryzen 9 9950X, 32 logical processors, 66,196,152,320 bytes installed RAM, RTX 5070 Ti with 16,303 MiB reported device memory, NVIDIA driver 617.42, Windows 11 Pro. Existing applications were preserved; this was an interactive workstation rather than a dedicated lab.

## What was tested

The experiment pins Three.js 0.186.1, glTF Transform 4.5.1, draco3dgltf 1.5.9, meshoptimizer 1.3.0 and gltfpack 1.3. It uses an independently licensed procedural grape hyacinth, a self-authored parametric surface with a morph animation, and a curved leaf with a self-authored 2048×2048 alpha texture. No private production assets are included.

Each geometry fixture has raw, Draco and Meshopt variants. The leaf adds ETC1S and UASTC KTX2 variants. The KTX2 files replace only the texture: transformed geometry from the texture encoder is discarded. Geometry compression uses 14-bit position, 10-bit normal, 12-bit UV and 12-bit generic-attribute settings, with the other defaults recorded in the script. Matching nominal bit settings does not make the codecs or preprocessing identical. No mesh simplification is applied.

The plant input contained near-zero-area faces. A first export changed its triangle count, so the original was retained and twenty faces at area <= 1e-12 were removed from the common baseline. Every reported plant variant starts from that same cleaned file and retains 6,036 triangles. This avoids counting cleanup as a compression benefit.

## File size depends on what dominates the asset

| Fixture | Raw bytes | Draco bytes | Meshopt bytes |
| --- | --- | --- | --- |
| botanical | 319,408 | 70,932 | 59,908 |
| morph | 874,896 | 303,896 | 93,368 |
| foliage | 3,198,060 | 2,902,220 | 2,922,204 |

![Complete GLB file sizes](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/compression-bytes.png)

The plant's Meshopt file is 59,908 bytes, versus 70,932 for Draco and 319,408 raw. The morph surface gives Meshopt an even larger size advantage. The leaf behaves differently: its PNG dominates, so compressing the geometry alone saves comparatively little. Draco's leaf is 2,902,220 bytes and Meshopt's is 2,922,204.

Changing that leaf texture to ETC1S yields 892,452 bytes; UASTC yields 2,007,384. These are texture-format results, not evidence that either geometry codec always wins.

The CSV also includes local gzip and Brotli arithmetic. For example, the raw leaf's 3,198,060-byte GLB becomes 2,885,907 bytes with local Brotli, versus 623,144 for ETC1S and 1,741,668 for UASTC. The browser test serves identity encoding over loopback. These calculations do not describe a measured CDN response.

## Loading protocol and first-frame results

Both runs use WebGPURenderer, once with its actual WebGPU backend and once with forceWebGL selecting WebGL2. The render buffer is fixed at 960×540, DPR 1. Runtime metadata identifies NVIDIA Blackwell for WebGPU; the WebGL renderer string identifies RTX 5070 Ti through ANGLE/D3D11.

Each backend performs one excluded decoder/shader warmup per variant, then three paired fresh-URL and immediately repeated warm-URL trials. Variant order reverses on alternate repeats. Eleven files × two cache conditions × three repeats × two backends gives 132 reported loading trials. The initialized renderer, decoder workers, driver and operating-system caches remain warm. A fresh URL is not a cold browser launch.

The stages are fetch, GLTFLoader parse/decode, scene setup plus compileAsync, and first render plus GPU completion wait. The compile column includes pose, material, camera/bounds and texture bookkeeping; it is not isolated shader compilation time. Parse includes image decoding, worker scheduling, glTF processing and KTX2 transcoding; it is not isolated codec execution time. A GPU-complete frame is also distinct from a physically presented display frame.

| 2K leaf variant | WebGPU parse ms | WebGPU fetch → GPU ms | WebGL2 fetch → GPU ms |
| --- | --- | --- | --- |
| raw | 24.2 (24.1–27.2) | 76.9 (60.0–102.3) | 100.4 (100.3–106.9) |
| draco | 24.2 (23.6–24.7) | 79.5 (55.7–95.9) | 100.4 (67.0–111.4) |
| meshopt | 27.7 (24.8–29.7) | 75.9 (70.2–96.7) | 83.7 (83.5–109.2) |
| etc1s | 22.2 (22.1–22.2) | 54.9 (43.9–55.3) | 67.0 (50.3–76.6) |
| uastc | 57.6 (55.9–58.0) | 116.1 (103.6–123.2) | 100.3 (100.2–126.1) |

Values are median (minimum–maximum), three fresh-URL trials per cell. The full CSV separates warm URL results. Resource timing reports zero transferred bytes for the repeated URL cache hits; the server ledger and exact trial URLs are retained for checking that behavior.

![Fresh-URL first-frame medians and ranges](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/compression-first-frame.png)

On this fixture, ETC1S reduces work on the loading path enough to help first-frame latency. UASTC's parse/transcode stage is about 57 ms, so its smaller file does not automatically mean a quicker first frame. The geometry-only raw plant also parses quickly: a codec can add decode work even while reducing transferred bytes. This local comparison does not predict the outcome on a slow network or phone.

## File bytes and GPU texture bytes are different

Both KTX2 variants transcode to the same BC7 target here, with twelve mip levels and 5,592,432 bytes of compressed mip payload. An RGBA8 texture at the same dimensions with a full mip chain is approximately 22.37 MB. ETC1S and UASTC differ substantially on disk but occupy the same compressed texture payload in this browser/GPU combination.

Those figures are texture payload accounting. They exclude render targets, buffers, allocation padding, browser copies, drivers and other applications, and are not total VRAM measurements. Another device may select a different target format. The loader's device-dependent format selection is described in the [official KTX2Loader documentation](https://threejs.org/docs/pages/KTX2Loader.html).

## Did the assets survive?

All eleven GLBs have zero validator errors. The KTX2 files retain two validator warnings each because this validator does not fully recognize the extension/image format; the warnings are included. Actual parsing, transcoding and rendering were checked on both backends.

Triangle counts, attribute names, animation counts and morph-target counts match each fixture's baseline. Nearest decoded world-position error stays below 0.000132 scene units across the geometry variants. Independent nearest-position checks retain the custom attributes within a 0.001 generic-value tolerance; the largest recorded custom-value error is approximately 0.000122 for Draco's plant stiffness. Animation sampler values and interpolation agree. The morph fixture is rendered at 0.5 seconds, providing an additional pose check.

Fixed-view images of the alpha leaf give full-image RGB PSNR of about 42.86 dB for ETC1S and 48.45 dB for UASTC against raw in WebGPU. The background is included, so these are reproducible image differences rather than universal perceptual quality scores. Fine alpha edges and other viewpoints still deserve inspection. Custom-attribute preservation is not live wind or skeletal-animation acceptance.

## Reproduce and choose within this scope

The accompanying README, package lock, input fixture, prepared assets, encoder hashes, scripts, screenshots and raw JSON/CSV are in this experiment folder. Run npm ci, npm run build and npm run serve, then use the visible controls at the local address. The optional asset regeneration step requires the official gltfpack binary; the browser tests can use the included GLBs. [glTF Transform's official command reference](https://gltf-transform.dev/cli) explains its available geometry and texture operations.

These results favor Meshopt for the two geometry-focused fixtures and ETC1S when this leaf's transfer and texture footprint matter more than the smaller observed image error of UASTC. They do not establish a universal codec ranking. A physical phone, a second browser engine, network shaping, cold application startup, normal/ORM textures and production-garden motion remain separate follow-ups. No phone results or changes to the production garden renderer are claimed.

## Data and reproduction files

[Browser lab ZIP](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/browser-lab.zip) and [Houdini lab ZIP](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/houdini-lab.zip) are the two parts of this dated study. Extract both into the same folder for the complete source, prepared fixtures and raw results. Each is below 24 MiB; their [manifest and SHA-256 hashes](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/reproduction-manifest.json) are public. Native runtimes and the encoder are not bundled. Public HDR copies omit workstation identity metadata while retaining exactly identical pixel arrays; timing and geometry data are unchanged.

[All 132 loading samples](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/compression-samples.csv) · [Grouped loading results](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/compression-summary.csv).

![Raw alpha leaf](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/leaf-raw.png)

![ETC1S alpha leaf](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/leaf-etc1s.png)

![UASTC alpha leaf](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/leaf-uastc.png)

## Related field notes

- [Three.js rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/)
- [GLB cache and first frame](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/)
- [Compression decision helper](https://digitalgarden.si/tools/web-3d-compression-decision-helper/)

AI-assisted disclosure: Codex assisted with the experiment implementation, execution, analysis and writing. The measurements, failures and limits are actual dated workstation results, not model estimates.
