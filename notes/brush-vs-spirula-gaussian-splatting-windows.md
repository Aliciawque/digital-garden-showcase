# Brush vs Spirula Studio on Windows with the Same Posed Garden

> Brush and Spirula Studio train one posed garden on a Windows RTX 5070 Ti. Four PLY exports and seven matched held-out images at 600 and 6,000 steps.

- Author: Alicia Lu
- Published: 2026-11-05
- Canonical HTML: https://digitalgarden.si/blog/brush-vs-spirula-gaussian-splatting-windows/

## Key takeaways

- Both trainers exported valid PLY files at 600 and 6,000 steps on the same 36 posed images.
- At 6,000 steps, independent RGB PSNR on seven common views was 21.48 dB for Brush and 38.17 dB for Spirula on this synthetic scene.

I trained a synthetic garden in Brush 0.3.0 and Spirula Studio 2026.10.16 on one Windows RTX 5070 Ti. Both exported a Gaussian PLY at 600 and 6,000 steps. At 6,000 steps, the saved RGB images averaged 21.48 dB PSNR for Brush and 38.17 dB for Spirula on the same seven held-out views. This result covers the chosen budgets, settings and scene.

![Two matched held-out views from Karma, Brush and Spirula at 600 and 6,000 training steps](https://digitalgarden.si/marketing/research/splat-training-windows-20261105/matched-training-views.png)

## The shared input and training limits

The 44 posed RGB images and camera intrinsics come from the [Houdini procedural splatting study](https://digitalgarden.si/blog/houdini-procedural-gaussian-splatting/). Each image is 192 × 192. I used views 1–36 for training and held out 37–44. Both trainers used SH0 and a 12,000-splat cap, with their own random initialization and optimization settings. Brush used wgpu on Vulkan and seed 381. Spirula used its Vulkan release, 3dgs preset, and explicit random initialization. The GPU driver was NVIDIA 617.42. This experiment starts with known camera poses.

Brush received separate Nerfstudio train and test manifests. Spirula needed a single transforms.json. I copied the identical PNGs, added train and eval prefixes for its filename split, and kept the original camera matrices and intrinsics. An initial Spirula probe using only the split manifests reported that transforms.json was missing. The derived dataset fixed the packaging issue. [Spirula dataset implementation](https://github.com/harry7557558/spirula-studio/blob/65246c5/src/data/parsers/DatasetCommon.cpp).

## Four exports and one independent comparison

| Trainer | Steps | Process wall time (s) | Exported splats | PLY bytes | Mean RGB PSNR (dB, seven views) |
| --- | ---: | ---: | ---: | ---: | ---: |
| Brush | 600 | 6.121 | 11,680 | 654,515 | 13.99 |
| Brush | 6,000 | 18.370 | 12,000 | 672,435 | 21.48 |
| Spirula | 600 | 1.490 | 1,200 | 82,014 | 28.21 |
| Spirula | 6,000 | 12.320 | 11,992 | 815,871 | 38.17 |

I matched the saved Spirula reference PNGs to the original input by exact decoded pixels. The 600-step output repeated view 37 and omitted view 39. The two renderings of view 37 were identical. Both Brush runs and the 6,000-step Spirula run provided all eight views. I kept views 37, 38, 40, 41, 42, 43 and 44 for every row. I independently measured full-frame PSNR in 8-bit RGB, then averaged the seven image-level dB values. The table uses those recalculated values. The application logs keep their own evaluation values.

The PLY count comes from each exported header. Spirula reported 12,000 active points at the end of the longer run and wrote 11,992 points. Process timing includes startup, import, training, evaluation and export. One run per configuration and the different initialization, growth and settings limit conclusions about speed or trainer quality. I kept whole-board memory samples separate from any per-process VRAM claim.

## Fixing the Brush image-header failure

The first two Brush attempts failed with early eof, using both a directory and ZIP. Brush 0.3.0 reads at least 16,387 bytes during its image-header probe. Two otherwise valid compressed PNGs were shorter. I regenerated all 44 PNGs with compression level zero and verified that every decoded RGB pixel matched the earlier files. The reproduced dataset includes the larger lossless PNGs. [Brush 0.3.0 loader](https://github.com/ArthurBrussee/brush/blob/v0.3.0/crates/brush-dataset/src/scene.rs).

Both used freely available Windows builds from their official releases. The test covers posed synthetic inputs and exported PLYs. It leaves real photos, SfM and other trainers for separate runs. [Brush releases](https://github.com/ArthurBrussee/brush/releases/tag/v0.3.0), [Spirula Studio release](https://github.com/harry7557558/spirula-studio/releases/tag/v2026.10.16).

Alicia is a 3D technical artist, TA and independent developer. Codex assisted with the dataset adapters, measurement and editing.

## Related studies

- [Train a splat from Houdini Karma renders](https://digitalgarden.si/blog/houdini-procedural-gaussian-splatting/)
- [Spirula Studio on the official Garden sample](https://digitalgarden.si/blog/spirula-studio-garden-splat-threejs/)

## Data and reproduction

- [matched-training.json](https://digitalgarden.si/marketing/research/splat-training-windows-20261105/matched-training.json)
- [splat-training-windows-reproduction.zip](https://digitalgarden.si/marketing/research/splat-training-windows-20261105/splat-training-windows-reproduction.zip)

## Questions

### Did the Windows comparison measure pose estimation?

The two trainers started from the same known camera intrinsics and poses. The experiment measures training, evaluation and export after pose estimation.
