# Houdini ML Volume Upres for Pyro: Pretrained Models and a Real Training Run

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A Houdini 22 Pyro cook log: pretrained 2x and 3x upres, CPU and DirectML inference, a small CUDA training run, cache sizes and field-resolution pitfalls.

- Author: Alicia Lu
- Published: 2026-10-19
- Canonical HTML: https://digitalgarden.si/blog/houdini-ml-volume-upres-pyro/

**Start with the pretrained ML Volume Upres nodes, then train a small model only after the data and inference path work.** On this workstation, the native billowy-smoke recipe produced valid 2× and 3× density volumes; the native training recipe also exported an ONNX model that could be run again through the same SOP. The interesting failures were mostly about field selection, environment setup and what a short training run can actually prove.

This is a cook log from Houdini 22.0.429 with a commercial license, OpenUSD 26.5, Windows 11, an AMD Ryzen 9 9950X, 64 GB RAM and an NVIDIA RTX 5070 Ti with 16 GB VRAM. The observed driver was 617.42. Training logs selected CUDA with PyTorch 2.9.0+cu130 in a separate recipe-managed Python environment. GPU inference below explicitly used DirectML where stated; selecting Automatic alone is not proof of which provider was chosen.

![Actual Houdini density volumes rendered side by side with one camera and one shared Blender volume shader: low resolution, pretrained 2x and pretrained 3x](https://digitalgarden.si/marketing/research/briefs-20261019/pyro-low-2x-3x.png)

Left to right: low resolution, pretrained 2×, pretrained 3×. These are the actual density VDBs at frame 12, rendered in Blender 5.2 with the same camera, lights and volume-shader density. This compares the volume output, not Karma/PyroLook renderer parity.

## What the node changes

ML Volume Upres applies a trained ONNX model to a native volume or VDB.[^1] It increases spatial resolution and sharpens density detail. The upstream simulation still determines the smoke's motion, shape and interaction with sources or colliders. It is not a replacement for the Pyro solver or a guarantee that a coarse simulation has the dynamics of a high-resolution one.

The distinction matters for the [Digital Garden](https://digitalgarden.si/virtual-garden/) pipeline. If I eventually use a volume element in a shot, I still need to art-direct its source, motion, density and lighting. A sharper cached field is one stage of that work. This is also a different job from image-to-mesh. The [ComfyUI 3D workflow](https://digitalgarden.si/blog/comfyui-3d-workflow/) cleans a generated prop and checks the GLB in Blender and Houdini; it does not upres a smoke cache.

SideFX's documentation also requires DOP-level permissions even though the inference node lives in SOPs. The documented supported license categories include FX, Apprentice, Indie and Education; Houdini Core is excluded.[^1] The local run used the licensed native application, not a Python package that merely resembles a volume operation.

## Reproduce the pretrained path

The native `sidefx::recipe::pyro::billowysmoke` recipe created the source, rasterization, Pyro solver, file-cache node, `ml_volumeupres_2x`, `ml_volumeupres_3x` and PyroLook nodes. Both ML nodes were present; the 2× branch feeds the default look path. The pretrained model paths resolve through Houdini's `shfs:` resources rather than an arbitrary downloaded model.

I set the solver's voxel size to 0.15 and cached frames 1–12 before inference. The first frame took about 1.54 s, followed by roughly 0.06–0.30 s per frame in this small fixture. Total low-resolution cache cook/write time was about 2.56 s. These figures include native geometry access and writing the compressed cache, not just solver time.

For each inference result, I connected a File SOP to the same cached frame. This prevents a live simulation recook from being mistaken for ML inference time.

| Frame-12 output | Density resolution | Voxel size | Compressed full-field cache |
| --- | --- | --- | --- |
| Low resolution | 33 × 17 × 32 | 0.15 | 97,957 bytes |
| Pretrained 2×, DirectML | 68 × 36 × 64 | 0.075 | 412,887 bytes |
| Pretrained 3×, DirectML | 102 × 54 × 96 | 0.05 | 1,164,333 bytes |

The dimensions are not exact multiples of the low-resolution active bounds. Sparse-grid resampling, padding and pruning can change those bounds. The voxel transform gives a more useful scale check than demanding `old_dimension × factor` on every active bounding-box dimension.

## CPU and GPU inference on the same input

| Model | Automatic | CPU | DirectML |
| --- | --- | --- | --- |
| Pretrained 2× | 2.582 s | 1.104 s | 0.638 s |
| Pretrained 3× | 2.155 s | 12.943 s | 1.990 s |

These are single sequential cook-and-write observations, not medians or isolated-kernel benchmarks. The first Automatic run includes model/provider warm-up. Another editor was open, and no allocator-level peak VRAM measurement was recorded. The table supports the narrow observation that DirectML successfully processed these inputs and the larger 3× CPU cook took longer here. It does not establish a universal acceleration ratio.

The node documentation lists Automatic, CPU, CUDA, DirectML on Windows and CoreML on macOS.[^1] This test explicitly exercised CPU and DirectML. CUDA was used by training; CUDA inference was not separately benchmarked, and the Automatic inference provider was not independently identified.

At this render size, both pretrained results show sharper folds than the coarse input, with additional small ridges in the 3× result. That is a visual observation about this frame. A short sequence, an alternate source and a render at the actual delivery size are needed before deciding whether the extra cache and cook cost is worthwhile for a shot.

## The field-resolution trap

The recipe's ML selection is `@name=density`. In the default result, temperature stayed at 0.15 voxel size while density became 0.075 for 2×. Velocity started at 0.30 and also stayed unchanged. Merely seeing a detailed density field does not prove that every companion field has been resampled.

I enabled **Naive Upsample Other Fields** and selected the fields using the actual primitive-group expression:

```text
@name=temperature @name=vel
```

Temperature then became approximately 0.075 and velocity approximately 0.15. The velocity field remained coarser than density because the operation scales its existing grid; it does not force every field onto the density transform. Flame, which was empty in this smoke fixture and not selected, retained its source transform.

A string such as `temperature vel` did not select those named primitives in this test. Check the result's field names and voxel transforms, rather than treating a checked parameter as proof that the intended fields changed. If a downstream operation requires identical transforms, explicitly resample to a reference grid and verify it.

## A real, deliberately small training run

The native **ML Train Volume Upres** recipe supplies a training branch and an inference branch.[^2] I used its own Pyro source, cached high-resolution frames at voxel size 0.075, and fed frames 8–12 to the training subnet.

| Setting | Local value |
| --- | --- |
| Scale factor | 2 |
| Training frames | 8–12, interval 1 |
| Tiles per frame | 2 |
| Tile size / input padding | 48 / 6 |
| Scatter seed | 1 |
| Channel multiplier | 1 |
| Batch size | 1 |
| Training schedule | Two partial cooks of two epochs; four epochs total |
| Training device | CUDA, as recorded by the recipe log |

The first end-to-end cook, including creation and population of the separate Python environment, took about 226.7 s. A second partial cook resumed the checkpoint and completed in 22.2 s. That second number is **not** the total setup-and-training time, and it includes TOP orchestration rather than isolating the GPU training kernel.

The recipe produced an ONNX graph of 20,304 bytes plus 210,816 bytes of external tensor data, alongside its metadata and checkpoint files. Preserve the external `.onnx.data` file with the graph. The presence of an `.onnx` filename by itself is not a complete model-delivery check.

The subsequent inference test loaded that exported model into ML Volume Upres and cooked the same cached low-resolution frame. The final recorded CPU and DirectML cook/write observations were about 0.591 s and 0.517 s, with no node errors or warnings. Both produced a valid density volume at approximately 0.075 voxel size.

## What the training run does not establish

Four epochs on ten source tiles are a functional smoke test. They demonstrate dataset export, native CUDA training, checkpoint continuation, ONNX export and independent inference. They do not show that the custom model beats the shipped model or generalizes to fire, mist, wind or collision-heavy smoke. The training log remains resumable; I did not label the model converged.

No model-specific blur or seam failure was established in this small run, so there is no invented before/after model-fix story. Before using a trained model for a shot, hold out frames or another simulation, compare against both the pretrained result and a genuine high-resolution solve, inspect temporal stability, and judge it under the final lighting.

For the comparison image I kept one common Blender shader unchanged. I did not perform a dedicated PyroLook/Karma retuning pass. A denser-looking render may need shader-density or exposure changes, but that decision should follow an actual shot comparison; it is not a mandatory numeric compensation derived from the upres factor.

## Deliverables and the next useful test

The [experiment archive](https://digitalgarden.si/marketing/research/briefs-20261019/pyro-upres-test.zip) contains the scripts, compact metrics and the small exported model with its dependencies. Full local simulation caches, Python environment and editable scene files remain in the experiment workspace rather than in the public web download.

Start with pretrained inference, get a reliable cache boundary, inspect every field, and only then train. If the source motion is wrong, fix the simulation. If only the visible density detail is too soft, the upres comparison becomes a meaningful next experiment. The [Houdini/Unreal test](https://digitalgarden.si/blog/houdini-unreal-usd/) and [OpenUSD beginner stage](https://digitalgarden.si/blog/openusd-pipeline-beginners/) cover downstream scene organization; neither is a claim that this smoke cache was shipped in the live browser garden.

## References

[^1]: SideFX, [ML Volume Upres](https://www.sidefx.com/docs/houdini/nodes/sop/ml_volumeupres.html), checked 19 October 2026.
[^2]: SideFX, [ML Train Volume Upres recipe](https://www.sidefx.com/docs/houdini/ml/train_solutions/ml_trainvolumeupres.html), checked 19 October 2026.
