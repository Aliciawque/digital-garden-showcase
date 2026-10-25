# Three.js Terrain Scatter: Fix Floating, Then Fix Clumping

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> I tested 70 terrain-scatter trials in Three.js and Houdini: BVH placement, weighted sampling, hard spacing and the boundary checks that fooled a ray test.

- Author: Alicia Lu
- Published: 2026-10-25
- Canonical HTML: https://digitalgarden.si/blog/threejs-scatter-objects-uneven-terrain/

Putting an instance on the terrain and keeping it away from its neighbors are separate jobs. In this desktop test, ordinary downward raycasting and its BVH accelerated counterpart produced the same positions. Both still clumped. Adding a minimum-distance rule fixed that second problem; changing the renderer would not.

I measured seven paths at 1,000 and 10,000 points, with five repetitions each. Houdini authored the terrain and two baked layouts. Three.js generated the runtime layouts and rendered the results. The useful outcome is a set of placement contracts and reproducible failures, rather than a universal software winner.

![The accepted native terrain layout rendered with upright flowers and slope-aligned rock proxies](https://digitalgarden.si/marketing/research/terrain-20261025/terrain-correct.jpg)

For the wider pipeline, see [the garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

## A controlled hilly scene

This is an independent, owned test scene. It is not the complete [production garden](https://digitalgarden.si/play/) or a physical phone test. The production garden has its own subtle ground-height function and seasonal art decisions; replacing that system was outside this experiment.

The fixture is an 80 by 80 metre height field, with 4,225 vertices and 8,192 triangles. A separate Houdini 22.0.429 Commercial process created its editable SOP network, exported its positions and triangles to JSON, and wrote a 200,876-byte GLB. The GLB passed Khronos validation with zero errors and warnings; its 4,225 unique positions match the terrain data used by the browser. Reopening the saved HIP returned 10,000 points in its final relaxed Scatter SOP.

The browser used Three.js 0.186.1 and three-mesh-bvh 0.9.16. The machine was a Ryzen 9 9950X with 32 logical processors and an RTX 5070 Ti, driver 617.42. The in-app Chromium reported version 156. Browser versions and drivers are part of the record, not a promise about other devices.

I deliberately moved the terrain by `(7, 2, -4)`, rotated it 0.31 radians around Y, and scaled it by `(1.2, 1.1, 0.9)`. That makes missing world transforms visible. Treating every test mesh as an identity transform would hide an important class of failures.

For surface-weighted paths, a whole triangle receives density zero when its **local** upward face-normal component is below 0.85. Exactly 1,050 triangles are masked. This is a local-space density preference, not a guaranteed world-space slope limit after nonuniform scaling and relaxation.

## Raycast placement: BVH changes search cost, not distribution

For the direct comparison, both [raycast](https://threejs.org/docs/pages/Raycaster.html) paths consume the same seeded XZ candidates over the interior of the terrain. The origins are transformed to world space and cast downward. The [BVH path](https://github.com/gkjohnson/three-mesh-bvh) overrides only the test mesh's raycast function; it does not modify the global Mesh prototype.

```js
import { MeshBVH, acceleratedRaycast } from 'three-mesh-bvh';

terrain.geometry.boundsTree = new MeshBVH(terrain.geometry);
terrain.raycast = acceleratedRaycast;
raycaster.firstHitOnly = true;
raycaster.set(worldOrigin, new THREE.Vector3(0, -1, 0));
const hit = raycaster.intersectObject(terrain, false)[0];
```

Across the exported seed-25 layouts, the accelerated and plain positions matched to within 1e-9 metres. The timing difference therefore concerns equivalent ray queries in this fixture.

The following values are median CPU placement times in milliseconds. They include preparing geometry, any acceleration or sampler build, point generation and output-array construction. They exclude fetching, JSON parsing, instance-matrix construction and GPU work. Build time is already inside the total and must not be added again.

| Path | 1,000 points | 10,000 points | Distribution |
| --- | ---: | ---: | --- |
| Downward raycast | 218.0 | 2,047.9 | Uniform local XZ |
| Same rays with BVH | 2.5 | 10.2 | Same candidates |
| Bridson Poisson + BVH rays | 69.1 | 71.2 | Hard planar spacing |
| Weighted MeshSurfaceSampler | 1.6 | 3.6 | Eligible local surface area |
| Weighted sampler + rejection | 2.7 | 27.4 | Area candidates with hard spacing |
| Rebuild baked Houdini positions | 0.1 | 0.4 | Already loaded seed-25 data |

The 10,000-ray plain range was 2,022.2–2,183.4 ms; BVH was 9.6–11.1 ms, including a median 1.0 ms BVH build. Those are warm browser-worker observations on a small known terrain, not website startup or a generic speedup guarantee. The native-data row is deliberately narrow: it transforms already decoded arrays and does not make the whole baked pipeline “free.” Small values reach the browser timer's resolution and should not be ranked by tenths of a millisecond.

The sampler and raycast rows have different distributions and eligibility rules. Their timings cannot establish which method produces the best equivalent scene.

## MeshSurfaceSampler uses geometry space

[MeshSurfaceSampler](https://threejs.org/docs/pages/MeshSurfaceSampler.html) samples the geometry's attributes. It does not apply the terrain object's world matrix for you. Positions need the complete matrix; normals need its inverse transpose when scale is nonuniform.

```js
const sampler = new MeshSurfaceSampler(terrain)
  .setWeightAttribute('density')
  .setRandomGenerator(seededRandom)
  .build();
const normalMatrix = new THREE.Matrix3().getNormalMatrix(terrain.matrixWorld);

sampler.sample(position, normal);
position.applyMatrix4(terrain.matrixWorld);
normal.applyNormalMatrix(normalMatrix);
```

![Deliberately omitted world transform: points float beside and intersect the transformed terrain](https://digitalgarden.si/marketing/research/terrain-20261025/terrain-missing-transform.jpg)

The negative control omitted the world matrix from 1,000 baked points. Nearest-surface validation then found 999 above the 1 mm distance threshold. Applying an ordinary direction transform to normals instead of the normal matrix introduced a maximum angular difference of about 10.29 degrees. These are measurable consequences of this transform, not universal error magnitudes.

## Random area sampling does not promise separation

At 10,000 points, the weighted sampler's median nearest-neighbor distance was about 0.369 m. A median of 7,192 points per trial had a neighbor closer than 0.5 m. Every point was correctly attached to the surface; clustering was still substantial.

The rejection variant stores accepted points in a spatial grid and rejects candidates closer than 0.5 m in world XZ. Cell size is 0.5 m and the implementation inspects neighboring cells. Across all five 10,000-point runs, its smallest accepted spacing was 0.500002 m and zero points violated the constraint. Filling the layout takes longer because some candidates are discarded.

That implementation is rejection sampling, not Bridson's algorithm. I tested Bridson separately using the [original 2007 procedure](https://www.cs.ubc.ca/~rbridson/docs/bridson-siggraph07-poissondisk.pdf): an active list, 30 annulus candidates and a grid with cell width `r / sqrt(2)`. The local radius accounts for the terrain's smallest planar scale. I fill the domain, shuffle, then select the requested count, so the 1,000-point result is not merely the beginning of one growing patch. That full-domain work explains why its 1k and 10k timings are relatively close.

After BVH projection, all five 10k Poisson layouts had at least 0.500287 m planar separation. Different sampling distributions remain different artistic choices. Independent all-pairs checks of the seven 1k layouts agreed with the spatial-grid nearest-neighbor calculation.

## Houdini Relax helps without imposing our hard radius

The native network uses a primitive density attribute, [Scatter 2.0](https://www.sidefx.com/docs/houdini/nodes/sop/scatter.html), fixed counts, seeds 25–29 and either zero or ten Relax Iterations. Five native cooks used different seeds to avoid mistaking a cached result for recomputation. The browser rebuilds one exported seed-25 layout; it does not silently resample Houdini five times.

For 10,000 points, median native cook time was 1.54 ms without relax and 34.95 ms with ten iterations. At 1,000 points those medians were 0.40 and 13.95 ms. They exclude process launch, modeling, export and browser loading.

Ten relax iterations increased the 10k seed-25 median planar nearest-neighbor distance from 0.372 to 0.692 m. However, 131 points still had a neighbor within 0.5 m, with a minimum of about 0.118 m. Relaxation improved the distribution without satisfying the particular hard exclusion rule used by the rejection sampler. If an asset requires a guaranteed radius, validate and filter that radius explicitly.

The relaxed 10k position/normal JSON is 1,305,037 bytes. A matching little-endian Float32 representation is 240,000 bytes. This is a storage comparison for six floats per point; neither file includes per-instance scale, variant IDs, compression or network timing. The binary round trip was exact for these already float-precision native values.

## A ray miss is not automatically a floating point

In the accepted relaxed 10k layout, 125 vertical reference rays missed. All 125 points were within the 1 mm surface tolerance. An independent audit built its BVH from world-transformed Float32 terrain positions; its maximum nearest-surface distance over the relaxed 10k layout was below two micrometres. The original local-geometry check and this world-space audit both passed. These boundary points exposed numerical edge sensitivity in the ray validator.

I retain both measurements: the number of ray misses and the independent closest-surface distance. Counting every miss as a floating instance would have falsely rejected this export. Ignoring every miss would also be unsafe, because the deliberate missing-transform control contains real large errors.

Every accepted method had zero point-to-surface distances above 1 mm. This validates the placement anchor, not the whole object volume. A wide rock can intersect a hillside even when its anchor is exact. A centered 0.8 m primitive deliberately shifted down by 0.4 m failed all 1,000 surface checks, demonstrating why the model's base pivot must be part of the contract.

## Rendering and orientation are separate checks

The preview alternates small owned flower proxies and rocks in two InstancedMesh batches. Flowers stay upright; rocks align to the transformed normal. It recomputes each batch's bounding sphere after updating matrices. The terrain and all layouts can be inspected in the same viewer.

The WebGPU and forced-WebGL2 profiles kept 1280 × 720 and DPR 1, warmed each layout for one second, then recorded five seconds of static RAF intervals. All fourteen trials rendered, with zero hidden-frame samples, about 59.99 FPS and 95th-percentile frame intervals of 16.7–16.8 ms. The renderer reported four draws and 308,193 triangles, including its background pass. Refresh-paced desktop results are compatibility observations, not evidence of equal GPU headroom, phone performance or power consumption. The existing [instancing benchmark](https://digitalgarden.si/benchmarks/threejs-webgpu-vs-webgl-instancing/) remains separate and unchanged.

Codex executed the native and browser work and made several harness mistakes during development: mixing indexed and non-indexed proxy geometry; assigning the density mask to points instead of primitives; initially using the wrong Houdini polygon winding; and relying solely on boundary ray hits. The corrected code asserts a nonempty eligible region, upward native normals, valid assets and surface distances. Rejected pilots are labeled separately from the accepted 70-row run.

## Choose the placement contract

| Need | Tested starting point |
| --- | --- |
| A few objects, straightforward terrain queries | Plain downward rays |
| Many equivalent runtime terrain queries | Mesh-local BVH acceleration |
| A density-weighted eligible surface | MeshSurfaceSampler plus correct world transforms |
| A strict minimum planar spacing | Explicit rejection or Poisson, then validate |
| Static authored layout | Houdini bake, with relax and delivery checks |

For a regular height field, a direct piecewise-triangle lookup may avoid ray search. I did not benchmark that additional path here. Bilinear height interpolation should not be assumed identical to a triangulated rendered surface.

The original [Houdini plant and wind note](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/) covers asset attributes. [GLB loading](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/) covers cache and first-frame timing. Neither result is replaced by this point-placement study.

Download the [owned terrain-scatter lab](https://digitalgarden.si/marketing/research/terrain-20261025/terrain-scatter-lab.zip) (6.81 MB), with the editable portable HIP, pinned package/lockfile, native point files and accepted raw receipts. The archive freezes the verified prepublication lab snapshot; its draft copy is historical. The [70-row CSV](https://digitalgarden.si/marketing/research/terrain-20261025/placement.csv), [grouped results](https://digitalgarden.si/marketing/research/terrain-20261025/results.json), [world-space audit](https://digitalgarden.si/marketing/research/terrain-20261025/world-space-audit.json) and [fourteen renderer profiles](https://digitalgarden.si/marketing/research/terrain-20261025/render-profiles.json) are also available separately. No physical phone, dynamic terrain deformation, full plant collision or production-garden replacement was tested.
