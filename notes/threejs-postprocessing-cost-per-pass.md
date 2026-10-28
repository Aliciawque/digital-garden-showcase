# Three.js Post-Processing: 192 Desktop Trials Beyond the FPS Counter

> 192 desktop trials of Three.js post-processing: GPU timings, bloom negative controls, N8AO quality, reflections and render resolution, with reproducible data.

- Author: Alicia Lu
- Published: 2026-10-28
- Canonical HTML: https://digitalgarden.si/blog/threejs-postprocessing-cost-per-pass/

*Measured and published on October 28, 2026.*

Turning on post-processing often left my browser at roughly 60 frames per second. It did not leave the rendering workload unchanged. Across three scenes, I ran 192 fresh-page trials covering N8AO, bloom, tone mapping, volumetric beams, reflections and render-buffer resolution. The most useful result was a negative control: bloom at zero intensity produced exactly the same Reliquary image as removing bloom, while retaining 16 extra draw calls.

The timing results also carried a warning. Repeated desktop runs sometimes varied more than the small effect I was trying to price. This article keeps those ranges visible. It offers reproducible measurements and a way to inspect a stack, rather than a universal table of milliseconds to add to any scene.

## What I actually tested

The starting point was [Dioramas](https://github.com/blendi-remade/dioramas/tree/868539fb5b64a843071a2247d5d8bacb706a07b1), pinned to commit 868539f. I adapted its Reliquary and Kōen pages to freeze the first hero camera and scene state. Reliquary provides reflective flooring and volumetric light; Kōen provides a textured bonsai and a very different bright background. A third, owned fixture places 1,600 instances of my Houdini tree in a separate WebGL scene. Its diagnostic vertex colors are retained.

The machine was an RTX 5070 Ti with 16,303 MiB reported memory, Ryzen 9 9950X, NVIDIA driver 617.42 and Windows build 26300.9711. Chromium 156 used ANGLE's D3D11 WebGL2 backend. Dependencies were three 0.186.1, postprocessing 6.39.5 and n8ao 2.0.1, pinned in the lockfile.

Browser layout stayed at 1440×900. Effective renderer DPR was 1, independently of Windows display scaling. I tested buffers at 1440×900, 2560×1440 and 3840×2160. The latter two are larger buffers on the same desktop, not tests on other displays or mobile devices. Their aspect ratio also differs from the first size, so this is an actual output-resolution change, not a perfectly isolated pixel-count experiment.

The production garden uses WebGPURenderer and TSL. This experiment uses WebGLRenderer and pmndrs postprocessing. Three.js documents that [WebGPURenderer can itself fall back to WebGL2](https://threejs.org/docs/pages/WebGPURenderer.html); that does not make the two renderer APIs interchangeable. I did not install this post stack into the live garden.

![Actual first-repeat Reliquary and Kōen renders, including bloom negative controls](https://digitalgarden.si/marketing/research/postprocessing-20261028/render-controls.jpg)

*Dioramas models and images: Dioramas (github.com/blendi-remade/dioramas), CC BY 4.0. These are fixed-camera adaptations. Upstream assets were AI-generated according to its license notice; this test reused them and made no fal generation calls.*

## “Cost per pass” needs a definition

My baseline is an HDR HalfFloat composer with a scene RenderPass and a linear ToneMappingEffect. It already includes scene shading and shadows. SMAA and Kōen's paper effect are excluded from every comparison; the dense plant capture consequently shows aliasing. I add one effect configuration at a time, then test a standardized full stack separately.

That full stack uses Medium full-resolution AO, bloom, AgX, vignette/noise and chromatic aberration. Reliquary also includes one volume beam and reflection resolution 0.5. This differs from the shipped Dioramas defaults: the upstream engine defaults to half-resolution AO. The explicit override makes the conditions comparable; it is not a claim about the original demo's performance.

Each independent reload warms 120 RAF renders and measures 480 more. An asynchronous [WebGL GPU timer query](https://registry.khronos.org/webgl/extensions/EXT_disjoint_timer_query_webgl2/) brackets rendering once every eight measured frames, producing 60 samples. I check query availability and reject disjoint timing. N8AO's own debug timer stays disabled to avoid nesting queries. Its accumulation setting remains false, so a static camera does not turn this into an accumulated-image reuse benchmark.

The three suites total **92,160 measured RAF frames and 11,520 accepted GPU samples**. They cover 64 conditions, with three reloads each. The second repeat reverses condition order and the third rotates it. One Kōen result failed to save locally and was rerun; its failure receipt is retained separately. All accepted runs have the expected sample counts, buffer dimensions, identical cameras within each scene/resolution, no hidden-tab measurement and no captured rendering error.

GPU elapsed time, CPU submission duration and RAF intervals remain separate columns. The tables report the median of three run medians, with the range of those medians. “p95” is the median of the three run p95 values. A delta is the difference from the corresponding scene/batch baseline; it is not a timer placed around an individual shader. Nor is a difference between two p95 values the p95 of a difference.

## The desktop result, including the uncertainty

At 1440×900, the baseline and full-stack whole-frame GPU times were:

| Scene | Baseline ms (trial range) | Full stack ms (trial range) | Median delta ms | Full-stack p95 ms |
| --- | --- | --- | --- | --- |
| Reliquary | 2.454 (0.412–2.654) | 4.372 (1.894–4.520) | +1.918 | 5.862 |
| Kōen | 2.335 (0.583–2.377) | 2.884 (2.554–2.916) | +0.549 | 4.215 |
| Owned plants | 0.266 (0.265–0.266) | 2.486 (2.454–2.767) | +2.220 | 3.820 |

![Whole-frame GPU times with trial ranges across three scenes](https://digitalgarden.si/marketing/research/postprocessing-20261028/core-gpu.png)

Reliquary illustrates why a single reload is insufficient: its baseline trial medians span 0.412–2.654 ms. The desktop was not clock-locked or isolated from normal system scheduling. I cannot attribute that spread to one cause, and I did not discard the slow runs. The longest recorded RAF interval across accepted conditions was 333.4 ms; the raw receipt remains in the download.

AgX in Reliquary measured 0.392 ms, with a 0.392–2.627 ms range; ACES measured 2.521 ms, with a 0.354–2.556 ms range. Those observations do not establish a portable speed ranking. Small negative baseline differences are also retained, rather than presented as proof that adding an effect accelerates rendering.

pmndrs can combine compatible effects into an EffectPass. Bloom adds its own work, while several display adjustments may share a pass. Changing the combination changes the program and scheduling context. For that reason, the full-stack measurement is useful in its own right; I do not add isolated deltas into a fabricated total.

## Bloom can do work without changing the image

In Reliquary, removing bloom produced 83 draw calls per measured frame. Enabling bloom produced 99; intensity zero still produced 99. The zero-intensity capture and the removed-bloom baseline have **zero different RGB pixels** at 1440×900. The work survives even when its visible contribution does not.

The corresponding GPU medians were 2.454 ms for baseline, 2.534 ms for bloom and 2.748 ms at zero intensity. Their ranges matter more than a tiny apparent ordering. The stable evidence is the unchanged image and persistent extra draws.

Kōen supplied another negative control. At 1440×900, this frozen view and bloom threshold of 1.25 produced an image identical to baseline. At 2560×1440 and 3840×2160, 839 and 903 pixels differed, respectively, so the exact-match result is resolution-specific. This does not show bloom is broken or always invisible in the scene. A bright background alone is not sufficient evidence of a useful bloom contribution. Before keeping an effect, inspect its actual output at representative views and sizes.

## AO quality, resolution and radius

The separate N8AO sweep uses Reliquary and its own baseline. Five quality presets are each tested at full and half internal resolution, plus Medium radius controls at 0.5× and 2×. Half resolution means half width and height, followed by the effect's reconstruction work; it is not a promise of one-quarter total frame time.

| N8AO preset | Full ms (trial range) | Half ms (trial range) |
| --- | --- | --- |
| Performance | 3.045 (2.844–3.452) | 2.743 (2.420–2.907) |
| Low | 3.018 (1.958–3.110) | 1.694 (1.124–2.966) |
| Medium | 2.902 (1.621–3.125) | 2.237 (1.709–2.759) |
| High | 3.022 (2.358–3.998) | 3.289 (0.679–3.307) |
| Ultra | 3.330 (3.167–3.725) | 2.000 (1.587–2.821) |

![N8AO quality and internal-resolution sweep with trial ranges](https://digitalgarden.si/marketing/research/postprocessing-20261028/ao-quality.png)

In the pinned [N8AO implementation](https://github.com/N8python/n8ao), presets alter sample and denoise settings; High and Ultra also change denoise radius relative to Medium. These are compound presets, not a controlled test of sample count alone. The CSV identifies every condition; its raw JSON contains the actual configuration. The radius controls are sensitivity checks at a fixed scene scale, not portable radius recommendations.

Pixel comparisons confirm that AO changes the images. They do not score temporal stability, disocclusion, edge quality or artist preference. A still frame can verify that an effect exists while missing the artifact that appears during camera movement. I would choose a candidate setting from these measurements and then test motion before shipping it.

## Reflections and volumes have different work shapes

Reliquary's reflection condition includes the mirrored scene render inside the same GPU query as the main frame. Draw calls rise from 83 to 139. A reflection scale of 0.5 uses a 720×450 target; 0.25 uses 360×225. Their whole-frame GPU medians were 2.516 and 2.815 ms. Both still submit the mirrored scene, so reducing target pixels does not remove that submission work.

The volume experiment keeps the upstream fixed 32-step raymarch. One beam measured 2.946 ms and four beams 3.561 ms. The four-beam control clones and offsets the original effect light while preserving direct scene lighting. It is a scaling test of the effect's light loop, not four newly lit physical fixtures. The same draw count can hide additional fragment work, just as extra draws can remain with zero visible bloom.

## Resolution is a separate budget

| Scene | Render buffer | Baseline GPU ms | Bloom GPU ms | Full-stack GPU ms |
| --- | --- | --- | --- | --- |
| Reliquary | 1440×900 | 2.454 | 2.534 | 4.372 |
| Reliquary | 2560×1440 | 2.883 | 3.096 | 6.669 |
| Reliquary | 3840×2160 | 3.594 | 3.711 | 10.200 |
| Kōen | 1440×900 | 2.335 | 2.404 | 2.884 |
| Kōen | 2560×1440 | 2.276 | 2.403 | 2.841 |
| Kōen | 3840×2160 | 2.503 | 2.714 | 6.335 |
| Owned plants | 1440×900 | 0.266 | 2.337 | 2.486 |
| Owned plants | 2560×1440 | 0.509 | 2.250 | 2.173 |
| Owned plants | 3840×2160 | 0.974 | 2.619 | 4.071 |

![Render-buffer scaling at three sizes on the same desktop](https://digitalgarden.si/marketing/research/postprocessing-20261028/resolution-gpu.png)

The 1440×900 points come from the core suite; larger sizes come from the resolution suite. Three repetitions are available for every point, with ranges in the chart and CSV. Treat the curves as scene-specific observations under this desktop session. They are not forecasts for a phone, a different GPU, or the live WebGPU garden.

For an integration decision, I would first remove effects that contribute nothing in the intended view, then inspect reflection target size and AO quality at the actual output resolution. The final check should include camera movement, image quality and a fresh full-stack measurement. A refresh-capped FPS display alone cannot tell which change reduced rendering work.

## Reproduction and scope

The accompanying source-and-results ZIP contains the pinned harness, unchanged MIT engine core, adapted scene pages, owned fixture, all 192 accepted JSON receipts, failure record, CSV, charts and six representative original PNGs. The complete image-comparison report and image hash inventory are included; the remaining full-size captures are retained locally. A download helper retrieves the larger upstream CC BY 4.0 assets at the exact commit and validates their hashes. Independent dependency installation, asset verification and a four-entry Vite build passed.

This is a warmed localhost desktop test. It does not measure network loading, cold shader compilation, exact VRAM, thermal soak, physical phones, another GPU or production integration. Object counts from renderer.info are not VRAM measurements. The original Notion brief and excluded pilots are preserved separately.

Codex implemented and ran the harness, checked the renders, analyzed the receipts and assisted this article for Alicia. The data includes failed setup attempts and timing uncertainty where relevant; generated prose is not a substitute for those receipts.


## Download the data

[Reproduction ZIP](https://digitalgarden.si/marketing/research/postprocessing-20261028/postprocessing-study-20261028.zip) · [Archive size and SHA-256](https://digitalgarden.si/marketing/research/postprocessing-20261028/archive-receipt.json) · [Summary CSV](https://digitalgarden.si/marketing/research/postprocessing-20261028/summary.csv) · [Summary JSON](https://digitalgarden.si/marketing/research/postprocessing-20261028/summary.json) · [Accepted trial inventory](https://digitalgarden.si/marketing/research/postprocessing-20261028/accepted-runs.json) · [Image comparisons](https://digitalgarden.si/marketing/research/postprocessing-20261028/image-checks.json).

The 8,764,655-byte archive contains all 192 timing receipts and six representative full-size captures. Its other 58 full-size captures remain local; the complete 64-image hash inventory and comparison report are included. Upstream models and images retain their CC BY 4.0 attribution and are fetched by the pinned download helper.

## Related field notes

- [Three.js rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/)
- [Shared grass and tree wind: 54 desktop trials](https://digitalgarden.si/blog/threejs-webgpu-grass-wind-field/)
- [1,600 trees: mesh LOD and impostors](https://digitalgarden.si/blog/threejs-webgpu-vegetation-lod/)
