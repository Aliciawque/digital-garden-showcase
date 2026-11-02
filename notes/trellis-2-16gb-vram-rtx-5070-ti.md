# TRELLIS.2 on a 16GB RTX 5070 Ti: 24 Runs, GLB Export and Shared Memory

> Twenty-four TRELLIS.2 runs on a 16GB RTX 5070 Ti track native GLB export, Windows shared GPU memory, decoder tiling and Three.js delivery.

- Author: Alicia Lu
- Published: 2026-11-02
- Canonical HTML: https://digitalgarden.si/blog/trellis-2-16gb-vram-rtx-5070-ti/

TRELLIS.2 generated all 24 requested outputs on this RTX 5070 Ti workstation, including actual 1536-resolution grids. Its native GLB export wrote 23 files. One official-path export raised `CUDA driver error: device not ready`; that is an observed export failure, not a confirmed out-of-memory exception.

The more consequential result appeared in a later successful run: Windows reported **10.30 GiB of shared GPU memory** in a snapshot during the slow official 1536 export. Completion on a card sold with 16GB therefore does not establish that the whole pipeline fit inside 16GB of local VRAM.

I also checked what those files contained, reopened imported Blender scenes, and ran 24 fresh-profile Three.js starts. Generation, writing a GLB, validating it, and delivering it to a browser each produced a different answer.

Measured on **2 November 2026**. Codex assisted the experiment, instrumentation and writing. These are three owned synthetic fixtures, one fixed seed, one desktop and one generated asset for the browser comparison. The production Digital Garden was not migrated.

## What ran, and what stayed fixed

The workstation has an RTX 5070 Ti 16GB, driver 617.42 and a Ryzen 9 9950X. Windows 11 Pro exposed about 61.65 GiB of system RAM. Ubuntu 24.04.2 under WSL2 had a 30.18 GiB RAM limit and 8 GiB of swap. The isolated environment used Python 3.12.3, PyTorch 2.8.0+cu128, CUDA compiler 12.8.93, FlashAttention 2.8.3.post2 and Transformers 4.57.1.

Microsoft's pinned [TRELLIS.2 README](https://github.com/microsoft/TRELLIS.2/tree/75fbf0183001ed9876c8dbb35de6b68552ee08bd) specifies Linux and at least 24GB of GPU memory. This Windows/WSL experiment does not revise that requirement.

The inputs were transparent 768×768 Blender renders of a watering can, a potted plant and a bench. Both paths used the same weights, seed 42, 12-step sampler defaults, low-VRAM offloading and a 49,152-token limit. All returned grid resolutions matched the requested 512, 1024 or 1536 tier; no observed success depended on silently falling back to a lower tier.

The primary matrix contains 18 fresh Python processes: three inputs × three tiers × two source paths, with `expandable_segments:True`. Six additional watering-can processes rechecked the default allocator. They are separate controls, not six more independent objects or a causal allocator benchmark.

