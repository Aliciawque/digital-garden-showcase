# Spark Radix vs Spark 2.3.1: What a 5M-Splat Desktop Test Shows

> Spark Radix and Spark 2.3.1 tested at 1080p on an RTX 5070 Ti: 54 trials, WebGL2 sort delay, real draw calls, format failures and migration limits.

- Author: Alicia Lu
- Published: 2026-10-31
- Canonical HTML: https://digitalgarden.si/blog/spark-radix-vs-spark-webgpu-benchmark/

*AI-assisted disclosure: Codex helped construct the fixture, run the browser experiment, check the data and write this article. The measurements, excluded pilots, failures and untested boundaries are retained.*

**I would keep Radix as a separate WebGPU experiment rather than migrate my garden for a speed claim.** On October 31, 2026, I ran Radix 0.0.1 and Spark 2.3.1 on one Windows desktop with an RTX 5070 Ti. All three rendering paths stayed near the browser's 60 Hz presentation ceiling, including at five million source splats. That result cannot establish an unbounded speed ratio.

The more useful difference was sorting: Radix's WebGL2 fallback continued presenting frames at about 60 FPS while its applied sort order updated less often. Its five-million-splat fast-orbit control measured about 58 ms from starting a sort to applying the result. I also checked the actual graphics calls, format failures, opaque/transparent mesh integration and one raycast interaction.

