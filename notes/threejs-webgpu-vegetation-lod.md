# 1,600 Trees in Three.js: Mesh LOD and 8/16-View Impostors Measured

> I tested 1,600 trees with mesh LOD and 8/16-view impostors in Three.js WebGPU and WebGL2, recording geometry, frame timing and terrain residuals.

- Author: Alicia Lu
- Published: 2026-10-26
- Canonical HTML: https://digitalgarden.si/blog/threejs-webgpu-vegetation-lod/

Distance LOD reduced submitted triangles from 1,032,193 to 193,921 in this independent 1,600-tree fixture. Replacing distant trees with azimuth impostors reduced them to 96,247. The measured display-paced frame interval stayed near 16.7 ms across all four variants on this RTX 5070 Ti, so the experiment establishes a geometry reduction rather than an observed FPS improvement.

Tested 26 October 2026. Hardware: Ryzen 9 9950X, 32 logical processors, 66,196,152,320 bytes installed RAM, RTX 5070 Ti with 16,303 MiB reported device memory, NVIDIA driver 617.42, Windows 11 Pro. Existing applications were preserved; this was an interactive workstation rather than a dedicated lab. The browser is Chromium 156, with Three.js 0.186.1. WebGPURenderer was tested with its actual WebGPU backend and its WebGL2 fallback, at a fixed 960×540 buffer and DPR 1. This is an independent deterministic conifer field, not a production garden acceptance test.

## Four variants, one fixed camera

The 1,600 placements share the same irregular offsets, scales, yaw values and ground-height function. Full mesh uses a 16-segment trunk/crown prototype. Mesh LOD uses 16-, six- and three-segment versions at horizontal camera distances below 45, from 45 to 90, and beyond 90 scene units. This is a simple distance fixture, not the production garden's screen-space LOD policy.

Both impostor variants retain the same 133 near trees as full mesh and replace the other 1,467 with vertical, camera-facing alpha-tested cards. Each card selects the nearest azimuth slice using its instance yaw and camera direction. The eight-view atlas has 45-degree spacing; sixteen views have 22.5-degree spacing. Cells are 128×128, and the atlases are baked from the same source geometry in the browser with lighting.

The camera remains at (0,24,85), looking toward (0,2,0). There is no wind, shadow map, post-processing chain or transparency blending. Those choices make the cost comparison reproducible but also limit how much it says about a finished garden.

| Variant | Submitted triangles | Draw calls | Full / mid / low mesh instances | Cards |
| --- | --- | --- | --- | --- |
| full | 1,032,193 | 3 | 1600 / 0 / 0 | 0 |
| lod | 193,921 | 5 | 133 / 629 / 838 | 0 |
| impostor8 | 96,247 | 4 | 133 / 0 / 0 | 1467 |
| impostor16 | 96,247 | 4 | 133 / 0 / 0 | 1467 |

Counts include the ground and background triangle; both backends agree. The extra LOD buckets increase draw calls even as they remove triangles.

![Geometry and draw-call counts](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/vegetation-geometry.png)

## Fewer triangles did not change the displayed rate here

Each variant gets thirty warmup frames and 180 recorded frames per trial. Three trials reverse variant order on the second repeat. Four variants × three repeats × two backends gives 24 trials and 4,320 reported frame samples. Raw per-frame intervals remain in JSON.

| Variant | WebGPU RAF ms: median of trial medians | WebGL2 RAF ms: median of trial medians | WebGPU p99 RAF ms |
| --- | --- | --- | --- |
| full | 16.70 | 16.70 | 16.90 |
| lod | 16.70 | 16.70 | 16.80 |
| impostor8 | 16.70 | 16.70 | 16.80 |
| impostor16 | 16.70 | 16.70 | 16.80 |

All variants are display paced near sixty updates per second. This workload does not saturate this GPU enough to expose a useful FPS separation. Submitting fewer triangles still changes the geometry budget, but reporting a multiplicative FPS gain would not match the observation.