The official source is commit `75fbf0183001ed9876c8dbb35de6b68552ee08bd`. The community comparison uses [Espalier's old MIT-licensed revision `13344be628517b5fc0c6a0969d0ca693b650cf44`](https://github.com/overyonder/espalier/tree/13344be628517b5fc0c6a0969d0ca693b650cf44), including tiled decoding with a 256-voxel core and margin 8. It does not execute the current branch or the full Espalier wrapper. Both paths use identical compiled baseline native extensions. The archive preserves the patch, license, hashes and CRLF-to-LF normalization.

Only local model-path adapters and an unused background-removal constructor stub are common changes. The transparent inputs use the upstream alpha preprocessing path; RMBG was not exercised. DINOv3 was obtained through approved access and its downloaded weight matched the official LFS hash. **Built with DINOv3:** this research uses Meta's DINOv3 feature model, subject to the [DINOv3 license](https://ai.meta.com/resources/models-and-libraries/dinov3-license/). No model weights are distributed with the study.

![Generation and native-export outcomes for the 18 primary TRELLIS.2 cases, with separate observed timing medians](https://digitalgarden.si/marketing/research/trellis2-16gb-20261102/tier-outcomes.png)

## Generation completed more often than native export

The native export retained the official example's 1,000,000-triangle target, 4096 textures, remeshing, band 1, projection 0 and WebP output. I did not reduce those settings after the failure. Saving private raw tensors and applying the example's 16,777,216-face safety cap happened before the timed `to_glb` plus file-write stage; whole-process wall time includes that work.

| Requested tier | Path | Generated | GLBs written | Generation median, s | Export/write median, s |
| --- | --- | --- | --- | --- | --- |
| 512 | Official | 3/3 | 3/3 | 24.33 | 31.22 |
| 512 | Patched | 3/3 | 3/3 | 24.51 | 35.40 |
| 1024 | Official | 3/3 | 3/3 | 54.84 | 105.55 |
| 1024 | Patched | 3/3 | 3/3 | 97.69 | 49.87 |
| 1536 | Official | 3/3 | 2/3 | 132.47 | 129.64 |
| 1536 | Patched | 3/3 | 3/3 | 123.09 | 88.68 |

Generation medians use all three different inputs. Export medians use written files only, so the official 1536 export has **n=2**. A written file is not necessarily a format-valid file.

Only the official 512 pilot had been warmed beforehand. New paths, tensor shapes and resolutions could compile kernels later; disk caches persisted. The patched 1024 generation median being slower is one reason these observations should not become a steady-state speed ranking. The later default-allocator 1024 watering-can runs took 43.96 seconds officially and 45.10 seconds patched, after more caches had warmed. Changing cache state prevents attributing that difference solely to allocator configuration.

All six default-allocator rechecks generated and wrote files. The official 1536 recheck required **121.20 seconds for generation, 475.52 seconds for export/write and 697.85 seconds inside the trial runner**—about 700 seconds including outer process orchestration. Before export its safety-capped mesh had 15,538,939 faces; remeshing temporarily expanded that to 36,114,868 faces. A roughly one-million-triangle delivery target does not bound the intermediate mesh.

## Why the memory counters must stay separate

![Separate PyTorch allocated and reserved generation peaks, whole-trial NVIDIA totals, and one late Windows dedicated/shared-memory snapshot](https://digitalgarden.si/marketing/research/trellis2-16gb-20261102/memory-accounting.png)

| Primary tier/path | Max generation allocated, GiB | Max generation reserved, GiB | Max sampled NVIDIA whole-trial total, GiB |
| --- | --- | --- | --- |
| 512 official | 2.62 | 2.67 | 6.93 |
| 512 patched | 2.62 | 2.67 | 6.79 |
| 1024 official | 5.18 | 7.82 | 12.74 |
| 1024 patched | 4.36 | 5.11 | 7.58 |
| 1536 official | 10.11 | 12.37 | 15.55 |
| 1536 patched | 6.36 | 7.95 | 8.94 |

Each cell is the observed maximum across three inputs within its own measurement scope. These are not interchangeable measurements of process-only VRAM. [PyTorch's allocator statistics](https://docs.pytorch.org/docs/2.8/notes/cuda.html) distinguish allocated tensors from reserved blocks, and [its memory tools](https://docs.pytorch.org/docs/2.8/torch_cuda_memory.html) cannot see every direct CUDA allocation.

A single NVIDIA sampler recorded 56,381 valid rows across the formal trials, including desktop usage. The intended interval was 100 ms; the observed median was 101 ms and the largest gap 2,032 ms. Short peaks can be missed, and the changing global baseline cannot be assigned to a particular application.

Windows adapter-wide counters were added late. The **single** snapshot at 16:27:30 Beijing during formal trial 23 reported 15.57 GiB dedicated usage and **10.30 GiB shared usage**. It is supplemental evidence, not that process's full-run peak. Microsoft's [GPU-memory accounting explanation](https://devblogs.microsoft.com/directx/gpus-in-the-task-manager/) distinguishes these categories. The default 1536 generation also reported 18.78 GiB reserved through PyTorch, above physical card capacity.

The successful patched default recheck had 110 late Windows observations, with observed shared usage at most 0.094 GiB in that partial window. Early primary trials lack those counters. I did not inspect or change a driver system-memory fallback setting, and these data do not identify the precise fallback mechanism. They do rule out presenting successful completion as proof of a universally local-VRAM-only 16GB workflow.

## Recovering the failed output did not require regenerating it

The failed official watering-can run had already saved a real 1536 raw result: 9,103,700 vertices and 18,252,884 faces. Three fresh, export-only controls reused that exact result without loading the generation models.

| Export-only control | File written | Export/write, s |
| --- | --- | --- |
| Official, GPU attributes | No; device not ready again | 6.08 |
| Patched, GPU attributes | Yes | 83.92 |
| Patched, CPU-staged attributes | Yes | 87.93 |

Those controls are outside the 24 full-pipeline denominator. No formal trial raised a literal OOM exception. A Windows System-log query found no matching display/NVIDIA events in the inspected window, which neither diagnoses the CUDA error nor excludes a driver fault.

Validation exposed a second issue. Of the 23 native files, **17 passed without errors and six contained zero-length normals**. Both written recovery files also contained zero normals. I retained all originals and produced eight separate corrected copies, replacing **21 exactly zero normal vectors** using adjacent geometry or valid adjacent normals. Positions, indices, UVs, every nonzero normal and encoded texture bytes remained identical. All eight derivatives passed with zero glTF validation errors.

This small repair leaves topology and degenerate triangles untouched. It is not a manifold, animation or production-quality certification. With the recovered, corrected file, every generated trial now has an explicitly mapped format-valid local delivery choice; that does not rewrite the original 23/24 export result.

![Owned synthetic inputs beside actual 1536 generated GLBs imported and CPU-rendered in Blender; the bench uses the documented normal correction](https://digitalgarden.si/marketing/research/trellis2-16gb-20261102/input-output.png)

Six native-import Blender scenes were saved, reopened, their packed images decoded and rendered again. These show recognizable shapes under different lighting, not a ground-truth reconstruction score. Blender can drop degenerate triangles during import: the representative watering can changed from 989,733 source triangles to 989,723 imported triangles. Reopening proves the Blender scene persisted, not a bit-exact glTF round trip.

The nine official/patched raw pairs were also compared after voxel-coordinate alignment. All three 512 cases matched aligned attributes exactly; other pairs sometimes differed. The largest aligned attribute difference was 0.0016203. Deterministic nearest-vertex probes of 20,000 vertices in each direction reached a maximum sampled distance of 5.54×10⁻⁶ normalized scene units. That is neither full-surface Hausdorff distance nor proof of patch-only numerical error: identical-within-path stochastic repeat controls were not run.

## A browser-ready copy is another deliberate tradeoff

The highest-resolution completed patched watering can supplied the browser test. Its native GLB is 35,546,480 bytes with 989,733 triangles and two 4096 WebP maps. The web baseline applies welding, simplification ratio 0.1 with error limit 0.001, and two 2048 WebP maps at quality 95. The actual output retained **154,568 triangles**, rather than an assumed 100,000.

Meshopt and Draco start from that same web baseline with declared 14-bit positions, 10-bit normals and 12-bit UVs. Native-to-web changes geometry and textures; the four files are not lossless encodings of one identical asset.

![Actual payload sizes and Three.js startup medians and ranges for native, reduced, Meshopt and Draco assets on WebGPU and WebGL2](https://digitalgarden.si/marketing/research/trellis2-16gb-20261102/web-delivery.png)

| File | Bytes | Triangles | WebGPU median, ms | WebGL2 median, ms |
| --- | --- | --- | --- | --- |
| Native | 35,546,480 | 989,733 | 376.7 | 352.1 |
| Web baseline | 8,008,016 | 154,568 | 188.1 | 166.9 |
| Meshopt | 2,625,716 | 154,568 | 180.1 | 167.7 |
| Draco | 1,154,304 | 154,568 | 218.0 | 200.5 |

These are local asset-fetch-through-first-GPU-completion measurements, three fresh profiles per file/backend. Renderer and procedural environment initialization happened before that timer; navigation timings retain their cost. Completion fences do not measure screen presentation.

All **24 formal starts and 2,880 post-ready RAF intervals** passed with physical NVIDIA WebGPU/ANGLE WebGL2 adapters, nonblank captures and no JavaScript page errors. Chrome 156 headless used Three.js 0.186.1 at 1280×720, DPR 1. Separate preflights and twelve alternate-view captures are excluded from those timing medians. Browser profiles were fresh, but driver caches remained. Frame intervals centered around 16.7 ms under the roughly 60 Hz cap; they cannot rank uncapped GPU throughput.

Serialized and independently decoded compression branches passed glTF validation. Two RGBA8 maps with ideal mip chains imply about 170.67 MiB at 4096 or 42.67 MiB at 2048. Those are logical texture estimates, not measured driver VRAM; a small WebP download alone does not demonstrate low residency.

## Files and the practical boundary

The [reproduction package](https://digitalgarden.si/marketing/research/trellis2-16gb-20261102/trellis2-16gb-study-20261102.zip) contains owned PNGs and editable input scenes, pinned patches/licenses, scripts, trial receipts, CSVs, memory samples, figures, native import receipts and the three bounded web derivatives. [The manifest and instructions](https://digitalgarden.si/marketing/research/trellis2-16gb-20261102/README.md) identify omissions. Gated weights, private raw tensors, large native GLBs and large imported scenes remain local. Model access and native environment setup are prerequisites for regenerating them; the packaged browser example can be checked separately.

For this workstation, tiled decoding reduced the observed high-tier generation allocation and the patched export recovered an otherwise failed raw result. The useful next gate is an inspected, validated asset with a measured web budget. A completed generation alone does not answer that question, and a 16GB card's name does not describe the whole memory path.

Related: the [ComfyUI 3D asset workflow](https://digitalgarden.si/blog/comfyui-3d-workflow/), and the [Three.js garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/) place generation inside the broader delivery pipeline.
