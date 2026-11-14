# Image to Gaussian Splat on a 16GB GPU: Idol and Three.js Tested

> Idol tested on an RTX 5070 Ti: 14 local jobs, cold and warm timing, memory, PLY sizes, and Three.js/WebGL2/Spark loading with owned garden images.

- Author: Alicia Lu
- Published: 2026-11-13
- Canonical HTML: https://digitalgarden.si/blog/image-to-gaussian-splat-threejs/

## Key takeaways

- Fourteen local generation jobs completed on the RTX 5070 Ti under WSL2; native Windows and iPhone remain separate tests.
- The default 262,144-splat call took 47.68 seconds first and 29.52 seconds warm; its uncompressed PLY is 17,826,208 bytes.
- Nineteen browser loads passed with explicit placement and color decoding; submitted-draw timing is distinct from visible-pixel latency.

An image-to-Gaussian-splat model is useful to my garden workflow only if its output survives a local browser loader. I tested Idol on an RTX 5070 Ti with two owned garden-object renders, then opened the generated PLY files with Three.js and Spark.

The default 50-step, 262,144-Gaussian request completed in **47.68 seconds on the first call with new compile caches**, then **29.52 seconds for the same input in the same process**. The default PLY was **17,826,208 bytes**. Fourteen generation jobs and nineteen browser loads completed. These measurements describe a Windows workstation running CUDA through WSL2, with desktop browser checks on Windows.

Codex assisted source review, instrumentation, local runs, figures and this manuscript. All result cells below come from saved machine records or browser readback. The article is a desktop study; physical iPhone Safari and public-network HTTPS remain untested.

## Can Idol run an image-to-Gaussian-splat job on this 16GB card?

| Question | Observed result |
|---|---|
| Default local job finishes? | Yes: 50 steps, seed 42, 262,144 Gaussians |
| First / later call | 47.68 / 29.52 s, with initial model load and saving separate |
| PyTorch peak allocated / reserved, first call | 4.983 / 5.951 GiB |
| Sampled whole-GPU use, first call | 8.628 GiB, including the desktop and allocator |
| Standard PLY output | 262,144 vertices, 17,826,208 bytes, finite fields |
| Browser result | Three.js WebGPU, its WebGL2 fallback and Spark WebGL2 displayed the object |
| Production garden / iPhone / public HTTPS | Isolated loader preview tested; production placement, phone and public-network tests are still open |

I use “runs on the 16GB card” to describe completed jobs on this workstation. The Windows adapter trace also reported shared memory, reaching 201.8 MiB across the observed adapters. Those counters include unrelated desktop work and stop a few seconds before the final process exits. They cannot establish per-process shared-memory use or a zero-spill claim. Full-call PyTorch and sampled whole-GPU records are retained separately.

## Which model and environment did we actually test?

