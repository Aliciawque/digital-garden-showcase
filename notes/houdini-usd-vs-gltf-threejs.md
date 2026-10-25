# Houdini USD vs glTF: A Two-Plant Three.js Test

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> The same Houdini plant pair exported as GLB, USDC and USDZ: measured browser loads, textures, variants, instance representation and export failures.

- Author: Alicia Lu
- Published: 2026-10-20
- Updated: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/houdini-usd-vs-gltf-threejs/

The native GLB loaded both plants. The same authored pair, delivered as USDC or USDZ through Three.js USDLoader 0.186.1, loaded only one plant and did not retain the vertex-colour or custom wind attributes. USD retained the authoring concepts in Houdini, but this browser representation failed the pair's feature requirements. That is the main result of this test, not a format-speed victory.

For where this delivery test sits among the other garden experiments, see the [rendering guide overview](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

Codex ran the native Houdini scripts and browser checks on 20 October 2026. The fixture is an owned procedural grape hyacinth used to study the asset boundary behind [digitalgarden.si](https://digitalgarden.si/), Alicia's interactive 3D garden. It is not a production model.

![The paired USDZ plant fixture loaded through the pinned Three.js USDLoader](https://digitalgarden.si/marketing/research/comparisons-20261020/usd-browser.jpg)

## Build a native two-plant fixture

Houdini 22.0.429 generated one 6,056-triangle plant, with normals, UVs, vertex colour and wind attributes. The visible pair places copies at X=0 and X=0.3, for 12,112 triangles. The USD stage contains two seed variants, 11 and 12, and an instanceable internal reference for the second plant. Both variants use 3,838 points and 6,056 triangles per plant, with different procedural leaf shapes.

The USD was authored with pxr under Houdini's hython and composed/read back through a native Solaris Sublayer LOP. It is not claimed to be an automatic Component Builder material conversion. The root layer was exported as USDC; UsdUtils packaged the layer and shared PNG as USDZ. The HIP was saved and reopened, with both variants and the native instance relation verified. OpenUSD reported 0.26.5.

The material probe used UsdPreviewSurface, an sRGB UsdUVTexture and a deliberately noisy 2048 PNG. glTF used Houdini's native principled-material export with the same image. Different exporter mappings can still produce shading differences, so I do not describe the materials as visually identical. MaterialX conversion was not tested.

## Two rejected exports mattered

Calling Usd.Stage.Export() flattened the composition and removed the variant set. Exporting the authored root layer preserved it. Flattening can be appropriate for delivery, but it cannot be presented as retaining editable variant structure.

The first packed-copy GLB also contained an unused image entry without a resource URI or buffer view. A downstream tool rejected it. I retained that failure, unpacked the pair before the native glTF ROP, and re-exported. The accepted GLB passed glTF validation with zero errors and warnings; seven buffer-view-target hints remain. The accepted result therefore bakes the two copies into one mesh. That is this exporter path's result, not a limitation imposed by the glTF format.

## Packaged bytes and local loading

Three.js 0.186.1 used GLTFLoader or USDLoader in the same classic WebGL viewer, 1280 × 720 at DPR 1. Each format used five distinct loopback URLs. The timer includes fetch, parsing, texture readiness, compilation and a submitted frame, without a GPU completion fence. Code, OS caches and GPU pipelines were warm. USDC's external PNG can be cached, so its timing is a multi-file diagnostic, not directly equivalent to the self-contained GLB/USDZ loads.

| Accepted input | Main-file MB | Additional image MB | Median local load to submitted frame |
| --- | --- | --- | --- |
| GLB | 4.726 | embedded | 63.7 ms |
| USDZ | 4.608 | embedded | 71.3 ms |
| USDC | 0.269 | 4.338 | 44.8 ms |

The standalone USDC is not a 0.27 MB textured delivery: its external image must be counted. Likewise, a smaller USDZ here does not prove USD is universally smaller than glTF. The two paths encode variants, geometry sharing and custom data differently. The large artificial PNG dominates both packages. More importantly, the loaded USD scene has half the intended triangles, so these timing rows are compatibility diagnostics and cannot be ranked as equal-workload performance.

## Inspect the browser objects

| Loaded input | Mesh objects | Unique geometries | Materials | Texture objects | Triangles | Submitted calls |
| --- | --- | --- | --- | --- | --- | --- |
| GLB | 1 | 1 | 1 | 1 | 12,112 | 1 |
| USDZ | 1 | 1 | 1 | 1 | 6,056 | 1 |
| USDC | 1 | 1 | 1 | 1 | 6,056 | 1 |

The GLB returned 12,112 triangles, UVs, vertex colour, _stiff and _phase. USDC/USDZ returned 6,056 triangles with position, normal and UV only. A texture object was present, but without the authored vertex colour the plant also appeared neutral rather than matching the GLB's colour modulation. A later [Houdini plant GLB export](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/) from Houdini and Blender Geometry Nodes kept the wind attributes in both accepted files. A Geometry Nodes GLB that lost its foliage colors was rejected even though validation reported no errors. The second instance's absence is a failed rendering requirement, not an optimization. A native USD instance flag does not by itself create Three.js InstancedMesh, and the glTF format's support for an instancing extension does not make this native ROP emit it.

USDComposer's explicit seed selections produced different loaded position hashes in this test. That check uses the pinned parser/composer API and recomposes the stage; it is not a claim that USDLoader supplies a complete interactive variant-switching UI. The separate USDA transform-animation diagnostic exposed 1 clip(s) and numerically changing transforms, but returned 0 mesh objects. It therefore did not pass visible plant-animation acceptance. No skinned or botanical animation export was tested.

## Choose the boundary you can validate

The [format fixture download](https://digitalgarden.si/marketing/research/comparisons-20261020/assets-tests.zip) includes the owned source generator, portable scripts, accepted files, loader checks and compact results. It also documents the rejected export paths. Bundle costs are measured separately in the receipt; a loader's dependency graph is distinct from asset bytes and from an already-cached application bundle.

For a Houdini authoring pipeline, keep composition and variants where they are needed, then test the specific browser representation that will ship. For repeated identical plants, an explicit runtime instancing step can matter more than the container extension. This test does not cover Quick Look on a physical iPhone, MaterialX, every USD schema, total GPU memory or production network speed. Two NVIDIA packages later followed a different route, [USD to GLB in Blender and Houdini](https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/), where PreviewSurface textures, a geometry-cache workaround, direct loading and a usd2gltf failure were separate outcomes. Continue with [OpenUSD pipeline basics](https://digitalgarden.si/blog/openusd-pipeline-beginners/), [the earlier USD web-viewer test](https://digitalgarden.si/blog/openusd-threejs-usd-web-viewer/) and [measured glTF optimization](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/).

[^1]: Three.js, [USDLoader](https://threejs.org/docs/pages/USDLoader.html), checked 20 October 2026. The pinned 0.186.1 source, rather than a historical USDZ-only assumption, defines this test.
