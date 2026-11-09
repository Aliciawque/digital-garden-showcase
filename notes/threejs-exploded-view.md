# Three.js Exploded View: Explode a Real Multi-Part GLB (and What It Costs in Draw Calls)

> Build a Three.js exploded view from a native Houdini GLB, with measured draw calls, first GPU completion, and click-to-isolate costs for 10 to 200 parts.

- Author: Alicia Lu
- Published: 2026-11-09
- Canonical HTML: https://digitalgarden.si/blog/threejs-exploded-view/

A Three.js exploded view moves a GLB assembly apart by changing its part transforms. For this test, Codex built and exported a potted flower in Houdini 22.0.429 and ran it on Alicia’s RTX 5070 Ti desktop. The fixture has five assemblies: pot, soil, stalk, leaves and bloom.

**The short answer.** Give each movable part a name and a pivot, then save its rest position. The merged, 10-, 50- and 200-part GLBs all contain 30,848 triangles. They took 1, 10, 50 and 200 model draws on both backends. The raw renderer counts were 2, 11, 51 and 201 because an empty scene recorded one draw. From merged to 200 parts, WebGPU load to first GPU completion went from 89.7 to 116.2 ms. Point picking went in the other direction because the split created tighter bounds.

![One purple potted flower assembled, flat-exploded and hierarchy-exploded in Three.js](https://digitalgarden.si/marketing/research/exploded-view-20261109/assembly-comparison.png)

## Export the parts and pivots from Houdini

The native scene uses a Geometry Stash SOP inside each OBJ mesh. Each part has a pivot at its local bounds center and is parented under a named assembly null. The merged control stores the same colored geometry under one mesh at the origin. The 10- and 50-part layouts group the same 200 source components in fewer meshes. The glTF 2.0 ROP exports the OBJ hierarchy, with one mesh primitive per part. A node extras dictionary stores the assembly name and component count. Three.js read both back from userData. [SideFX glTF 2.0 ROP](https://www.sidefx.com/docs/houdini/nodes/out/gltf.html)

All four native exports passed the glTF validator with zero errors and warnings. A separate audit matched every oriented triangle against the merged control and compared its world-space corners, normals and vertex colors. The largest component difference was 2.46 × 10⁻⁷. The saved HIP reopened with all four layouts and their geometry intact.

## Flat traverse and a two-stage hierarchy

Flat explode moves each part away from the model center. The hierarchy path first moves the five parents apart. At slider value 0.5, the children still have their rest positions inside those groups. The second half moves each child away from its own group center. The two modes use different offsets to illustrate how they open. The flat view spreads parts along the plant’s overall shape, while the hierarchy view opens the five assemblies in stages.

Both methods compute positions from stored rest values. The test model sits under a rotated, nonuniformly scaled parent. To apply a displacement in world units, convert two points to the parent’s local space and subtract them. The resulting vector length accounts for parent scale. [Three.js Object3D](https://threejs.org/docs/pages/Object3D.html)

```js
function localOffset(node, point, worldOffset) {
  const start = node.parent.worldToLocal(point.clone());
  return node.parent.worldToLocal(point.clone().add(worldOffset)).sub(start);
}

// Hierarchy path after the local offsets have been prepared.
const groupT = Math.min(1, t * 2);
const partT = Math.max(0, t * 2 - 1);
for (const group of groups) {
  group.node.position.copy(group.rest).addScaledVector(group.delta, groupT);
}
for (const part of parts) {
  part.node.position.copy(part.rest).addScaledVector(part.child, partT);
}
```

Across 24 parted contexts, 960 out-and-back cycles returned to rest with zero measured drift. The largest world-position error in the displacement checks was 1.33 × 10⁻¹⁵ scene units. The two GIFs share a wider camera so the plant stays in frame at full separation. The timing runs use their own fixed camera.

![Flat explosion of the 10-part Houdini plant from rest to full separation and back](https://digitalgarden.si/marketing/research/exploded-view-20261109/flat.gif)

![Two-stage hierarchy explosion of the same plant with parent assemblies opening before their children](https://digitalgarden.si/marketing/research/exploded-view-20261109/hierarchy.gif)

## Draw calls and first completion

Chrome and Three.js 0.186.1 ran at 1280 × 720, device pixel ratio 1. The WebGL2 path uses the WebGPU renderer’s WebGL backend on ANGLE/D3D11. Both selected NVIDIA hardware. Four fresh contexts per layout and backend loaded each GLB from localhost with cache disabled. Contexts shared a Chrome process and did not clear driver or OS caches.

These are medians in milliseconds. Asset-to-completion starts after renderer initialization and includes loading, scene setup, awaited compile and the first render with its work wait. WebGPU used the queue completion promise and WebGL2 used gl.finish(). This measures submitted work completion before display presentation.

| Layout | Backend | Raw draws | GLB MiB | Fetch ms | Parse ms | Compile ms | First render + wait ms | Asset to completion ms |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Merged | WebGPU | 2 | 0.965 | 5.6 | 2.4 | 60.9 | 20.7 | 89.7 |
| 10 parts | WebGPU | 11 | 0.977 | 5.2 | 3.4 | 61.5 | 20.7 | 91.5 |
| 50 parts | WebGPU | 51 | 1.027 | 5.2 | 4.9 | 64.6 | 21.0 | 96.4 |
| 200 parts | WebGPU | 201 | 1.215 | 5.9 | 9.8 | 71.2 | 30.0 | 116.2 |
| Merged | WebGL2 | 2 | 0.965 | 6.2 | 2.4 | 24.2 | 7.6 | 43.4 |
| 10 parts | WebGL2 | 11 | 0.977 | 6.0 | 3.2 | 27.0 | 8.5 | 47.3 |
| 50 parts | WebGL2 | 51 | 1.027 | 6.3 | 5.1 | 36.8 | 10.0 | 58.0 |
| 200 parts | WebGL2 | 201 | 1.215 | 6.4 | 9.2 | 29.7 | 13.4 | 59.1 |

The 200-part asset cost about 26.6 ms more than the merged WebGPU median. The WebGL2 50-part runs ranged from 46.4 to 78.1 ms, which is wider than the difference between the 50- and 200-part medians. The file size, number of nodes and buffer layout all change with the split. The separate columns are medians from four runs and do not necessarily add up.

![Median load to first GPU completion for the merged and parted GLBs](https://digitalgarden.si/marketing/research/exploded-view-20261109/first-completion.png)

The harness reads renderer.info.render.drawCalls. Early pilots caught the empty-scene draw and a second issue: back-to-back render calls could accumulate statistics. The formal harness resets counters before each render and measures the empty scene after first completion. [Three.js Info](https://threejs.org/docs/pages/Info.html)

| 200-part mode | Raw draws | WebGPU CPU submit ms | WebGL2 CPU submit ms | WebGPU / WebGL2 p95 rAF ms |
| --- | ---: | ---: | ---: | ---: |
| Assembled | 201 | 0.8 | 0.6 | 16.8 / 16.7 |
| Flat | 201 | 1.1 | 0.9 | 16.8 / 16.8 |
| Hierarchy | 201 | 1.0 | 0.9 | 16.8 / 16.8 |

CPU submit is elapsed time around the render call and counter reset. It excludes the per-frame transform update and GPU wait. Warm rAF intervals stayed near the refresh cadence. The assembly uses one opaque vertex-color material, no shadows and disabled frustum culling, so moving parts does not change the draw count.

## Picking got cheaper as the mesh count rose

The harness times setFromCamera and intersectObjects on the same 49 rays. Each context ran 12 timed sweeps after three warm-up sweeps. Every sweep had 12 hits. The per-ray times are CPU measurements grouped by the renderer in use in that context. Picking uses an explicit mesh list and reuses its result array. [Three.js Raycaster](https://threejs.org/docs/pages/Raycaster.html)

| Layout | WebGPU µs/ray | WebGL2 µs/ray | Meshes reaching triangle loop per sweep | Triangle loop entries per sweep |
| --- | ---: | ---: | ---: | ---: |
| Merged | 1,373.5 | 676.5 | 41 | 1,264,768 |
| 10 parts | 234.7 | 117.3 | 59 | 208,192 |
| 50 parts | 106.1 | 55.1 | 105 | 78,104 |
| 200 parts | 65.3 | 30.6 | 130 | 37,952 |

An untimed CPU control counted entries after the built-in bounding tests in the same Three.js version. From 10 to 200 meshes, more candidates passed the bounds checks but many fewer triangles reached the inner loop. All 49 hit-or-miss outcomes matched. The largest difference in the nearest hit distance was below 0.000002 scene units. The result depends on the asset’s bounds and the ray distribution. [Three.js r186 mesh raycasting source](https://github.com/mrdoob/three.js/blob/r186/src/objects/Mesh.js)

![Measured mean-per-sweep ray-picking cost summarized by median for 10, 50 and 200 parts](https://digitalgarden.si/marketing/research/exploded-view-20261109/ray-picking.png)

## Click to fade or hide

The live viewer picks on pointer release and ignores drags longer than five pixels. Fade assigns unselected meshes a shared translucent material at opacity 0.17. The selected mesh retains the opaque material. Hide makes the other meshes invisible and filters them out of the next pick. The camera stays fixed for the comparison.

| Layout | Fade / hide raw draws | WebGPU fade / hide CPU submit ms | WebGL2 fade / hide CPU submit ms | Model triangles when hidden |
| --- | ---: | ---: | ---: | ---: |
| 10 parts | 11 / 2 | 0.2 / 0.2 | 0.2 / 0.2 | 2,176 |
| 50 parts | 51 / 2 | 0.4 / 0.2 | 0.3 / 0.1 | 2,176 |
| 200 parts | 201 / 2 | 0.6 / 0.2 | 0.6 / 0.2 | 1,152 |

Fade kept the whole shape visible. Hide left one model draw plus the empty-scene draw. The hidden triangle count depends on the chosen part, whose grouping changes with the layout. The table shows warmed CPU submit medians. Transition completion samples are in the raw data. Some WebGL2 samples hit the timer floor, so they cannot support a more precise latency claim.

A separate browser interaction test moved the slider, changed the rendered canvas, picked a part, toggled hide and fade, and reset. A 300 ms idle window after each timed sequence logged zero further render calls in all 32 contexts. This counts submitted frames. GPU utilization and power were outside the test.

## Reproduce the experiment

The bundle includes four native GLBs, the Houdini scene and generator, browser viewer, raw trials, chart sources, GIFs and the picking control. The formal set contains 32 contexts, 7,680 rAF intervals and 18,816 timed ray evaluations, including 14,112 on the three parted layouts. Pilot failures are separate from the formal data. [Download the reproduction bundle](https://digitalgarden.si/marketing/research/exploded-view-20261109/exploded-view-reproduction.zip) or [inspect the summary CSV](https://digitalgarden.si/marketing/research/exploded-view-20261109/summary.csv).

The results cover one desktop, one browser version, one camera for timing and an untextured authored model. Mobile devices, production asset loading and batched alternatives are outside the measured scope. Related studies cover [GLB loading and first frame](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/) and the [forest draw budget](https://digitalgarden.si/blog/threejs-forest-performance-budget/) with different fixtures.

*By Alicia, a 3D technical artist, TA and independent developer. Codex assisted with implementation, measurement and drafting.*

## Questions

### Can one merged GLB mesh explode?

A merged mesh has one object transform. Independent movement requires separate logical parts, such as separately transformed meshes or an explicit per-part representation. This study measured separate native GLB meshes.

### Does every named part always add one draw?

Each mesh added one model draw in this one-primitive, one-material test. Extra materials, passes and batching change the count. The raw counter also recorded one draw in an empty scene.

### Should click-to-isolate fade or hide other parts?

Fade keeps the surrounding assembly visible and retained all model draws here. Hide left one model draw for the selected mesh. Choose based on whether the rest of the assembly helps the reader.
