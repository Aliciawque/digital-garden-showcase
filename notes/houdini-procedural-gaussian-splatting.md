# Procedural Gaussian Splatting in Houdini: Six Fits Without Photos

> A procedural Houdini valley, 44 Karma EXRs and six gsplat fits compare 12, 24 and 36 training views, with held-out metrics and a verified SPZ export.

- Author: Alicia Lu
- Published: 2026-11-04
- Canonical HTML: https://digitalgarden.si/blog/houdini-procedural-gaussian-splatting/

## Key takeaways

- From 12 to 24 training views, foreground PSNR rose from 28.51–28.92 dB to 33.18–33.26 dB across two seeds.
- The selected 4,096-splat fit converted from a 278,942-byte PLY to a 64,915-byte SPZ. Both formats were displayed in Spark.

I rendered a small procedural valley in Houdini and trained six Gaussian splats on the resulting synthetic images. With the same 4,096 splats and 600 training steps, going from 12 to 24 training views improved held-out quality for both seeds. Going from 24 to 36 gave little extra full-frame PSNR and slightly lower foreground PSNR in both runs. This is a result for one 192 × 192 scene and one fixed-capacity trainer.

![Karma targets alongside 12-, 24- and 36-view predictions for two held-out cameras](https://digitalgarden.si/marketing/research/procedural-splats-20261104/held-out-visuals.png)

## Make a scene and keep the cameras honest

The test scene is a small valley diorama with a pool, rocks and eight trees. A Houdini 22.0.429 hython script built a USD stage from procedural primitives, set materials and lights, and saved a HIP with a relative reference to the source USD. Karma CPU produced 44 RGBA EXRs at 192 × 192, with 12 samples per pixel and motion blur disabled.

Training used the first 12 cameras, then a nested 24-camera set, then all 36 training cameras. The extra eight cameras use offset azimuths and alternating elevations. They were used only to render ground truth and score the finished fits. Initialization sampled random centers in a broad scene-sized box. The fitter received no geometry points, depth, or pixels from the eight held-out views. The scene's approximate dimensions and camera aim were authored and shared across the trial.

The renderer output was composited over a fixed background in render-linear RGB, then clipped and passed through a fixed gamma curve. It defines the training target for these numbers. It is separate from the native Karma viewer's display transform.

## Six GPU fits at fixed capacity

| Training views | Seed | Full-frame PSNR (dB) | Foreground PSNR (dB) | SSIM | Fit wall time (s) |
| --- | --- | --- | --- | --- | --- |
| 12 | 381 | 32.55 | 28.92 | 0.92114 | 13.039 |
| 24 | 381 | 35.80 | 33.26 | 0.94525 | 1.663 |
| 36 | 381 | 35.97 | 33.21 | 0.94670 | 1.717 |
| 12 | 719 | 32.18 | 28.51 | 0.91714 | 1.337 |
| 24 | 719 | 35.71 | 33.18 | 0.94457 | 1.443 |
| 36 | 719 | 35.82 | 33.11 | 0.94539 | 1.596 |

The workstation ran Torch 2.9.1+cu128 and gsplat 1.5.3 on an RTX 5070 Ti. Each fit optimized positions, scales, rotations, opacity and a view-independent RGB value, with no point splitting or pruning. Loss gave equal weight to full-frame and alpha-masked absolute error. Metrics are means of eight per-image scores. Foreground scores exclude target pixels with alpha at or below 0.5. SSIM uses a Gaussian 11 × 11 window.

The first fit had a much larger wall time than subsequent fits. Those times include initial work in the shared Python process and do not isolate a camera-count speed effect. Peak Torch allocations were 22.45–22.46 MiB and peak Torch reservations 26 MiB for this tiny scene, including cached target tensors. They do not report total driver or GPU residency. The 12-step pilot is separate from the six fits.

![Full-frame and foreground PSNR against training-view count for two seeds](https://digitalgarden.si/marketing/research/procedural-splats-20261104/validation-chart.png)

## Browser delivery and collision

I preselected the 36-view, seed-381 output for delivery. The binary PLY is 278,942 bytes and the SPZ v3 is 64,915 bytes, 76.7% smaller. Both decoded as 4,096 splats and reopened in Spark 2.4.0 on NVIDIA WebGL2. Browser screenshots retain visible blur and residual floaters. The numeric quality table describes the trainer's renders at 192 × 192. It does not score the browser's differently sized render or the format conversion.

The invisible Three.js proxy reproduces the base slab of the authored USD using a 2.64 × 0.26 × 2.26 metre box. Three downward rays inside its footprint hit the top at y = 0 within 4.77 × 10⁻⁹ m. Two outside rays missed. Both PLY and SPZ runs gave the same result because the rays tested the separate mesh. The test covers the slab only, with no physics solver, full collision mesh or relighting claim.

![Spark WebGL2 screenshot of the SPZ fit; blur and residual floaters remain visible](https://digitalgarden.si/marketing/research/procedural-splats-20261104/browser-spz.png)

The [previous Houdini native export study](https://digitalgarden.si/blog/houdini-22-gaussian-splats-threejs/) covers GSplat SOP and format transport. This study instead varies synthetic views and separates visual display from collision. [Spark SplatMesh reference](https://sparkjs.dev/docs/splat-mesh/).

[Reproduction archive and raw evidence](https://digitalgarden.si/marketing/research/procedural-splats-20261104/procedural-splats-reproduction.zip). Codex implemented and ran the experiment and drafted the article.

Related study: [Digital Garden field note](https://digitalgarden.si/blog/threejs-gaussian-splat-renderer-comparison/).

### Raw measurement files

- [training.json](https://digitalgarden.si/marketing/research/procedural-splats-20261104/training.json)

## Questions

### Did adding more than 24 views improve the result?

At 36 views full-frame PSNR improved slightly on both seeds. Foreground PSNR fell slightly on the same eight held-out views.
