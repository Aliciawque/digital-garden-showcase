# Houdini 22 Gaussian Splats to Three.js: A Native Export Test

> Native Houdini GSplats export tested with a 124-point control, a 12,000-splat Karma plant and 12 browser starts: PLY, SPZ and color limits.

- Author: Alicia Lu
- Published: 2026-11-02
- Canonical HTML: https://digitalgarden.si/blog/houdini-22-gaussian-splats-threejs/

**Houdini 22 produced a usable web splat, but a successful PLY write was not enough.** The native point writer used Houdini attribute names and big-endian binary output. Spark 2.3.1 rejected that byte order. I tested an explicit inverse attribute conversion and a separate, value-preserving endian adapter, then loaded both PLY and SPZ in Three.js.

The second fixture went through the official Karma-to-GSplats recipe: 24 rendered views of an owned procedural plant, a 12,000-point initialization, 1,200 training steps and native PLY/USD export. Its PLY was **3,121,605 bytes**; its SPZ derivative was **235,748 bytes**. Both displayed 12,000 splats. Native Karma and browser appearance still differed, so this is a verified transfer with documented limits, not a lossless rendering claim.

Codex assisted the experiment and writing. The raw data, failed setup records, editable scenes and exact adapters accompany the article. Neither fixture is a photographic capture or the production Digital Garden scene.

