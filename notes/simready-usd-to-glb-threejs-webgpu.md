# SimReady USD to GLB: Blender and Houdini Export Traps, Tested in WebGPU

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Two NVIDIA SimReady assets tested in WebGPU: Blender visibility fixes, a Houdini geometry-cache workaround, direct USD loading and usd2gltf failures.

- Author: Alicia Lu
- Published: 2026-10-23
- Updated: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/

My first GLB opened successfully and still looked wrong. A pair of glasses had doubled arms, and its triangle count was exactly twice the source count. Passing a file-format validator was not enough: I had exported Blender's hidden USD prototype objects along with their visible instances.

For these two NVIDIA SimReady assets, Blender preserved PreviewSurface texture networks after two export corrections. Houdini produced valid geometry and vertex colors after a geometry-cache workaround, but no exported PBR materials or textures. The original USD files also loaded directly in a Three.js WebGPU scene. usd2gltf failed on material connections. These are different levels of preservation, not four interchangeable conversion buttons.

![Visible-only Blender export of the NVIDIA glasses, rendered in WebGPU](https://digitalgarden.si/marketing/research/daily-20261023/eyeglasses-blender-visible.jpg)

Tested locally with Codex on 23 October 2026. [Download the browser and validation results](https://digitalgarden.si/marketing/research/daily-20261023/simready-results.json).

## What I actually downloaded

I pinned [NVIDIA's SimReady dataset](https://huggingface.co/datasets/nvidia/simready-assets/tree/524747d9b5e1f21b5b18b56041788c01d745f1a1) at `524747d9b5e1f21b5b18b56041788c01d745f1a1` and downloaded `accessory_eyeglasses_a01` and `IV_pole_a01`, preserving their relative directory structures. The receipt contains SHA-256 hashes for 92 downloaded files totaling 573,930,888 bytes, excluding thumbnails. That is the complete selected package download, not the browser's transferred payload.

Both samples use payloads and composition; neither is a self-contained single-file example. Their root stages declare metres and Z-up. The composed stages contain both MDL materials and UsdPreviewSurface networks. Calling the packages “MDL only” would have predicted the wrong failure mode.

| Source | Eye glasses | IV pole |
| --- | ---: | ---: |
| Visible mesh prims, including instance proxies | 4 | 16 |
| Triangles | 42,168 | 458,714 |
| Instance prims in composed traversal | 11 | 60 |
| Source size, X/Y/Z metres | 0.144580 / 0.174321 / 0.056924 | 0.722052 / 0.570911 / 1.799375 |

Instance prim counts are USD hierarchy observations, not GPU instance counts or numbers of separate props.

## The four paths and their actual outcomes

| Path | Observed outcome |
| --- | --- |
| Blender 5.2.0 LTS, stock USD importer | Imported geometry and PreviewSurface texture networks. Default GLB export needed two corrections described below. Saved `.blend` files were reopened. |
| Houdini 22.0.429, USD File Import to glTF ROP | Imported polygons, normalized axes, saved `.bgeo.sc`, reopened through a File SOP and exported GLB. Both `.hip` files reopened; accepted raw and optimized GLBs passed validation and matched browser triangle counts. Vertex colors survived; PBR materials and textures did not. |
| usd2gltf 0.3.5 with usd-core 26.11 | Both conversions exited with `TypeError: 'NoneType' object is not subscriptable` while resolving a surface connection in `usd_material.py`. No usable GLB was produced. |
| `@huggingface/usd-web-viewer` 0.1.1 `loadUsd`, Three.js 0.186.1 | Both original composed USD stages rendered through WebGPU. The loader reported 4 / 16 meshes, 42,168 / 458,714 triangles, no missing layers and no warnings; 6 / 15 textures finished with zero failed textures. |

An initial local Houdini license timeout was resolved during the session, so it is not the final result. At cleanup, the standalone hython process crashed during exit; saved artifacts still matched their pre-exit hashes. That limits any claim of process stability, but does not replace the saved-file and browser checks. I did not install an Omniverse material mapper or patch usd2gltf. [Houdini's USD File Import documentation](https://www.sidefx.com/docs/houdini/nodes/sop/usdimport.html) distinguishes packed USD selection from polygon unpacking. The direct loader's behavior and options are documented in the [USD Web Viewer package](https://www.npmjs.com/package/@huggingface/usd-web-viewer).

## The two Blender corrections

First, `export_extras=True` failed with `Out of range float values are not JSON compliant: inf`. Exporting custom properties encountered a non-finite value. I preserved that failed attempt and used `export_extras=False` for the visual delivery.

Second, exporting every object included hidden prototype objects created by the USD importer. The glasses GLB drew 84,336 triangles and overlapping arms. Restricting the export to visible and renderable objects restored the expected 42,168 triangles and pose. The IV pole also matched its source count after filtering.

```python
bpy.ops.wm.usd_import(filepath=source_usd)
bpy.ops.export_scene.gltf(
    filepath=derived_glb,
    export_format='GLB',
    export_extras=False,
    use_visible=True,
    use_renderable=True,
)
```

This is a visual export. Disabling extras does not preserve the USD physics schemas, joints, semantic metadata or composition model. Those remain in the untouched USD source. Copying a few labels into JSON would not make the GLB simulation-ready.

![Rejected Blender baseline with overlapping hidden prototype geometry](https://digitalgarden.si/marketing/research/daily-20261023/eyeglasses-blender.jpg)

## Houdini: validate before optimizing

The successful node path was `USD File Import → Transform → .bgeo.sc snapshot → File SOP → glTF ROP`. I imported boundable prims with proxy/render purpose, unpacked to polygons, used the source's metre scale and rotated Z-up to Y-up. The source files were not changed.

Two earlier paths failed acceptance. Exporting the live imported SOP produced a 196-byte scene with no geometry. Exporting the object produced larger files, but glTF Validator found 587 / 26,515 out-of-bounds index errors. Optimization alone removed those reported errors while changing the data; that was not a trustworthy repair. I rejected those files and used the geometry snapshot path. Its **raw files already passed with zero errors and zero warnings**, before compression.

| Accepted Houdini snapshot export | Eye glasses | IV pole |
| --- | ---: | ---: |
| Raw GLB bytes | 1,664,008 | 18,333,196 |
| Optimized GLB bytes | 254,296 | 2,259,324 |
| Browser triangles | 42,168 | 458,714 |
| glTF materials / images | 0 / 0 | 0 / 0 |

The browser rendered one mesh per asset with vertex colors, using a loader-supplied default material. The largest bounding-box extent differences versus direct USD were approximately 0.0125 mm and 0.0329 mm after compression. These files are smaller partly because they omit the texture/material payload; comparing their bytes to the textured Blender files as an efficiency victory would be misleading. The [glTF ROP documentation](https://www.sidefx.com/docs/houdini/nodes/out/gltf.html) describes supported Houdini material export; USD material-path attributes alone did not reconstruct those materials in this test.

![Accepted Houdini snapshot export with vertex colors and no exported PBR material](https://digitalgarden.si/marketing/research/daily-20261023/eyeglasses-houdini.jpg)

![The NVIDIA IV pole exported through the Houdini snapshot path and rendered in WebGPU](https://digitalgarden.si/marketing/research/daily-20261023/ivpole-houdini.jpg)

## Smaller Blender GLBs, with the tradeoffs stated

I applied the same derived-file recipe to both successful Blender exports: deduplicate, weld, prune unused data, meshopt compression and quantization; textures resized to at most 1024 pixels and encoded as lossless WebP. No triangle simplification was requested. “Lossless WebP” describes the encoding after resizing: reducing 4096-pixel images and converting 16-bit channels to 8-bit is not lossless relative to the source.

| Visible-only result | Eye glasses | IV pole |
| --- | ---: | ---: |
| Raw GLB bytes | 81,662,964 | 210,674,944 |
| Optimized GLB bytes | 2,454,060 | 8,493,172 |
| Browser triangles | 42,168 | 458,714 |
| Browser mesh objects after material splitting | 7 | 44 |
| Largest difference in bounding-box extent vs direct USD | 0.0082 mm | 0.0997 mm |

Both optimized files passed glTF Validator with zero errors. There were 7 and 28 warnings, mainly missing authored tangent space; the validator also cannot validate meshopt's compressed payload. I therefore also loaded the files with `GLTFLoader` plus `MeshoptDecoder`, checked the actual geometry and inspected the renders. The dimensional comparison is an axis-aligned bounds check, not a complete per-vertex Hausdorff test.

An additional reproducibility trap appeared in the optimization environment: glTF Transform CLI 4.3.0 and a transitive dependency loaded different Sharp versions, and texture conversion failed with a libvips colourspace error. The accepted script uses a single direct Sharp 0.35.5 encoder and the resolved glTF Transform SDK 4.5.1. Its package lock and script are preserved in the local experiment archive; I did not change the production site's dependencies.

![The visible-only Blender IV pole with its preview textures in WebGPU](https://digitalgarden.si/marketing/research/daily-20261023/ivpole-blender-visible.jpg)

## Materials: preview is a meaningful limit

The viewer explicitly reported `materialKinds.preview` for all 7 / 44 source material networks. Base-color and normal textures were visible in the GLB and direct USD versions. The glasses' lenses remained opaque in this preview path. That is not evidence of faithful MDL glass, transmission or all Omniverse shading behavior.

![Original composed USD glasses loaded directly in WebGPU with PreviewSurface shading](https://digitalgarden.si/marketing/research/daily-20261023/eyeglasses-usd.jpg)

The paths were viewed in the same small test scene with a hemisphere and directional light. Houdini's vertex-color-only output visibly differed from the textured PreviewSurface versions. This is a conversion inspection, not the production garden or a matched Omniverse render. I did not add these NVIDIA props to the published flower garden.

Recorded local load-to-completed-frame observations were 879 ms / 1,564 ms for direct USD and 75 ms / 403 ms for the corrected GLBs. These were single exploratory loads with different caching and dependency state; they are retained in the evidence, **not used as a speed ranking**. The glasses also completed a ten-second camera orbit at about 59.9 FPS on this display-capped desktop.

The test machine was Windows, Ryzen 9 9950X, RTX 5070 Ti and Chromium 155. The inspection canvas was 1265×520 at renderer DPR 1. No physical phone, AR/Quick Look, collision simulation or MDL reference render was tested.

## What I would use next

For these textured visual web props, I would keep the composed USD as the master and ship a reviewed visible-only Blender GLB. Houdini's snapshot path is useful when I want geometry and vertex colors for a custom web material; a textured delivery still needs material reconstruction and review. For examining the original asset hierarchy in a browser, direct `loadUsd` was already useful on both samples. I would test glass against a proper reference before calling its appearance preserved.

The practical checkpoint is simple: compare the visible render, triangle count and physical dimensions to the source. An export button and a valid file header are only the beginning.

## Asset credit and evidence

Added 24 October: the [controlled plant lab](https://digitalgarden.si/marketing/research/followup-20261024/followup-lab.zip) includes `inspect-gltf.mjs`, a reusable read-only preflight. After installing the pinned dependencies, run `node inspect-gltf.mjs path/to/asset.glb`. It validates JSON and glTF, reports material/image counts and custom attributes, and flags prototype-like node names for review. It does not automatically delete nodes or establish source-visibility and material fidelity. The [Geometry Nodes export test](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/) includes a valid GLB that nevertheless lost its intended colors.

`accessory_eyeglasses_a01` and `IV_pole_a01` © NVIDIA, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), from the pinned dataset above. Modified for this experiment by Alicia with Codex: USD-to-GLB conversion, visibility filtering, texture resizing and compression. The package license files and original sources are preserved.

[Machine-readable results](https://digitalgarden.si/marketing/research/daily-20261023/simready-results.json) contain accepted file sizes, validator counts, rendered triangles and bounds. The failed baseline is shown above; the original USD and native authoring files were preserved locally. The published screenshots retain the asset credit and modification notice.

Continue with the [Three.js rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/) or [visit the garden](https://digitalgarden.si/play/).
