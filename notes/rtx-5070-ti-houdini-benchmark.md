# RTX 5070 Ti in Houdini 22: Karma, Vellum and a Small Dense Pyro Test

> Karma CPU, XPU GPU and hybrid rendering, Vellum OpenCL devices and a small dense Pyro case tested on an RTX 5070 Ti in Houdini 22.0.429.

- Author: Alicia Lu
- Published: 2026-10-26
- Canonical HTML: https://digitalgarden.si/blog/rtx-5070-ti-houdini-benchmark/

For the same 960×540 render job, Karma XPU on the RTX 5070 Ti completed in a median 8.97 seconds including process startup, versus 17.17 seconds for Karma CPU. Enabling the CPU alongside the GPU increased the median to 9.62 seconds in this particular scene. Vellum's GPU run had the lower median after initialization, while a very small dense Pyro case showed no useful GPU advantage.

Tested 26 October 2026. Hardware: Ryzen 9 9950X, 32 logical processors, 66,196,152,320 bytes installed RAM, RTX 5070 Ti with 16,303 MiB reported device memory, NVIDIA driver 617.42, Windows 11 Pro. Existing applications were preserved; this was an interactive workstation rather than a dedicated lab. Houdini build: 22.0.429. These are reproducible scene-specific measurements, not a GPU buying guide or a 16GB capacity test.

## One render scene, three device configurations

The self-authored USD contains 400 instances of a 4,096-triangle deformed prototype, eight reflective spheres, a floor, an area light and a dome. The prototype is shared: instance count does not multiply all geometry allocations as if every object were unique. A companion HIP loads the relative USD; it was saved and reopened before testing.

All jobs use the same camera, 960×540 output, 64 primary samples, path-traced mode, seed 23, no denoising and the same materials and lighting. The three configurations are Karma CPU, XPU with the Embree device explicitly disabled, and XPU with its default GPU/CPU combination. Each uses three fresh husk processes, reversing mode order on the second repeat. The source and complete command lines are retained.

| Mode | Whole-process seconds: median (range) | XPU render-phase seconds |
| --- | --- | --- |
| cpu | 17.17 (16.92–17.27) | different engine; not compared as XPU phase |
| xpu-gpu | 8.97 (8.94–10.21) | 0.931 (0.926–0.959) |
| xpu-hybrid | 9.62 (9.49–10.70) | 1.424 (1.414–1.426) |

![Houdini timings with full observed ranges](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/houdini-timings.png)

The whole-process number includes startup, USD loading, shader/device preparation and file output. XPU's reported render phase is much shorter than that total, so quoting it alone would exaggerate the improvement a user sees when launching this small job. The approximately 1.91× whole-job ratio is a comparison of these declared jobs, not an equal-noise or equal-visual-quality score: CPU and XPU are different engines and their rendered pixels are not identical.

