# Spirula Studio to Three.js: An Official Garden Sample Test

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Official Mip-NeRF 360 Garden sample tested in Spirula Studio on Vulkan and Spark: 100,000 splats, PLY-to-SPZ sizes, browser results and reproducible limits.

- Author: Alicia Lu
- Published: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/spirula-studio-garden-splat-threejs/

**Short answer:** Spirula Studio's Windows Vulkan build produced a 100,000-splat Garden scene that loaded in Spark 2.4.0 with Three.js 0.186.1. A short 2,000-iteration run took 13.561 seconds for the whole process. Its 24.80 MB PLY converted to a 1.95 MB SPZ. Both rendered in a local WebGL scene, including an independently generated Houdini plant.

This was a test of the **official Mip-NeRF 360 Garden sample**, with camera poses already supplied. I did not film a garden on a phone, run fresh structure-from-motion, or add this sample to the production Digital Garden.

![Measured PLY and SPZ file sizes for the official Garden sample; timings cover different stages and are not a speed comparison](https://digitalgarden.si/marketing/research/spirula-20261024/pipeline-results.png)

Tested and written with Codex assistance on 24 October 2026. [Measurement JSON](https://digitalgarden.si/marketing/research/spirula-20261024/measurements.json), [reproduction package](https://digitalgarden.si/marketing/research/spirula-20261024/spirula-reproduction.zip), and [source and scope](https://digitalgarden.si/marketing/research/spirula-20261024/README.md).

## The input and the question

I wanted to check a concrete handoff: can a Vulkan-trained splat leave Spirula as a file, become small enough for a web experiment, and coexist with a regular GLB in Spark?

The [Mip-NeRF 360 project](https://jonbarron.info/mipnerf360/) provides an official outdoor Garden scene. I downloaded its 185 quarter-resolution images and sparse camera/point files from the project's archive. A selective range download transferred about 234 MB instead of the entire 12.5 GB multi-scene archive. This download is preparation, not a training-time result.

Using prepared camera poses deliberately excludes capture quality and SfM reliability. It makes the first experiment easier to interpret: if training or browser rendering fails, moving leaves or a bad phone trajectory are not additional unknowns introduced by a new capture.

## The measured setup

| Item | This run |
| --- | --- |
| Machine | Windows, NVIDIA RTX 5070 Ti, driver 617.42 |
| Trainer | Spirula Studio v2026.10.16, Windows Vulkan executable, commit 65246c5 |
| Input | Official Garden `images_4`, supplied `sparse/0` camera poses |
| Split | 161 training images, 24 held-out images |
| Training resolution | Additional divisor 2; effective 648 × 420 |
| Training budget | 2,000 iterations; cap of 100,000 splats |
| Browser | Chromium 155, Three.js 0.186.1, Spark 2.4.0 |
| Renderer | WebGLRenderer; 1280 × 720 backing buffer, DPR 1 |

I used the [official portable release](https://github.com/harry7557558/spirula-studio/releases/tag/v2026.10.16). Training used Vulkan; this experiment required no new Python or CUDA training environment. It does not test AMD, Intel, Apple hardware, or Spirula's alternative backends.

The important flags were:

```text
spirula.exe train 3dgs
  --data <official-garden-folder>
  --image-dir images_4
  --train-resolution-divisor 2
  --num-iterations 2000
  --cap-max 100000
  --eval-mode interval
  --eval-interval 8
  --save-eval-images 1
  --steps-per-save 2000
  --disable-viewer 1
  --keep-viewer-alive 0
```

The reproduction script includes the output folder and logging options, and refuses to overwrite an existing run. These settings are a deliberately short pipeline check, not a claim about default training quality.

## Training and the actual export

The process exited successfully. The checkpoint contained `splat.ply`, with a binary PLY header declaring 100,000 vertices and Gaussian color, opacity, scale, rotation and spherical-harmonic fields. That is the export verified here; I did not inspect every GUI format or test direct SPZ and mesh export.

| Observation | Result |
| --- | --- |
| Whole-process wall time | 13.561 s, including initialization, evaluation and save |
| Trainer-reported training time | 11.3152 s |
| Final splats | 100,000 |
| Held-out mean PSNR | 24.8733 |
| Held-out mean SSIM | 0.763251 |
| PLY bytes | 24,801,531 |

The PSNR and SSIM values are Spirula's output across its 24 held-out views. They are not measurements of the later SPZ or Spark render. I also sampled board-level GPU memory, but other applications were running. That log cannot establish a per-process peak VRAM claim.

Visual inspection agrees with the modest training budget: the table, vase and large garden shapes are recognizable, while grass and fine foliage are visibly softer than the held-out reference image. This is enough to validate the pipeline; it is not a final reconstruction-quality result.

## PLY to SPZ with Spark's own converter

Spark v2.4.0 includes an [official SPZ conversion script](https://github.com/sparkjsdev/spark/blob/v2.4.0/scripts/compress-to-spz.js), which calls the package's `transcodeSpz` export. I used the same API locally:

```javascript
import { transcodeSpz } from '@sparkjsdev/spark';

const result = await transcodeSpz({
  inputs: [{
    fileBytes: plyBytes,
    pathOrUrl: 'garden.ply',
    transform: {
      translate: [0, 0, 0],
      quaternion: [0, 0, 0, 1],
      scale: 1,
    },
  }],
  fractionalBits: 12,
});
```

Conversion took 171.89 ms on this machine and wrote **1,946,509 bytes**, 92.15% smaller than the PLY. The converter reported zero clipped splats. I applied no spatial crop or opacity filter, and the browser loaded 100,000 splats from both formats.

SPZ quantization is not a lossless equivalence claim. Equal splat counts and zero clipping do not prove identical colors or pixels, and this run did not calculate a post-conversion image-quality score.

## Spark rendering, orientation and a Houdini plant

The [SplatMesh API](https://sparkjs.dev/docs/splat-mesh/) can load the exported PLY and converted SPZ. My test scene used `WebGLRenderer`, a `SparkRenderer`, a `SplatMesh` with LOD disabled, and normal Three.js orbit controls.

Rather than copy an upside-down correction from an unrelated example, I used the dataset's actual camera pose. COLMAP stores a world-to-camera rotation with Y pointing down and Z forward. The viewer inverts that pose, derives the camera's up/forward vectors, and leaves the splat transform at identity. This rendered the garden upright without rotating the splat by guesswork.

I verified reset, rotation, zoom, file switching and a plant-visibility toggle in a real browser. The additional GLB is the independently owned Houdini hyacinth fixture from the plant export experiment. Its position and scale here are illustrative. It is not a metric reconstruction of a plant standing in this photographed garden.

The combined scene is useful, but the limitations are visible. Splat geometry does not give the table a normal mesh collider or a clean opaque surface for placing objects. A successful mixed render does not establish physically correct occlusion for every leaf or placement. I did not test Spark inside WebGPURenderer in this run.

## What the browser numbers mean

One foreground-tab orbit sample per condition lasted about ten seconds:

| SPZ scene | Frames | Average RAF FPS | Median interval | P95 interval |
| --- | --- | --- | --- | --- |
| Splat only | 601 | 60.003 | 16.7 ms | 16.8 ms |
| Splat plus Houdini plant | 601 | 60.002 | 16.7 ms | 16.8 ms |

Both samples are refresh limited on this desktop. They show that the small scene remained responsive during this camera path; they do not rank renderers or measure the performance cost of adding the plant. RequestAnimationFrame intervals are not GPU execution times, and no phone or thermal test ran.

The initial SPZ initialization observation was 120.6 ms; a later PLY switch took 84.2 ms. These came from localhost with an already running renderer, in a different order and cache state. They are functional observations, **not evidence that a 24.8 MB PLY loads faster over the internet than a 1.95 MB SPZ**.

## What is ready, and what comes next

The tested handoff is now concrete: official prepared Garden sample → Vulkan training → checkpoint PLY → Spark SPZ conversion → local Three.js WebGL scene. The commands, receipts and viewer source are available in the reproduction package so the numbers can be traced to a run.

The public reproduction package contains the commands, converter, viewer source and measurement records. Obtain the Garden dataset from its original project; the photographs and trained splats are not included in that package. The independent plant fixture and Garden dataset have separate provenance. A production addition should use a capture whose publication rights are clear, then test fresh SfM, a longer quality budget, mobile behavior and intentional object placement.

For related measured boundaries, see the [MuTracer fixed-sample plant test](https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/), [GLB loading and cache experiment](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/), and [Houdini versus Geometry Nodes plant export](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/).

For the surrounding workflow, see [Gaussian splats in Three.js](https://digitalgarden.si/blog/gaussian-splats-threejs/), [Spark versus GaussianSplats3D](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/), and the [Houdini-to-Three.js garden notes](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/).