The lab separately records CPU submission plus a GPU-completion wait. WebGPU and WebGL2 have very different wait/IPC overhead in this browser, so those wall-time numbers are not pure GPU timestamps and should not be used to declare one backend faster. The complete samples and stage definitions are included for that reason.

## Impostors have a visible cost

![Full mesh reference](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/vegetation-webgpu-full.png)

![Sixteen-view impostor variant](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/vegetation-webgpu-impostor16.png)

The cards are not visually equivalent to full geometry. They use baked lighting, approximate the elevated camera with azimuth-only views, and lose depth and crown detail. Their different shading and silhouettes remain visible in the captures. This prevents treating the geometry reduction as equal-quality performance.

At four bytes per base-level RGBA8 texel, the eight-view 1024×128 atlas is 524,288 bytes and the sixteen-view 2048×128 atlas is 1,048,576 bytes, before mipmaps and allocation overhead. The sixteen-view atlas doubles this base payload while keeping the same card count and submitted triangles. A fixed camera cannot establish how much the additional views improve rotation popping; a moving-camera quality test is still needed.

## Anchoring needs the mesh, not only a formula

The field uses an analytic sloped height function and a 64×64-segment triangulated ground. Assigning each tree its analytic height gives zero error when compared with that same formula. That construction check does not prove exact contact with the finite ground mesh.

An independent downward raycast against the regenerated measured ground mesh finds a maximum residual 0.008791 and RMS 0.003316 scene units. A negative control that moves the whole batch to its center height has maximum error 1.694 and RMS 0.772. Per-instance height is substantially better, but exact contact requires sampling the actual terrain mesh or increasing its resolution. These numbers belong to the fixture, not the production terrain.

## A backend-specific readback trap

The first WebGPU atlas was vertically inverted because the readback path applied the WebGL vertical flip unconditionally. Inspecting the saved atlas revealed it. The final bake flips WebGL readback and preserves WebGPU's origin; both were rechecked visually. Pilots with incomplete framing, incorrect count fields or inverted atlases are excluded from the final summaries.

This is also why a benchmark needs its pictures. A geometry counter would still look impressive if every card sampled the wrong image.

## Reproduction and the next useful experiment

The local lab exposes visible controls for both backends and saves raw samples and images to evidence. The package pins the renderer version, includes deterministic fixture code and generated atlases, and contains the CSV summaries and chart sources. Three.js's [official WebGPURenderer reference](https://threejs.org/docs/pages/WebGPURenderer.html) describes its WebGL fallback option; the exact tested implementation is pinned in the package lock.

The measured outcome is a static cost/appearance baseline. It does not validate octahedral impostors, dynamic view blending, dithered LOD transitions, wind-deformed cards, distant-grass baking, shadows, production assets or physical phones. Those original broad brief items remain future work rather than being filled with invented values. For this desktop fixture, triangle reduction is real, the image tradeoff is visible, and no FPS improvement was established at the display limit.

## Data and reproduction files

[Browser lab ZIP](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/browser-lab.zip) and [Houdini lab ZIP](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/houdini-lab.zip) are the two parts of this dated study. Extract both into the same folder for the complete source, prepared fixtures and raw results. Each is below 24 MiB; their [manifest and SHA-256 hashes](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/reproduction-manifest.json) are public. Native runtimes and the encoder are not bundled. Public HDR copies omit workstation identity metadata while retaining exactly identical pixel arrays; timing and geometry data are unchanged.

[Vegetation summary](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/vegetation-summary.csv) · [Independent triangle-ground raycast](https://digitalgarden.si/marketing/research/desktop-benchmarks-20261026/anchoring-raycast.json).

## Related field notes

- [Uneven-terrain scatter](https://digitalgarden.si/blog/threejs-scatter-objects-uneven-terrain/)
- [WebGPU/WebGL2 plant comparison](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/)
- [Three.js rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/)

AI-assisted disclosure: Codex assisted with the experiment implementation, execution, analysis and writing. The measurements, failures and limits are actual dated workstation results, not model estimates.