[Idol's official SDK](https://github.com/speridlabs/idol-sdk/tree/8db700349a5570aac390e4780fa32e3eed6b587a) turns an object image into a standard 3DGS PLY, a preprocessed input PNG and predicted-camera JSON. The PLY here contains 17 float fields per Gaussian and degree-zero spherical-harmonic color. Its zero mesh-normal columns are expected for this Gaussian export; they were left intact.

| Component | Measured configuration |
|---|---|
| GPU / driver | RTX 5070 Ti, 16,303 MiB reported / 621.14 |
| Execution | Windows host, WSL2 Ubuntu, Linux CUDA |
| Python / PyTorch / CUDA build | 3.12.3 / 2.8.0+cu128 / 12.8 |
| SDK commit | 8db700349a5570aac390e4780fa32e3eed6b587a |
| Transformers / Diffusers / Hub | 5.23.0 / 0.43.0 / 2.4.0 |
| Triton / attention | 3.4.0; SDPA selected, SDK cuDNN attention while sampling |
| Flash Attention | Installed 2.8.3.post2; not selected for this matrix |
| Sampling | 50 steps, CFG 3.0, time shift 3.0, seed 42, optimization enabled |
| Encoder precision | DINOv3 bf16; VAE loaded as bf16 |

The environment resolves the SDK's dependencies while reusing the earlier local Torch/CUDA installation through a read-only dependency path. The older environment was preserved. An active-interpreter dependency audit passed. Cross-platform Git marked CRLF text as dirty; all 121 tracked SDK objects match the pinned commit after declared line-ending normalization. The SDK's inference code was not patched.

The [official model card](https://huggingface.co/speridlabs/idol) mentions under-two-second generation, while its 50-step table lists 14.1 seconds for a 5090 and 19.1 seconds for a 4090. It has no 5070 Ti row. Those are vendor measurements with their own setup; they are not cells in my result table.

## What happened when the VAE download failed?

The logged-in Hugging Face account already had access to the exact DINOv3 H+ and FLUX.2-dev repositories. The downloaded DINOv3 file matched the official 3,362,432,800-byte SHA-256. Browser automation could not complete the separate VAE transfer.

An existing, unmodified ComfyUI file turned out to match FLUX.2-dev's official **ae.safetensors** checkpoint. I used the [pinned Diffusers conversion helpers](https://github.com/huggingface/diffusers/blob/61f175660a8ac54f1470a74a810e6c38fb4795d5/scripts/convert_flux2_to_diffusers.py) to produce the folder layout Idol expects. Following the official conversion's default BatchNorm training-counter handling produced a file that matches the target VAE **byte for byte**:

```text
Original checkpoint SHA-256:
868fe7b343cc8f3a19dbcfcafbc3d5f888802be3f89bd81b65b3621a066ce8f3

Converted VAE SHA-256, identical to the official Diffusers file:
d64f3a68e1cc4f9f4e29b6e0da38a0204fe9a49f2d4053f0ec1fa1ca02f9c4b5
```

The original ComfyUI checkpoint and downloaded files remain unchanged. No browser protection was disabled, and no token is included in the records or reproduction materials.

Idol accepts local encoder folders through its documented Python API:

```python
from idol import IdolPipeline

pipeline = IdolPipeline.from_pretrained(
    "models/idol",
    device="cuda",
    dinov3_dtype="bf16",
    optimize=True,
    image_encoders={
        "dinov3": "models/dinov3-hplus",
        "flux2_vae": "models/flux2-dev",
    },
)
result = pipeline("inputs/potted-plant.png", mode="synthetic", seed=42,
                  steps=50, cfg_strength=3.0, rescale_t=3.0,
                  num_gaussians=262144)
result.gaussian.save_ply("plant.ply")
result.preprocessed_image.save("plant_input.png")
```

The SDK/model's Apache-2.0 label does not replace the separate DINOv3 and FLUX.2-dev terms. Model weights are excluded from the reproduction archive. The archive's notices describe the owned fixtures and code separately.

## What was the input, and did background removal run?

The primary input is an owned 768 × 768 RGBA Blender render of a simple potted plant. A watering-can render is the second input. Both were authored for the earlier local study and reused as input fixtures; all generations and timings in this article are new.

![Owned plant render and the actual synthetic and real-mode preprocessed images](https://digitalgarden.si/marketing/research/idol-20261113/01-input-and-crop.png)

An existing alpha channel takes Idol's official alpha branch and skips learned background removal. That distinction matters for timing. I added a white-background RGB composite of the same owned plant to exercise BiRefNet. It is still a synthetic render, not a photograph. The deliberately mismatched real-mode row changes the crop margin from 1.2 to 1.7 and is labeled as a control.

## How long did the local jobs take, and how large were the files?

The following table uses the default dev VAE. Time is a synchronized pipeline call, including image preprocessing, encoders, sampling and decoding. Initial model load was 19.59 seconds and is separate. Disk saving is separate. The first call uses a fresh process and new Inductor/Triton cache directories; existing driver caches were not flushed.

| Job | Call s | Torch allocated GiB | Torch reserved GiB | Whole GPU sampled GiB | PLY bytes |
|---|---:|---:|---:|---:|---:|
| First RGBA, new compile cache | 47.68 | 4.983 | 5.951 | 8.628 | 17,826,208 |
| Same RGBA, same process | 29.52 | 4.983 | 5.951 | 8.626 | 17,826,208 |
| Second owned image | 29.53 | 4.983 | 5.953 | 8.626 | 17,826,208 |
| 65,536 Gaussians | 30.54 | 4.983 | 5.957 | 8.824 | 4,456,863 |
| Real-mode crop control | 31.77 | 4.983 | 5.953 | 8.814 | 17,826,208 |
| RGB with background removal | 36.73 | 4.983 | 6.820 | 9.398 | 17,826,208 |
| 1,048,576 Gaussians | 33.37 | 4.983 | 5.953 | 8.526 | 71,303,585 |

The first and warm PLYs match byte for byte in this process. The 65,536-point output is 4,456,863 bytes; the million-point output is 71,303,585 bytes. The default raw PLY is therefore a 17.83 MB transfer before any compression or alternative-format conversion. This study does not measure SPZ/SOG conversion.

![Fourteen measured local requests and three actual uncompressed PLY file sizes](https://digitalgarden.si/marketing/research/idol-20261113/02-generation-and-bytes.png)

I also ran the same seven conditions using the SDK-supported [FLUX.2-klein-4B VAE](https://huggingface.co/black-forest-labs/FLUX.2-klein-4B), an ungated Apache-2.0 source. Its checkpoint is BF16 and has a different file hash; I retain it as a separate source/control.

| Condition | Default dev VAE s | Klein VAE control s |
|---|---:|---:|
| First RGBA, new compile cache | 47.68 | 55.85 |
| Same RGBA, same process | 29.52 | 29.50 |
| Second owned image | 29.53 | 29.71 |
| 65,536 Gaussians | 30.54 | 29.46 |
| Real-mode crop control | 31.77 | 29.72 |
| RGB with background removal | 36.73 | 30.65 |
| 1,048,576 Gaussians | 33.37 | 30.85 |

Each condition is one observation at seed 42. The two objects do not establish broad image quality, and seven controls are not seven independent stochastic repeats. Cross-source output hashes differed even though the converted weight comparison matched the BF16 values apart from the historical training counter. The saved outputs, rather than an assumed determinism guarantee, define the result. These observations do not establish a speed or quality ranking between VAEs.

## What do the unseen sides look like?

The plant keeps a recognizable terracotta pot, stem and broad leaves when rotated. Side views show thin, curled or merged-looking leaf edges. The opposite view supplies a plausible hidden arrangement, but it is inferred content; it does not verify the hidden surfaces of the original Blender scene. The watering can retains its blue body, spout and curved handle.

![Actual browser captures of the generated plant from four headings](https://digitalgarden.si/marketing/research/idol-20261113/03-turnaround.png)

The input-facing heading is derived from Idol's predicted camera and the export basis. The preview uses its own distance and field of view, so these images are not registered ground-truth comparisons. No photogrammetry accuracy, PSNR, rigging or mesh-topology score is claimed.

## Will the PLY display in Three.js and Spark?

Yes, in the isolated desktop loader built for this study. It uses the garden project's Three.js **0.186.1** and Spark **2.3.1**, with a 960 × 540 canvas at DPR 1. The page makes unique, no-store model requests and creates a new renderer for each load in the same browser page.

Two integration details were visible in the initial attempts. The SDK's exported basis put the plant upside down in this viewer; an explicit X rotation of pi made placement upright. The native Three.js path initially appeared pale; decoding these SH0 RGB values from sRGB into the linear working space corrected the preview. Both are viewer adaptations. The generated PLYs were preserved without edits.

```javascript
import { GaussianSplat } from "three/addons/objects/GaussianSplat.js";
import { GaussianSplatPLYLoader } from "three/addons/loaders/GaussianSplatPLYLoader.js";
import { colorSpaceToWorking } from "three/tsl";
import { SRGBColorSpace } from "three";

const geometry = await new GaussianSplatPLYLoader().loadAsync("plant.ply");
const splat = new GaussianSplat(geometry);
splat.rotation.x = Math.PI;
splat.material.colorNode = colorSpaceToWorking(splat.material.colorNode, SRGBColorSpace);
scene.add(splat); // render with WebGPURenderer or its forceWebGL fallback
```

![The same generated PLY displayed by Three.js WebGPU, WebGL2 fallback and Spark](https://digitalgarden.si/marketing/research/idol-20261113/04-browser-paths.png)

| Path | Actual backend | Median load-to-submitted-draw ms | Observed range ms | Loads |
|---|---|---:|---:|---:|
| native-webgpu | WebGPUBackend | 243.0 | 236.2–277.2 | 3 |
| native-webgl | WebGLBackend | 297.7 | 264.9–300.5 | 3 |
| spark | WebGLRenderer | 83.5 | 74.3–93.2 | 3 |

The automatic initial WebGPU load is excluded from that three-load summary. Final validation includes **19 loads and 456 post-ready rendered RAFs**, covering the million-point file on all three paths, the second object, the Klein control, and smaller/RGB controls. The earlier upside-down and pale-color phases remain in separate diagnostic records.

These timings end at submission of the first draw. In particular, a Spark worker can continue preparing visible data after its initial submission. Actual visible output was checked and captured after 24 further frames. The numbers are not first-visible-pixel latency or an engine-speed ranking. Spark's worker fetch did not appear in the main-thread Resource Timing entries; the shared PLY's file bytes are known, while its worker wire-transfer bytes remain unmeasured. JavaScript heap values are retained as context, not GPU-memory measurements.

This is a local loader integration experiment. The model has not been placed in the production garden, and a 17.83 MB loopback load does not predict download time on a phone or public website.

## Reproduction and author

The [reproduction bundle](https://digitalgarden.si/marketing/research/idol-20261113/idol-threejs-study-20261113.zip) contains the two owned inputs, two sample PLY exports (262,144 and 65,536 points), actual JSON/CSV records, viewer code, measurement scripts, figures and license notices. It excludes model weights, private tokens, compile caches and user scenes. The complete private working evidence also retains all fourteen outputs and the failed setup attempts.

For the wider browser-rendering context, see the [Three.js garden rendering guide](/blog/threejs-3d-garden-rendering/).

**Alicia — 3D technical artist, TA and independent developer.** Original study text, figures and owned fixtures: CC BY 4.0. Original instrumentation: MIT; upstream components retain their licenses. Those notices do not relicense model weights.

## Questions

### Is this a mesh or a Gaussian splat?

A Gaussian splat. This PLY stores positions, colors, opacity, scale and rotation; it does not provide editable polygon topology, UVs, a collision mesh or a skeleton. For mesh delivery, the separate TRELLIS.2 local study covers a different pipeline and different measurements.

### Is the local test free to run?

No paid generation API was used. Model access and applicable licenses still need to be handled for the selected encoders. The test retains local model paths rather than embedding credentials.

### Did it actually take two seconds on the 5070 Ti?

No. The default first call was 47.68 seconds and the same-process warm call was 29.52 seconds, at the declared 50-step settings. Download, model load and disk save are separately recorded.

### Does real mode mean that the input was a real photograph?

No. The real-mode row deliberately applies the alternate crop to the same synthetic RGBA image. The RGB background-removal row is also an owned render. No new photograph was tested.

### Does the same URL work in iPhone Safari?

Physical iPhone Safari, public-network HTTPS and production-scene placement remain untested. The demonstrated browser run is Windows Chromium on loopback HTTP. A later device test will receive its own results rather than inheriting these desktop numbers.
