# Three.js WebGPU Grass Wind: One Field, 54 Desktop Trials

> 54 desktop trials of shared TSL wind, 100k–1M grass blades, Houdini wind masks and chunk recycling in Three.js WebGPU and WebGL2.

- Author: Alicia Lu
- Published: 2026-10-27
- Canonical HTML: https://digitalgarden.si/blog/threejs-webgpu-grass-wind-field/

A shared TSL wind field can bend grass and Houdini-exported plants coherently on both Three.js WebGPU and its WebGL2 fallback. In this independent desktop meadow, 1,000,008 opaque grass blades produced median GPU query times of **1.51 ms on WebGPU and 1.22 ms on WebGL2**, while displayed frame intervals stayed near **16.7 ms**. Those are different measurements, and neither is a universal grass capacity limit.

Codex built the fixture and ran **54 trials**, **12,960 measured animation frames** and **1,620 GPU query samples** on my RTX 5070 Ti workstation. The experiment also checks root pinning, world-space wind continuity, chunk bounds, shadow behavior and native Houdini wind masks. It is a separate research scene for [Digital Garden](https://digitalgarden.si/), not an optimization already deployed to the live garden.

![Owned grass and Houdini plant fixtures using one wind field](https://digitalgarden.si/marketing/research/grass-wind-20261027/shared-wind-scene.png)

## The test scene and measurement boundary

The test used Three.js **0.186.1**, Windows 11 build **26300**, an AMD Ryzen 9 9950X, approximately 64 GB of system memory and an **RTX 5070 Ti, 16,303 MiB**, driver **617.42**. It ran in the Codex app's Chromium **156** browser, not a separately tested Chrome installation. The canvas rendered at **2560 × 1440**, DPR 1, with no MSAA. WebGL2 reported the NVIDIA adapter through ANGLE/D3D11; WebGPU reported NVIDIA/Blackwell.

Measurements were taken on **27 October 2026 in Asia/Shanghai**, corresponding to **26 October in Pacific time**. Each configuration had three repetitions, with configuration order rotated between repetitions. WebGPU and WebGL2 ran sequentially. Each trial warmed up for 45 frames, then recorded 240 RAF frames. A separate pass collected 30 serialized GPU timestamp queries, with a fresh animation frame between queries. GPU query resolution deliberately waits; those samples do not measure free-running throughput or end-to-end latency.

The blades are opaque, double-sided strips: four segments, ten vertices and eight triangles each. The static count sweep uses 36 grass cells, 18 low-detail plant instances and a ground plane. It has **no shadow maps, alpha-tested textures, LOD, collision, post effects or compute placement**. Shadows are tested separately. The simple bend does not recompute deformed normals and is not an inextensible beam simulation.

The camera sees the entire resident patch in the count sweep. A nominal count is rounded up evenly across cells, so 100k means 100,008 actual blades. All four count configurations report 39 renderer draw calls, including plants, ground and the output pass. One million blades submit about eight million triangles; this is a deliberately simple geometry workload.

## One field, shared roots, different bend weights

The wind object holds direction, speed, strength and a clock uniform. Grass and plants use the same function and the same world-space origin convention. Two noise octaves produce a bounded positive amplitude, so changing the direction rotates the push consistently. Roots remain fixed because the main bend weight is zero there.

The implementation below is extracted from the tested source. It uses TSL's vertex-stage nodes; it does not run a compute shader for sway. Imports, the noise texture and uniforms are supplied by `wind.mjs` in the reproduction project.

```javascript
export function field(mode){return Fn(([root])=>{
 if(mode==='off') return vec2(0);
 const scroll=wind.direction.mul(wind.clock.mul(wind.speed));
 const p=root.mul(.08).sub(scroll),q=root.mul(.4).sub(scroll.mul(2.3));
 const broad=mode==='texture'?texture(noiseTexture,p.mul(.08)).level(0).r.sub(.5).mul(2):mx_noise_float(p);
 const fine=mode==='texture'?texture(noiseTexture,q.mul(.08)).level(0).r.sub(.5).mul(2):mx_noise_float(q);
 const amplitude=broad.add(fine.mul(.35)).add(.8).clamp(.25,1.5).mul(wind.strength);
 return wind.direction.mul(amplitude);
});}
export function deformGrass(base,root,yawHeight,windAt){
 const h=base.y,weight=h.mul(h),c=yawHeight.x.cos(),s=yawHeight.x.sin();
 const local=vec3(base.x.mul(c),h.mul(yawHeight.y),base.x.mul(s));
 const push=windAt(root.xz).mul(yawHeight.y);
 return local.add(root).add(vec3(push.x.mul(weight),push.length().mul(weight).mul(-.2),push.y.mul(weight)));
}
```

Sampling at the root means all vertices in a blade evaluate wind at the same coordinate. **It does not mean the GPU evaluates the noise only once per blade.** This vertex shader can still evaluate the function for each vertex invocation. Sharing a uniform object is also different from storing a precomputed wind field.

The added downward offset makes the strip bow visually, but does not preserve its length exactly. That is an artistic deformation, not a physical material model. The trees use their exported main-bend mask, with a separate small leaf-flutter term. Six of the 18 instances are scaled shrub variants of the same owned tree geometry, not a second botanical asset.

![Recorded 360-degree direction sweep in the tested WebGPU scene](https://digitalgarden.si/marketing/research/grass-wind-20261027/wind-direction-small.gif)

The animation is a recorded programmatic direction sweep of the actual renderer. It uses 25,020 resident blades and 18 plants for visual inspection; it is not the million-blade performance trial. The local viewer also provides a direction slider.

## What the Houdini mask actually contains

The native fixture was generated and exported in **Houdini 22.0.429 Commercial**, then its HIP was reopened. The mesh contains **568 points and 400 triangles**. The GLB is **26,652 bytes** and passed Khronos validation with zero errors and zero warnings.

For this fixture only, the exported `COLOR_0` channels mean:

- **R:** `min(1, (localY / 5)^2)`, the main bend weight.
- **G:** leaf flutter weight, 0 on the trunk and 1 on leaves.
- **B:** branch index divided by 24, a repeatable phase.

The browser loaded all 568 color values. An independent binary GLB audit found a maximum R-versus-height mapping error below **8.54 × 10⁻⁸**, with zero weight at the base. An early fixture let a tall leaf's R value exceed 1; that pilot is retained and excluded. The final mask is clamped to the valid color range. These channel meanings are not claims about the live garden's existing plant assets.

For the surrounding export workflow, see [Houdini procedural plants for the real-time web](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/).

## Blade count: GPU time changes while RAF stays flat

Each number below is the median of three per-trial percentiles. GPU p50 and p95 come from the separate query pass. RAF values describe the display-paced animation loop.

| Nominal blades | Actually drawn | WebGPU GPU p50 / p95, ms | WebGL2 GPU p50 / p95, ms | RAF p50 / p95, both backends |
| --- | --- | --- | --- | --- |
| 100,000 | 100,008 | 0.20 / 0.26 | 0.18 / 0.73 | 16.70 / 16.80 ms |
| 250,000 | 250,020 | 0.46 / 0.46 | 0.35 / 1.58 | 16.70 / 16.80 ms |
| 500,000 | 500,004 | 0.79 / 0.85 | 0.63 / 1.64 | 16.70 / 16.80 ms |
| 1,000,000 | 1,000,008 | 1.51 / 1.57 | 1.22 / 1.31 | 16.70 / 16.80 ms |

![Blade count versus GPU render-query duration](https://digitalgarden.si/marketing/research/grass-wind-20261027/blade-gpu-time.png)

WebGL2 had lower GPU medians in this particular static workload, but its query tails were much less stable. For example, the 250k noise configuration's median-of-trial GPU p95 was **1.58 ms**, versus a **0.35 ms** p50. These short sequential runs do not establish a general renderer ranking.

WebGPU query values were visibly quantized in roughly **0.065536 ms steps**. Differences on that scale should not be presented as precise shader cost. Both backends kept RAF p95 near 16.8 ms even at the largest count; the experiment did not find a sustained 60 FPS limit.

There was **one 200 ms RAF interval** in a 250k WebGL2 trial. It remains in the data. There was no recorded hidden-tab event, device loss or application error, and the experiment did not establish its cause. A p95 table alone hides this kind of rare stall, which is why the raw intervals and maxima are included.

### Wind off, procedural noise and texture noise

| 250,020 resident blades | WebGPU GPU p50, ms | WebGL2 GPU p50, ms |
| --- | --- | --- |
| off | 0.39 | 0.36 |
| noise | 0.46 | 0.35 |
| texture | 0.46 | 0.37 |

The WebGPU noise-on versus off difference is about one observed timestamp quantum; WebGL2 did not show a reliable noise-off advantage. The 256² texture was not a clear winner. Its periodic waveform also differs from MaterialX noise, so this is not a quality-equivalent substitution test.

The practical result is modest: both implementations worked in this scene. The data does not justify a claim that procedural wind is free, or that texture sampling is always faster.

## Chunk size changes both batching and refill bursts

The pool covers a **96 × 96 metre resident square**. A camera path moves 47.8 m along x over four seconds of fixed simulation time. Cells leaving the square are reassigned and refilled from a seed derived from their world coordinates. Meshes, geometry objects and the two per-instance typed arrays keep their identities. Revisited cells reproduce the same buffer hashes.

This implementation refills a whole entering row synchronously. It still uses JavaScript maps, arrays and audit hashes. It is **not allocation-free**, and the timings include deterministic generation plus hashing. Incremental one-cell-per-frame scheduling was not tested.

| Cell width | Resident / typical drawn blades | Typical total draws | RAF p95 WebGPU / WebGL2 | Largest refill burst WebGPU / WebGL2 |
| --- | --- | --- | --- | --- |
| 8 m | 250,128 / 166,752 | 99 | 16.80 / 16.80 ms | 1.30 / 1.40 ms |
| 16 m | 250,020 / 187,515 | 30 | 16.70 / 16.80 ms | 1.90 / 2.40 ms |
| 32 m | 250,002 / 166,668 | 9 | 16.80 / 16.80 ms | 4.10 / 4.10 ms |

![Chunk size versus RAF p95 during the walking path](https://digitalgarden.si/marketing/research/grass-wind-20261027/chunk-frame-time.png)

![Chunk size versus the largest synchronous refill burst](https://digitalgarden.si/marketing/research/grass-wind-20261027/chunk-refill-burst.png)

Larger cells reduce draw submissions but create larger individual refill bursts. At 32m, the median largest burst was **4.1 ms** on both backends, compared with **1.3–1.4 ms** at 8m. The short path crossed five 8m boundaries, two 16m boundaries and one 32m boundary. No measured walking interval exceeded 25 ms; that does not prove hitch-free behavior on a longer or slower-device run.

The resident footprint has the same width, but grid phase and frustum acceptance differ with cell size. Typical drawn counts range from roughly **167k to 188k**. Therefore the draw/GPU differences are not a pure batching comparison of identical visible blades. There is no universal winning chunk size in this result. I would use the recorded tradeoff to choose the next workload, rather than promote a 0.1 ms RAF p95 difference into a recommendation.

Placement on uneven surfaces remains a separate problem, covered by the [terrain-scatter study](https://digitalgarden.si/blog/threejs-scatter-objects-uneven-terrain/). This fixture has flat ground. Distant vegetation and impostors belong to the [vegetation LOD experiment](https://digitalgarden.si/blog/threejs-webgpu-vegetation-lod/).

## Roots, seams, bounds and shadows: checked with controls

There are **24 GPU function checks and eight image controls per backend**. Float render-target probes reuse the same deformation functions as the scene. At 0°, 90°, 180° and 270°, root displacement was zero in the tested samples; the normalized grass/tree field mismatch stayed below **1.37 × 10⁻⁶**. Identical world roots attributed to adjacent chunks produced identical displacement. This is a sampled function check, not proof of every possible scene or floating-origin scheme.

The grass bounds include a one-metre horizontal margin. With the fixed strength and blade-height limits, the maximum horizontal push is below 0.78 m. Padded frustum culling and disabled culling produced identical images at the four tested angles. Deliberately invalid bounds changed about 42,500 pixels, confirming the comparison could detect missing grass.

The shadow result corrected the original draft. **In pinned Three.js r186, `positionNode` is automatically used when `castShadowPositionNode` is absent.** The installed renderer source has that fallback, and automatic versus explicit shadow-position images were pixel-identical on both backends. A fresh fixture with frozen shadow deformation changed **39,631 pixels on WebGPU and 39,623 on WebGL2** at 1024×576.

Early tests that changed an already compiled shadow override did not produce a useful negative control. The accepted comparison creates fresh fixture meshes/materials and samples across renderer frames. The failed harness runs remain separate; they are not counted as passing evidence. This is why checking that a property was assigned would have been insufficient.

## The RedGPU examples: a separate smoke check

I also opened RedGPU's official [Procedural Grass Field](https://redcamel.github.io/RedGPU/examples/3d/landscape/proceduralGrass/) and [Open World Integration](https://redcamel.github.io/RedGPU/examples/3d/landscape/openWorldIntegration/) examples on the same workstation and browser. Both rendered their terrain and vegetation without captured console errors. At **1936×1089**, each saved performance-panel snapshot showed **60 FPS / 16.70 ms**. The grass panel showed 129,536 instances and capacity 262,144.

These are UI snapshots of different scenes with different rendering features, not repetitions of the Three.js benchmark. I did not independently measure their GPU time or validate their memory counters. The hosted examples were not pinned to an independently verified deployment commit.

The [V4.4.0-Alpha release notes](https://github.com/redcamel/RedGPU/releases/tag/V4.4.0-Alpha) describe camera-centered grass chunks, and identify Foliage and Water.Lake as consumers of the shared wind manager. That wording is not evidence that its grass uses the same wind path. The useful reference here is the organization of the engine features, not an engine winner.

## Memory and reproduction

The two grass instance attributes hold six float32 values per blade: root xyz, yaw, height and a spare coefficient. Their payload is **2,400,192 bytes at 100,008 blades** and **24,000,192 bytes at 1,000,008 blades**. The alternate RGBA8 noise texture adds **262,144 bytes**, without mipmaps.

Those are known data payloads. Renderer memory counters also include buffers, programs and render targets, but are not independently measured total scene VRAM. I did not report an exact per-scene VRAM number.

The reproduction package includes the pinned source, lockfile, owned GLB/HIP, accepted JSON/CSV measurements, protocol, selected captures and charts. Run `npm ci`, `npm run build` and `npm run serve`, then choose the backend in the local viewer. The server refuses to overwrite an existing evidence file. The original draft and excluded pilots remain in the local archive.

This study does not cover physical phones, another GPU or browser engine, thermal soak, alpha-heavy foliage, slope alignment, dynamic LOD, production integration or interactive trampling. It tests a bounded shared vertex wind field and synchronous chunk recycling. It does not show that a complete million-blade game or garden fits the same frame budget.

**Authoring and testing:** Alicia's Digital Garden, with Codex-assisted implementation, native Houdini generation, browser execution and analysis. All reported timings come from saved runs; the article was published on 27 October 2026.

## Download the data and reproduce the study

[Reproduction ZIP](https://digitalgarden.si/marketing/research/grass-wind-20261027/grass-wind-lab.zip) · [File manifest and SHA-256](https://digitalgarden.si/marketing/research/grass-wind-20261027/reproduction-manifest.json) · [All 54 trial summaries](https://digitalgarden.si/marketing/research/grass-wind-20261027/trials.csv) · [All measured RAF intervals](https://digitalgarden.si/marketing/research/grass-wind-20261027/frames.csv) · [GPU timestamp samples](https://digitalgarden.si/marketing/research/grass-wind-20261027/gpu-timestamps.csv).

The archive contains the pinned browser source, owned GLB/HIP fixture, accepted raw JSON/CSV data, failed pilot receipts, protocol and figures. Native Houdini requires a compatible installation and license; the prepared GLB runs without Houdini. Private editorial history and operational account records are excluded. The original local measurements remain unchanged.

## Related field notes

- [Three.js rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/)
- [1,600 trees: mesh LOD and impostors](https://digitalgarden.si/blog/threejs-webgpu-vegetation-lod/)
- [Terrain scattering and root placement](https://digitalgarden.si/blog/threejs-scatter-objects-uneven-terrain/)