![The owned 5M-splat SH0 flower-bed fixture rendered by Radix WebGPU, with an opaque red cube](https://digitalgarden.si/marketing/research/radix-20261031/hero.png)

*Actual browser capture, not a photographic scan. Fixture and figures by Alicia, CC BY 4.0.*

## What I actually tested

This is an isolated integration study for [Digital Garden](https://digitalgarden.si/), not a replacement of its live renderer. The asset is an owned, deterministic **synthetic flower-bed stress fixture**, with 36 green beds and pink, yellow and purple flower shapes. It is not a photographic garden capture. SH degree is zero, so it cannot test higher-order view-dependent appearance.

The same SPZ v3 file at each size reached every renderer: 500,000 / 2,000,000 / 5,000,000 splats, about 6.44 / 25.76 / 64.39 MB. A normal opaque Three.js cube shares the scene. I checked the decoded counts rather than accepting an empty canvas's FPS.

| Component | Tested setting |
| --- | --- |
| Workstation | Ryzen 9 9950X, approximately 64 GiB RAM, RTX 5070 Ti 16 GB, driver 617.42 |
| OS and browser | Windows 11 Pro 10.0.26300, embedded Chrome 156.0.0.0 |
| Libraries | Three.js 0.186.1, Radix 0.0.1, Spark 2.3.1 |
| Canvas | 1920 × 1080 drawing buffer, renderer pixel ratio 1, antialias off |
| Radix paths | Explicit WebGPU and explicit webgpu-gl; active backend checked |
| Spark path | WebGLRenderer / WebGL2, dynamic LoD disabled |
| Shots | Fixed camera and one identical orbit per 15 seconds |
| Repetitions | Discard 5 seconds of warmup, then three 15-second measurements per configuration |

That makes 54 accepted trials, 810 measured seconds and 48,597 recorded RAF intervals. No trial became hidden. The machine retained its normal desktop environment, with incidental editorial work; this was not an exclusive thermal or device-isolation test. The order was fixed, not randomized.

I changed one Radix integration setting before freezing the measured harness: **directCanvas:false**. With the default direct canvas path, the ordinary MeshBasicMaterial and clear color differed in my pilot. Using Three.js's output path aligned the ordinary scene's color handling. That adds an output pass, so this is not a reproduction of the official default-settings benchmark.

Both renderers use radial sorting here. Radix uses its default EWA projection and 24-bit sort. I did not tune all footprint/culling parameters to prove equal image quality. Matching asset, camera and resolution is necessary, but it does not by itself make two rasterizers equivalent.

## The FPS result has a ceiling

The orbit results below pool the raw samples from three repetitions. RAF is the time between browser animation callbacks. CPU time wraps the synchronous renderer.render call and excludes work on a worker and GPU completion.

| Source splats | Path | RAF mean / p95 ms | CPU mean / p95 ms | Hitches > 2× median |
| --- | --- | --- | --- | --- |
| 0.5M | Radix WebGPU | 16.67 / 16.80 | 0.271 / 0.400 | 0 |
| 0.5M | Radix WebGL2 | 16.67 / 16.80 | 0.816 / 1.100 | 0 |
| 0.5M | Spark 2.3.1 WebGL2 | 16.67 / 16.80 | 0.154 / 0.300 | 0 |
| 2M | Radix WebGPU | 16.67 / 16.80 | 0.280 / 0.400 | 0 |
| 2M | Radix WebGL2 | 16.67 / 16.80 | 0.860 / 1.600 | 0 |
| 2M | Spark 2.3.1 WebGL2 | 16.67 / 16.80 | 0.219 / 0.300 | 1 |
| 5M | Radix WebGPU | 16.67 / 16.80 | 0.355 / 0.500 | 0 |
| 5M | Radix WebGL2 | 16.67 / 16.80 | 0.677 / 1.700 | 0 |
| 5M | Spark 2.3.1 WebGL2 | 16.67 / 16.80 | 0.221 / 0.400 | 0 |

All the orbit p95 values round to 16.80 ms. The full suite retained two intervals above the strict hitch threshold, including a 33.4 ms fixed-shot Radix WebGPU interval at five million splats. Those outliers remain in the report; I did not remove them to improve the graph.

The CPU call costs differ, but none is a complete renderer cost. A shorter Spark main-thread call does not count its worker's sorting time. Likewise, WebGPU dispatch submission can return before GPU execution. I would not turn these columns into a headline “x times faster” comparison.

Separate timestamp diagnostics add context. At five million splats during the orbit, Radix WebGPU reported mean projection / sort / draw phases of about 0.36 / 1.33 / 7.14 ms. Spark's separate WebGL render query averaged about 5.94 ms. These probes use different backend-specific timing paths and do not include the same asynchronous work. The Radix draw phase also includes the normal scene/output work. They are useful receipts, not a matched total-latency ranking.

![All three paths reach the 60 Hz presentation ceiling during the identical orbit](https://digitalgarden.si/marketing/research/radix-20261031/raf.png)

![Synchronous render-call CPU time; worker execution and GPU completion are excluded](https://digitalgarden.si/marketing/research/radix-20261031/cpu.png)

## A 60 FPS fallback can still show an older order

Radix's [sorting documentation](https://github.com/sparkjsdev/radix/blob/b831cc5e98b52a83b6f0ac6ba5fc50b5ac883be1/docs/sorting.md) distinguishes the same-frame GPU sort from the WebGL2 key pass, readback and worker sort. I checked worker messages and successive applied-order versions in my run.

| Source splats | Applied-order interval mean / p95 ms | Worker round-trip mean / p95 ms | Key pass + readback mean / p95 ms |
| --- | --- | --- | --- |
| 0.5M | 16.68 / 16.80 | 2.78 / 5.90 | 4.78 / 6.30 |
| 2M | 33.40 / 33.40 | 11.74 / 13.10 | 10.53 / 13.50 |
| 5M | 55.45 / 66.70 | 27.29 / 29.20 | 21.70 / 29.60 |

The stationary shot can reuse an order. The moving camera produces new ones, but the five-million-splat fallback did not deliver a new order every presented frame. Its roughly 55 ms order interval and 16.67 ms RAF interval describe different parts of the pipeline.

I then recorded a separate six-second fast orbit: two turns, three seconds per turn, at five million splats. Wrapping the view's public sort/apply methods measured 89 completed Radix WebGL2 sorts. Start-to-apply delay averaged **57.88 ms**, with **64.30 ms p95** and **79.90 ms maximum**. The camera moved an average 0.87 world units between those two events.

The three clips are 1080p VP9 at about 15 fps and fully decoded with ffmpeg. Recording overhead is included in that control and excluded from the main comparison. The flower-bed fixture stayed recognizable in the inspected frames. I did not measure a flicker rate or establish that photographic edges and transparent foliage would look acceptable.

![Same-camera 5M flower beds rendered by Radix WebGPU, Radix WebGL2 and Spark WebGL2](https://digitalgarden.si/marketing/research/radix-20261031/comparison.png)

## One splat draw is real, and Spark does it too

I captured application calls to the graphics APIs after settling the scene, using one and then three objects with 500,000 splats each. Radix WebGPU issued one **drawIndexedIndirect** for the combined splat collection in both cases. Three objects added three projection dispatches, but not three splat draws. Radix WebGL2 similarly issued one instanced splat draw.

Spark also issued one **drawElementsInstanced** for the combined 500,000 / 1,500,000 splats. Therefore batching overlapping objects is useful, but this control does not establish it as a Radix-only advantage.

Whole-scene draw totals were 3 for Radix WebGPU and 2 for Spark: cube, splats and, for the controlled Radix path, the output pass. Radix WebGL2 added fragment passes and reported 7 / 8 calls in these captures. “One splat draw” does not mean “one call for the entire application.” These are API-boundary captures, not a Spector.js or complete hardware trace.

## Format and integration checks changed the migration question

SPZ v3 worked throughout the measured matrix. Additional 500,000-splat files were written from the same source data:

| Format | Radix 0.0.1 WebGPU | Spark 2.3.1 WebGL2 |
| --- | --- | --- |
| PLY | 500,000 decoded and drawn | 500,000 decoded and drawn |
| SPZ v4 | 500,000 decoded and drawn | Failed: Invalid gzip header |
| RDX | 500,000 decoded and drawn | Failed: Unknown file type |

Those failures belong to the pinned versions and files. I did not separately exercise SPLAT, KSPLAT, compressed PLY, RAD or the SOG/LCC example adapters. Core formats and example importers should not be conflated.

The opaque cube rendered with the expected front/back occlusion in all three paths. A transparent cube with opacity 0.45 and depthWrite=false also rendered, but its composition differed. Sorting all Gaussian objects together does not promise exact interleaving with arbitrary transparent Three.js meshes.

The stills are similar rather than pixel-identical. The five-million-splat Radix/Spark comparison had approximately 1.25/255 whole-image RGB mean absolute error, or 4.07/255 over the foreground union. This is a descriptive difference; the large shared background must not be mistaken for a quality pass.

A central Three.js Raycaster ray hit the ordinary cube on every path. It returned four Spark splat hits and zero Radix splat hits in this setup. Radix offers CPU-picking examples, but I did not port one. A working render is not a drop-in replacement for Spark's per-Gaussian picking API.

![Radix WebGPU controls with the cube absent, opaque and transparent](https://digitalgarden.si/marketing/research/radix-20261031/mesh-radix-webgpu.png)

![Spark WebGL2 controls with the cube absent, opaque and transparent](https://digitalgarden.si/marketing/research/radix-20261031/mesh-spark-webgl.png)

## Installation and teardown belong in the result

A normal Windows npm installation of Radix 0.0.1 failed with EBADPLATFORM: the published package required macOS-only fsevents. I used npm pack with scripts disabled, retained the official tarball integrity, and imported its unchanged prebuilt modules. The runtime then worked on both backends. This is a package-install workaround, not a rendering speed optimization.

The harness also exposed a Spark teardown boundary. Disposing while asynchronous sorting was active produced unhandled Promise events; a focused transition control reproduced four, including a readbackDepth / driveSort “No target” stack. I kept those failures and added a separately verified quiesce-before-dispose runner. The accepted timing data and frozen original harness remain intact. The workaround uses exposed fields of pinned Spark 2.3.1 and needs rechecking for another version.

## Would I switch the garden?

I would continue evaluating Radix for a bounded WebGPU scene that benefits from compute sorting or TSL-generated splats. I would keep Spark where its existing LoD/paging, asset handling and picking workflow are needed. Radix's public examples implement some features outside the core, so a simplistic “Radix cannot do edits or picking” table would hide possible integration work.

For my garden, these 54 trials do not justify a migration for FPS. The next useful study would use a licensed photographic scene with nonzero SH, matched visual controls, a scene that actually exceeds the display budget, and the target devices. Phones, Safari, Firefox, headsets, higher-order SH, 10M splats, streaming, sustained thermals and production integration remain untested.

The [official Radix benchmark](https://sparkjs.dev/radix/bench-report.html) remains evidence from its own machine, scenes and settings. Its live report now discusses matched culling and frame stalls in detail. I have not copied an older draft's Apple M3 speed ratio into my results.

For the earlier integration route, see [Spark vs GaussianSplats3D vs Native Three.js](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/), [Gaussian splats in Three.js](https://digitalgarden.si/blog/gaussian-splats-threejs/) and the [Spirula garden sample](https://digitalgarden.si/blog/spirula-studio-garden-splat-threejs/). Those published experiments are separate from this synthetic Radix study.

## Download the reproducible study

[Reproduction ZIP](https://digitalgarden.si/marketing/research/radix-20261031/spark-radix-reproduction-20261031.zip) · [Size and SHA-256](https://digitalgarden.si/marketing/research/radix-20261031/reproduction-receipt.json) · [Raw-derived results JSON](https://digitalgarden.si/marketing/research/radix-20261031/analysis.json) · [CSV](https://digitalgarden.si/marketing/research/radix-20261031/results.csv) · [Integration controls](https://digitalgarden.si/marketing/research/radix-20261031/controls-analysis.json) · [Fixture hashes](https://digitalgarden.si/marketing/research/radix-20261031/fixture-manifest.json) · [Environment](https://digitalgarden.si/marketing/research/radix-20261031/ENVIRONMENT.md) · [Full test report](https://digitalgarden.si/marketing/research/radix-20261031/TEST-REPORT.md).

The 12,002,053-byte ZIP contains 160 files: the pinned harness and official package modules, all accepted raw timing and GPU observations, API/format/picking/lifecycle receipts, selected figures, three recordings and the 0.5M SPZ. Regenerate the omitted 2M/5M and additional-format fixture binaries with the included generator. Original image/recording-data JSON and most full-resolution captures remain local. The pinned Radix source maps are omitted; they can produce an optional Vite debug warning. The unpacked package passed dependency installation, build, hash verification and a three-path browser pilot.

The safe runner waits for Spark 2.3.1 sorting before teardown; the frozen measured runner and its original failures are retained. Read the environment and test report before interpreting a rerun. Charts and the owned fixture are by Alicia, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/); upstream licences are retained.

[Three.js rendering field guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).