![The same trained plant rendered from a matching camera in native Karma XPU, standard PLY in Spark and SPZ in Spark, with remaining color differences visible](https://digitalgarden.si/marketing/research/houdini-gsplats-20261102/plant-views.png)

## What is native, and what changed in H22?

The [Bake GSplats SOP](https://www.sidefx.com/docs/houdini/nodes/sop/bakegsplat.html) dates to **Houdini 21.0**. Houdini 22 adds the [ML Train GSplats TOP](https://www.sidefx.com/docs/houdini/nodes/top/ml_traingsplats.html) and the [ML Train GSplats from Karma recipe](https://www.sidefx.com/docs/houdini/ml/train_solutions/ml_traingsplatsfromkarma.html) used here. The node called Surface Splat is an unrelated paint-mask tool; its name does not establish Gaussian-splat export support.

The workstation was Windows 11, Ryzen 9 9950X, 64 GB RAM, RTX 5070 Ti 16 GB and driver 617.42. Houdini was **22.0.429** with a commercial license. Its isolated training environment installed Torch **2.9.1+cu128**, torchvision 0.24.1+cu128 and gsplat **1.5.3**. The browser lab pinned Three.js **0.186.1** and Spark **2.3.1**; the production garden dependencies were not upgraded.

## First check: a controlled round trip

I created 120 asymmetric splats along a small helix, with nonuniform scales, Y rotations, opacity 0.85, color and nonzero degree-one SH terms inside a degree-three property layout. Four deliberate contaminants made cleanup testable. This is a numerical control, not a scan-quality example.

The File SOP read the standard 3DGS input. Bake GSplats converted it to `Cd`, linear `scale`, xyzw `orient`, `restorient`, `GS_Alpha` and three 16-component `GS_SPH_*` attributes. I enabled SPH coefficients and disabled color linearization for this bounded web round trip. These choices preserve the incoming learned color convention; they are not a universal Houdini display setting.

| Cleanup stage | Rule on this fixture | Remaining points |
|---|---|---|
| Input | 120 intended splats plus four contaminants | 124 |
| Crop | X/Z inside ±0.75; Y between −0.1 and 1.5 | 123 |
| Alpha | Remove GS_Alpha below 0.05 | 122 |
| Scale | Remove a splat with any linear scale component above 0.1 | 121 |
| Neighbors | At least three points, including self, within radius 0.25 | 120 |

Those thresholds are fixture-specific. Applying them indiscriminately to a real scan could destroy valid detail.

The inverse Python SOP restored scalar `f_dc_0..2`, `f_rest_0..44`, opacity as a logit, scale as logarithms and rotation as wxyz. It took the DC and higher-order terms from the native `GS_SPH_*` tuples. The helper rejects an orientation change relative to `restorient`, rather than pretending it can export arbitrarily transformed SH correctly.

For opacity and scale, the relevant inverse operations are:

```python
opacity = math.log(alpha / (1.0 - alpha))
log_scale = [math.log(component) for component in linear_scale]
wxyz = [orient[3], orient[0], orient[1], orient[2]]
```

The full helper clamps numerical endpoints and is in the package. After native serialization and independent parsing, the maximum scalar difference from the original 120 intended splats was **2.384e-7**. That verifies this cleanup-only round trip, not general transform, gamut or SH-editing behavior.

## Two different export problems

Saving the baked native attributes directly produced `GS_Alpha`, `orient1..4`, `scale1..3` and `GS_SPH_*` properties. A web reader expecting `opacity`, `rot_0..3`, `scale_0..2` and `f_rest_*` does not recover the same splat representation from those names.

There was a separate byte-order issue. The native File SOP wrote **binary_big_endian** PLY. The pinned Radix core reader could decode it, but Spark 2.3.1 returned `Unsupported PLY format: binary_big_endian`. Houdini's `gply -a` generated ASCII, which this Radix core reader rejected too. Neither failure means the file was absent or the native cook failed.

For Spark, I made a little-endian derivative by swapping each scalar field's byte order and changing the format declaration. Independent parsing verified **every value and property name exactly equal** before and after that adaptation. It changes no axis, scale, color or coefficient. The native binary originals remain available.

![Raw Houdini attribute export appears as tiny points, while the inverse schema and SPZ derivative render the controlled splats from the same browser camera](https://digitalgarden.si/marketing/research/houdini-gsplats-20261102/native-schema.png)

The first panel uses an endian-adapted native-attribute file to isolate the schema problem. The middle and right panels use the cleaned 120-splat representation. A loader returning a point count alone did not prove a correct image.

## One plant through the official Karma recipe

The owned source contains a cylinder stem, five double-sided mesh leaves, a spherical flower, simple materials and a dome light. The recipe created a 24-camera Fibonacci sphere. I used **384 × 384** images, increased the camera offset after a clipped-framing pilot, and limited native render concurrency to one job.

The useful graph was:

1. `IN_Stage` loads the source USD.
2. `Setup_Camera_Array` and the Karma render settings define views and AOVs.
3. TOP rendering writes multipart EXRs containing C/RGBA, albedo, N and depth.
4. `ML Preprocess GSplats` writes camera metadata, the COLMAP-like dataset and a 12,000-point initialization.
5. `ML Train GSplats` runs the bounded fit and exports PLY and USD.

I kept the setup failures. The headless recipe wrapper tried to access a network-editor UI after creating its nodes. Mixed path separators then coincided with 24 failed file-copy tasks; normalizing the experiment's base directory to forward slashes cleared those failures. A hyphen in `owned-plant` later caused the native USD prim export to fail after writing the trained PLY. Using `owned_plant` as the export name fixed the final export. With scripted linked parameters, `follow_parm_reference=False` was needed to keep that change from also renaming the dataset.

The successful final cook reused the rendered dataset and installed environments and took **54.675 seconds**. That includes cached graph work, launching, training, validation and export; it is one observation, not an end-to-end installation time or repeated training benchmark.

The final native USD reopened as `ParticleField3DGaussianSplat` with **12,000 positions**. Separate portable copies of the source recipe scene and the 120-point round-trip scene were saved and reopened. A real Karma XPU render of the exported USD also completed.

![Two native trainer validation views show the Karma source and trained splats; aggregate PSNR and SSIM are labeled with the shared initial geometry limitation](https://digitalgarden.si/marketing/research/houdini-gsplats-20261102/validation-views.png)

The trainer reported **31.303 dB PSNR and 0.97446 SSIM** over six validation views after 1,200 steps. Eighteen views supplied photometric training, but the initial depth point cloud used the complete 24-camera array. These are not fully unseen geometry tests, and the black background makes aggregate image scores easier than a foreground-only quality judgment. Leaf color and overlap differences are visible.

The export has the **SH3 property layout**, while the final log says the active trained degree was **one**: higher bands had not been trained by this 1,200-step schedule. Counting 45 `f_rest_*` properties is not proof that all those bands contain learned detail.

## PLY to SPZ, with measured losses

I used the MIT-licensed **@sparkjsdev/radix 0.0.1 core bundle**, pinned locally, to write SPZ v3 with SH enabled. I did not install or test GSOPs. The converter retained the splat count and SH3 layout on decoding the SPZ back.

| Asset | Standard PLY bytes used by Spark | SPZ v3 bytes | Splats in both |
|---|---|---|---|
| Cleaned numerical control | 31,432 | 1,328 | 120 |
| Karma-trained plant | 3,121,605 | 235,748 | 12,000 |

For the plant, SPZ decoding changed position components by at most **0.0001221 scene units**, scale by at most **3.175% relative**, quaternion angle by at most **0.00805 radians**, and SH coefficients by at most **0.003922** in the converter's decoded representation. Opacity matched that representation in this sample. These are quantization observations, not a universal bound for SPZ or proof of perceptual equivalence.

The Radix intermediate also has its own packing, so those error figures isolate the decoded intermediate-to-SPZ step rather than proving float-exact retention from the original PLY. The independent native scalar check above covers a different boundary.

![Measured PLY and SPZ file sizes for the 120-splat control and the 12000-splat trained plant](https://digitalgarden.si/marketing/research/houdini-gsplats-20261102/payloads.png)

## Browser acceptance

The formal browser run had **12 fresh-profile starts**: four asset/format cases, three repeats each. Chrome used Spark 2.3.1 with Three.js WebGLRenderer at 960 × 640, DPR 1. After 90 settling frames, each trial recorded 120 RAF intervals: **1,440 intervals** in total. Houdini training and Ollama inference were not running during these trials.

| File | Median fetch/decode/setup | Median first GPU submission completion | Decoded count |
|---|---|---|---|
| Control PLY | 22.8 ms | 281.4 ms | 120 |
| Control SPZ | 25.8 ms | 283.4 ms | 120 |
| Plant PLY | 35.7 ms | 285.6 ms | 12,000 |
| Plant SPZ | 32.7 ms | 283.9 ms | 12,000 |

These are localhost observations from three trials per case, not network delivery measurements. First GPU submission completion is a `gl.finish()` boundary before the settling period; it does not certify the first complete scene or physical display presentation. All trials stayed visible and had no captured page errors. The median RAF interval was 16.7 ms in every case, at this desktop's roughly 60 Hz ceiling. That does not rank renderer throughput.

The actual SPZ display used the normal Spark API:

```javascript
const spark = new SparkRenderer({ renderer, enableLod: false });
scene.add(spark);
const splat = new SplatMesh({ url: '/assets/trained-plant.spz', lod: false });
await splat.initialized;
scene.add(splat);
```

At the same static browser camera, plant PLY versus SPZ had foreground-union RGB RMSE **1.100 on an 8-bit scale**, over 36,843 pixels. The mask includes pixels where either image differs from its background by more than three channel values. This is a limited image-difference check, not an assessment of all viewpoints.

For the native/browser comparison figure, I first corrected my native EXR preview to use the **same OCIO configuration and scene_linear → sRGB - Display conversion as training**. A simple sRGB transfer without the configured working-space conversion had been misleading. The corrected preview and original failed preview are retained. Remaining color and rendering differences are visible in both the trainer comparisons and cross-renderer views; I have not established a single universal correction for them.

## FAQ

**Can Houdini 22 export a standard Gaussian-splat PLY?** The training TOP did so here. For edited native SOP attributes, I needed an explicit inverse schema conversion; the native File SOP's big-endian output also needed an adapter for this Spark version.

**Can I turn a Karma scene into a web splat?** Yes, this small synthetic plant completed the official recipe and displayed in PLY and SPZ. It does not establish scan quality, dynamic animation support or suitability for every garden asset.

**Is SPZ lossless?** No. This plant was much smaller on disk, with measured scale, position, rotation and SH differences. Only two static browser views and one formal camera were inspected.

**Does a point count prove the export worked?** No. The raw native schema could be decoded as a tiny point cloud after endian adaptation, while the intended Gaussian sizes and opacity were not represented correctly.

**Is SH3 fully trained here?** No. The file stores the SH3 layout, but only degree one was active at the end of this short fit.

## Data and reproduction

The [reproduction package](https://digitalgarden.si/marketing/research/houdini-gsplats-20261102/houdini-gsplats-study-20261102.zip), [analysis JSON](https://digitalgarden.si/marketing/research/houdini-gsplats-20261102/analysis.json), [browser CSV](https://digitalgarden.si/marketing/research/houdini-gsplats-20261102/browser-results.csv) and [README](https://digitalgarden.si/marketing/research/houdini-gsplats-20261102/README.md) contain the actual outputs and limits. Model installation caches and browser profiles are excluded.

Related: [Gaussian splats in Three.js](https://digitalgarden.si/blog/gaussian-splats-threejs/), [Spark Radix desktop tests](https://digitalgarden.si/blog/spark-radix-vs-spark-webgpu-benchmark/), [Copernicus baking to GLB](https://digitalgarden.si/blog/houdini-copernicus-bake-glb-threejs/) and [local Ollama VEX tests in Houdini](https://digitalgarden.si/blog/local-llm-houdini-ollama-vex/).