The EXR render statistics confirm the actual devices, beyond the requested environment values. Every GPU-only run records one OptiX device named NVIDIA GeForce RTX 5070 Ti, contributing all 64 samples. Every hybrid run records 56 GPU samples and eight Embree CPU samples. There are no reported device errors. SideFX documents the default combined behavior and the device-disable switches in its [Karma XPU documentation](https://www.sidefx.com/docs/houdini/solaris/karma_xpu.html).

![CPU inspection preview](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/karma-cpu.png)

![GPU-only inspection preview](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/karma-xpu-gpu.png)

The previews apply the same exposure 1/64, per-channel Reinhard transform and gamma 1/2.2. They are inspection views; the original HDR EXRs remain unchanged. No denoising was applied to either.

## Vellum: compare OpenCL devices explicitly

The cloth is a 64×64 grid with its first row pinned, two substeps and sixty frames at 24 fps. Source setup, checkpoint statistics and checkpoint writes are outside the summed per-frame cook timings. Exporting/readback of the SOP result remains part of the cook path. Checkpoints at frames 1, 30 and 60 are included as BGEO files.

This is OpenCL GPU versus OpenCL CPU, not an invented general OpenCL-off Vellum switch. The CPU device is the Ryzen 9 9950X exposed by Houdini's bundled Intel OpenCL platform. Child-only HOM selection is applied before cooking and after loading, then read back alongside the matching device inventory. A versioned temporary preference directory isolates the experiment, without changing the user's Houdini settings.

| Vellum device | 60-frame cook seconds | First-frame seconds | Median later-frame seconds |
| --- | --- | --- | --- |
| gpu | 7.22 (6.87–19.60) | 0.661 (0.645–8.726) | 0.1017 (0.0970–0.1044) |
| cpu | 9.24 (9.03–10.06) | 0.684 (0.675–1.298) | 0.1296 (0.1275–0.1302) |

The GPU median is lower, but its first recorded run includes a large initialization/kernel-cache cost. Its total spans 6.87–19.60 seconds. The range is retained rather than dropping the slow run. Driver and operating-system caches were not reset, and pilots had already exercised some code paths; this is not a cold-boot benchmark.

![Actual Vellum BGEO checkpoints, Y up](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/vellum-checkpoints.png)

The cloth moves from its initial plane to a hanging configuration. At frame sixty, CPU/GPU point positions differ by a maximum 0.000682 and RMS 0.000193 scene units. Small numeric differences are expected; preserving movement and the pinned setup matters more than forcing bit-identical hashes. This test includes no grains or collision acceptance and does not measure GUI viewport FPS. SideFX describes the solver's OpenCL-related controls in the [Vellum Solver reference](https://www.sidefx.com/docs/houdini/nodes/dop/vellumsolver.html).

## Dense Pyro: the small case does not establish a GPU win

The test deliberately identifies its solver: classic dense pyrosolver::2.0 DOP, a Smoke Object, a fixed 3×4×3 domain and voxel size 0.075. Its exported volumes are 40×53×40, or 84,800 voxels per scalar field. Density and temperature are sourced into the same domain for forty frames. This is not a sparse SOP Pyro benchmark.

| Classic dense Pyro | 40-frame cook seconds | Median later-frame seconds |
| --- | --- | --- |
| opencl | 1.179 (1.176–4.838) | 0.0199 (0.0193–0.0202) |
| cpu | 1.061 (1.048–1.120) | 0.0198 (0.0195–0.0199) |

The CPU cook-sum median is 1.061 seconds; OpenCL is 1.179 seconds. Later-frame medians are both close to 0.020 seconds, and an initialization outlier makes the first OpenCL run much longer. This fixture is too small to support a GPU-throughput conclusion. SideFX explicitly discusses transfer overhead and non-GPU operations in the [classic Pyro Solver reference](https://www.sidefx.com/docs/houdini/nodes/dop/pyrosolver.html).

![Density projections from actual CPU/OpenCL checkpoints](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/pyro-checkpoints.png)

The output is not just a successful cook: the saved volumes contain nonzero density, changing hashes and a rising plume. Frame one is the initialized state before first sourcing. By the final checkpoint, smoke reaches the fixed top boundary. This bounded domain is useful for repeatable timing, but it is not a production smoke-quality comparison or a high-resolution scaling result. The sparse solver's dense/OpenCL restrictions are a separate issue described by [SideFX](https://www.sidefx.com/docs/houdini/nodes/dop/pyrosolver_sparse.html).

## Acceptance failures were excluded

Several setup failures were caught before accepting the numbers. A RenderVar initially reported an invalid color output even though husk returned zero. A later version accidentally marked beauty as Cryptomatte; the XPU image and EXR metadata exposed it. Vellum's initial CPU request was also overwritten to GPU by startup configuration. These pilots remain in excluded folders, separate from the final samples.

Final acceptance checks logs, AOV type, output dimensions, finite/nonempty HDR pixels, actual XPU devices and sample contributions, OpenCL configuration readback, node errors/warnings, reopened HIP files and simulation checkpoints. A return code alone is insufficient.

## What the memory numbers do and do not show

The 500 ms nvidia-smi telemetry peaks at 3,358 MiB across the accepted Karma runs, including ordinary desktop allocations. Vellum and Pyro have separate sampled telemetry. Those are whole-device sampled values, not exact per-process allocations. The scene is heavily instanced and small; it says nothing about the maximum scene that fits into 16GB. No deliberate OOM, unrelated DCC shutdown, driver change or overclock change was performed.

The reproduction material includes the HIP/USD builders, prepared scenes, raw EXRs, BGEO checkpoints, commands, device evidence, CSV and plotting scripts. Larger unique geometry/textures, grains, modern sparse Pyro, long interactive viewport paths and physical-phone garden performance remain outside this measured scope. The three independent workloads should not be averaged into a single hardware score.

## Data and reproduction files

[Browser lab ZIP](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/browser-lab.zip) and [Houdini lab ZIP](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/houdini-lab.zip) are the two parts of this dated study. Extract both into the same folder for the complete source, prepared fixtures and raw results. Each is below 24 MiB; their [manifest and SHA-256 hashes](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/reproduction-manifest.json) are public. Native runtimes and the encoder are not bundled. Public HDR copies omit workstation identity metadata while retaining exactly identical pixel arrays; timing and geometry data are unchanged.

[All 21 native process and cook samples](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/native-samples.csv) · [Native summary and device contributions](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/native-summary.json).

## Related field notes

- [Same-plant Houdini and Geometry Nodes export](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/)
- [ML volume upres and training](https://digitalgarden.si/blog/houdini-ml-volume-upres-pyro/)
- [Three.js rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/)

AI-assisted disclosure: Codex assisted with the experiment implementation, execution, analysis and writing. The measurements, failures and limits are actual dated workstation results, not model estimates.
