# Procedural Plants for the Web: Houdini Geometry and Wind Data

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Build and test a small Houdini plant for Three.js: seeded geometry, two LODs, custom wind attributes, instancing and alpha tested leaf cards.

- Author: Alicia Lu
- Published: 2026-10-19
- Updated: 2026-10-25
- Canonical HTML: https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/

A procedural plant for the web needs more than an attractive still. It needs repeatable geometry, a stable root, a useful lower detail version and data that survives export. I built an original grape hyacinth fixture in Houdini 22.0.429 and tested it in Three.js 0.186.1 on 19 October 2026. The result is a small working HDA and a browser experiment, with explicit limits around wind, transparency and performance.

This plant fixture is one step in [the full rendering guide for the interactive 3D garden](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

![One thousand instances of the original test plant with shader driven wind in WebGPU](https://digitalgarden.si/marketing/research/webgpu-series-20261019/plant-wind.png)

## A small generator with inspectable parameters

The HDA contains a Python SOP that builds a stem, five leaves and eight tiers of five bell shaped flowers. It exposes a seed, LOD, height in meters and stem side count. The tested default is seed 11, height 0.6 meters and eight stem sides. The geometry includes UVs, vertex colors and normals; separate point attributes describe stiffness and wind phase.

| Output | Points | Triangles | Forced cook, seed 11 |
| --- | --- | --- | --- |
| LOD0 | 3,838 | 6,056 | 89.30 ms |
| LOD1 | 1,553 | 2,116 | 38.06 ms |

One hundred forced seed cooks took 9.38 seconds. These are wall clock measurements of this Python SOP in hython on this workstation. They are not a Houdini Performance Monitor breakdown or a comparison with a compiled SOP implementation. The saved HIP was loaded again for the Gaussian fixture generation, which read the expected plant geometry successfully.

The lower detail plant is authored by reducing generator segments. It is useful for testing a smaller shape, but it is not the production garden’s nested LOD generation and transition system. A distant silhouette can be acceptable even when a close view is visibly coarser. Those acceptance distances belong to each scene and camera.

## Bake attributes, then animate in the shader

`_STIFF` ranges from zero at the base to one toward flexible tips. `_PHASE` is a stable per plant seed value; seed 11 produces 0.507. In Three.js, the GLTFLoader exposes those attribute names in lowercase. The browser fixture reads `_stiff` and `_phase` through TSL.

Its displacement is deliberately simple:

```js
const stiff = attribute('_stiff', 'float');
const phase = attribute('_phase', 'float');
const bend = sin(clock.mul(1.8).add(phase.mul(6.283)))
  .mul(stiff.pow(2)).mul(strength).mul(0.10);
material.positionNode = positionLocal.add(vec3(bend, 0, 0));
```

Because zero stiffness produces zero displacement, the fixture’s root remains fixed algebraically. This is baked deformation data, not a baked animation. The function offsets points horizontally; it does not preserve stem arc length, solve collisions or correct every shading normal. Calling it a botanical simulation would overstate what was built.

The actual [interactive garden](https://digitalgarden.si/play/) uses a more involved shared bend and normal deformation, with coherent flower crowns and per stalk anchors for relevant species. The downloadable shader illustrates the export contract without redistributing that private engine. A copied sine shader should not be mistaken for the whole production animation system.

## Attribute survival is an animation test

The native Houdini GLB included `_STIFF` and `_PHASE`. So did its Draco export and the tested glTF Transform Meshopt candidate. The gltfpack candidates dropped them, including a run with `-kv`. That discovery changes which output can be accepted for this shader: a texture optimized file may load successfully while lacking the intended motion data.

In the harness, a missing stiffness attribute falls back to height and a missing phase falls back to a constant. That keeps a comparison visible, but it is not proof of a correct export. A production acceptance check should fail or explicitly flag that fallback. The [compression article](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/) separates successful decoding from usable deformation attributes.

An earlier native export also failed normal validation at degenerate leaf endpoints. Adding a defined fallback normal corrected the final GLBs. Validation found a problem the distant plant preview did not make obvious. A later [Houdini versus Geometry Nodes plant export](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/) assembled one controlled plant in both applications. The accepted GLBs corresponded triangle for triangle and kept the wind attributes, after a Geometry Nodes material setup that lost its foliage colors was rejected.

## Wind, interaction signals and leaf cards

I tested a 1,000 plant patch at 1280 × 720, DPR 1, on an RTX 5070 Ti in the visible Chromium 155 browser. Each run used 1.5 seconds of warmup and 30 seconds of the same scripted orbit. Wind off and wind on both measured about 59.99 FPS with two total render calls, including the output pass. This ceiling limited observation does not establish the shader’s cost on a phone or at a higher resolution. A later [WebGPU versus WebGL plant test](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/) kept this plant and separated InstancedMesh, BatchedMesh and one mesh per copy, because a capped frame rate hides the submission difference.

The interaction variant adds a scripted oscillating touch uniform. It also stayed near 59.99 FPS with two calls. It tests a deformation signal without a pointer event, spatial search, collision or input latency measurement. Real garden brushing involves more work and cannot inherit this fixture’s result automatically.

For the alpha test variant, I removed the five solid leaves and added five segmented cards per plant, using an original canvas generated leaf mask and `alphaTest = 0.5`. This is 5,000 leaf instances in an additional draw: three total calls, 5,936,001 total triangles and 59.99 FPS in the sampled patch. It changes the geometry and shader workload together. It is a practical construction comparison, not a controlled measurement of alpha testing in isolation.

![The same fixture with alpha tested leaf cards replacing the solid leaves](https://digitalgarden.si/marketing/research/webgpu-series-20261019/plant-alpha-cards.png)

The cards use a hard alpha cutoff rather than blended transparency. Camera angle, double sided rendering, silhouette loss and distant aliasing still need visual inspection. A desktop FPS number does not settle those questions. A later still, [MuTracer with a Houdini plant](https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/), path-traced double-sided alpha masks and vertex colors on frozen geometry. It does not measure this orbiting patch, and its WebGL run lost the context before a finished image.

For placing those assets, the later [uneven-terrain scatter test](https://digitalgarden.si/blog/threejs-scatter-objects-uneven-terrain/) separates surface attachment from minimum spacing. It uses an independent owned terrain, native Houdini layouts and runtime sampling, including a boundary-ray validator that initially rejected valid points.

## Keep the shadow contract explicit

Moving vertices in a color material raises a second question: which positions does the shadow pass use? I captured frozen wind poses with default, explicit rest position and explicit deformed `castShadowPositionNode` settings. The default and explicit deformed screenshots were identical in the inspected region; forcing rest positions changed 12,014 pixels there. The default worked for this pinned material and renderer. This was not a reproduced default shadow bug.

![A frozen bent test plant with its default directional shadow, which matched the explicit deformed shadow setting](https://digitalgarden.si/marketing/research/webgpu-series-20261019/plant-shadow.png)

The production garden’s stylized lighting is a separate implementation. This small directional shadow map test does not certify production shadows or promise correct results for every custom TSL material. Inspect the actual bent silhouette and its ground shadow whenever a deformation changes.

## What is included

Codex executed the generator, native export and browser checks; the dated results describe that workstation run.

The [experiment package](https://digitalgarden.si/marketing/research/webgpu-series-20261019/webgpu-garden-tests.zip) includes the original generator and HDA, compact native and browser results, and a standalone viewer. The large noisy test texture can be regenerated from its seeded script. Editable full HIP files and local capture evidence remain with the project; third party tool binaries and the private garden engine are excluded.

The test proves a seeded Houdini plant can reach the browser with its wind data intact through selected export paths. It does not prove scan quality, biome realism, mobile performance or robust physical bending. It also does not settle [code versus Houdini](https://digitalgarden.si/blog/houdini-vs-code-procedural-threejs-slop-newyork/) for a finished scene. A later 1080p profile compared this garden with slop-newyork and kept the two pipelines mixed rather than declaring a winner. For the larger context, read [the garden pipeline](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/) and the separate [Gaussian splat compatibility test](https://digitalgarden.si/blog/gaussian-splats-threejs/). That splat fixture is compared again in [Spark, GaussianSplats3D and native Three.js](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/).

## References

SideFX documents the [glTF exporter and custom attribute rules](https://www.sidefx.com/docs/houdini/nodes/out/gltf.html). The browser uses the official [Three.js WebGPURenderer](https://threejs.org/docs/pages/WebGPURenderer.html). Versions and documentation were checked on 19 October 2026; the download records the exact experiment dependencies.
