# Houdini Copernicus to GLB: Nine Bakes and 18 Browser Starts

> Nine Houdini Copernicus bakes and 18 browser starts test normal maps, ORM, glTF 2.0 export repairs and PNG versus KTX2 on an RTX 5070 Ti.

- Author: Alicia Lu
- Published: 2026-11-02
- Canonical HTML: https://digitalgarden.si/blog/houdini-copernicus-bake-glb-threejs/

I baked a 55,296-triangle plant fixture onto a 1,056-triangle mesh in Houdini Copernicus, then checked the GLBs in Three.js. The useful result was not a bigger FPS number. It was discovering what the bake, the exporter and the browser each did to the asset.

Nine native map-set writes took median **0.762 s at 512, 1.217 s at 1024 and 3.026 s at 2048**. Those are render-to-disk wall times, including native preparation and PNG encoding, not isolated ray-tracing times. The corrected 1K PNG GLB was **3,019,048 bytes**, larger than the **1,571,676-byte high mesh**. A KTX2 derivative cut that to **645,788 bytes**, while adding a transcode step during loading.

There was also an export problem worth catching before publication: in my Houdini 22.0.429 `gltf::2.0` output, this Principled Shader setup produced a three-component `TANGENT`, zero normal scale and zero occlusion strength. I preserved that original file and tested a separate, explicitly repaired web copy. A successful ROP cook was not enough.

*Measured on 2 November 2026. Codex assisted the experiment, analysis and writing. This is an owned parametric mesh-leaf fixture, not a migration of the production Digital Garden or a test of alpha-card foliage.*

![Actual WebGPU renders of the untouched native GLB and the separately corrected web copy, with the same camera and lights](https://digitalgarden.si/marketing/research/copernicus-20261102/hero.png)

The left image is the native export; the right includes the documented material and tangent repairs. Both retain the 1,056-triangle low mesh. These are actual browser captures, not a Houdini viewport substituted for the browser result.

## What I kept fixed

The workstation was Windows 11, Ryzen 9 9950X, 64 GB RAM and an RTX 5070 Ti with 16,303 MiB reported by `nvidia-smi`, driver 617.42. I used Houdini **22.0.429**, Bake Geometry Textures **2.0**, OptiX v9.0, Three.js **0.186.1** and Chrome **156**.

The plant is a deliberately small, self-owned fixture: three opaque mesh leaves and a stem, four separate UV regions and procedural high-resolution veins. The high mesh has 55,296 triangles; the low mesh has 1,056. I generated both from the same parameterization rather than decimating a production asset. The low mesh had final point normals, triangulated faces and vertex UVs before PolyFrame generated MikkT tangents, bitangents and signs.

For the primary bake I used Surface Normal tracing, 64 AO samples, a 0.15-unit AO distance limit, primitive `name` piece matching, UV-boundary filling and 0.008 padding. Matching pieces isolates the leaf/stem regions, so this AO is not a claim about every neighboring leaf casting occlusion in a full plant canopy.

Each size ran three times in a fresh `hython` process, with size order rotated between rounds. Geometry was cooked before the timer. The timed operation wrote normal, AO, packed ORM and a constant base-color image. I did not clear driver or OS caches, and the first setup experiments had already initialized parts of the workstation's graphics stack.

## The native bake and the four outputs

[SideFX's Bake Geometry Textures 2.0 documentation](https://www.sidefx.com/docs/houdini/nodes/cop/bakegeometrytextures-2.0) describes the low/high/cage inputs and the normal/tangent requirements. My primary network used SOP Import nodes for low and high, then the baker's normal and occlusion outputs.

I connected AO to the red input of a **Channel Join** COP, a mono constant of 0.65 to green and a mono constant of zero to blue. The resulting ORM therefore stores **occlusion in R, roughness in G and metallic in B**. The 1K file's measured channel ranges were R=203–255, G=166 and B=0. These are actual exported pixel values, not a diagram of intended wiring.

Separate ROP Image Output nodes wrote the selected ports. Normal PNGs were 16-bit; ORM and constant color were 8-bit. Normal and ORM used Raw color conversion. The color image used the explicitly selected sRGB output conversion. I tested individual output files, not an unverified all-AOV `<LAYER>` recipe.

| Square size | Three-run median | Full observed range |
| --- | ---: | ---: |
| 512 | 762.3 ms | 762.1–781.9 ms |
| 1024 | 1,216.8 ms | 1,213.5–1,232.6 ms |
| 2048 | 3,025.7 ms | 2,992.7–3,042.8 ms |

Here “three-run median” means the median of three complete map-set operations; each operation writes four maps. The first normal request includes shared native work, so adding the later AO write time does not isolate an independent AO kernel cost.

![Nine measured Copernicus map-set writes at 512, 1024 and 2048, with all three samples shown for each size](https://digitalgarden.si/marketing/research/copernicus-20261102/bake-times.png)

Two setup attempts failed because Add AOVs from Input left the Constant COP's AOV name empty. The fix was to set that output name explicitly. Those failed folders and the later successful export pilot remain separate from the nine formal cells.

## Why I inspected the GLB before judging the bake

Houdini 22 has both old and new glTF implementations; [SideFX's import/export overview](https://www.sidefx.com/docs/houdini/io/gltf.html) distinguishes them. The exporter in this experiment was **`gltf::2.0`**, not an assumed legacy node. The material was a Principled Shader with file textures assigned through `shop_materialpath`.

The native 1K GLB contained all four texture images, but inspection found:

| Field | Native file in this fixture | Explicit web-copy treatment |
| --- | --- | --- |
| TANGENT | VEC3; validator error | VEC4 with the verified sign |
| normalTexture.scale | 0 | 1 |
| occlusionTexture.strength | 0 | 1 |
| Roughness | 0.65 factor and a roughness map | Factor 1 with ORM G |
| ORM images | Separate roughness and occlusion images | One shared ORM texture |
| Alpha / sides | BLEND, single-sided | OPAQUE, double-sided mesh leaves |

The native material parameters for normal and AO intensity were both 1 in the reopened HIP. The exported zero values therefore cannot be explained by me intentionally disabling those maps in the source material. This is an observed result for this pinned build and setup, not a verdict on every Houdini material or exporter version.

For the tangent repair I reopened the HIP, recooked `LOW_MIKKT` and checked all **3,168 vertex signs**: every sign was +1. Only that fixture-specific evidence justified appending +1 to its exported tangent vectors. A mesh with mirrored UVs needs its real handedness or regenerated MikkTSpace tangents; blindly appending +1 is not a general repair.

The reproducible repair script also set the intended material factors and shared texture references, and removed unused textures. It optimized the constant color texture into a factor. Consequently the corrected asset does not have a base-color texture to label sRGB; in the separate native-loader check the base texture was sRGB and normal/ORM were data textures with NoColorSpace.

The corrected PNG variants passed the validator without errors. The untouched native file's tangent error remains in its own report. The installed validator does not understand `KHR_texture_basisu` fully and emits KTX2 format warnings; I keep those warnings and verify actual compressed-texture loading in both browser backends.

## Green Up, Green Down and a cage that cooked but missed

I baked a second 512 map with Green Down and compared both directions against the high mesh from three fixed cameras on each backend. AO was disabled for these direction comparisons. The +Y bake was closer in all six common-foreground comparisons: RGB RMSE **12.37–13.35**, versus **13.95–15.81** for the -Y control, on an 8-bit screenshot scale.

That is a bounded shading check, not perceptual equivalence. The low mesh still changes silhouette and fine geometry. The common-foreground mask also excludes non-overlapping outlines; it cannot establish that a normal map restores the original outline.

![High mesh, Green Up bake and Green Down control from the same WebGPU camera; AO is disabled for the comparison](https://digitalgarden.si/marketing/research/copernicus-20261102/normals.png)

The cage experiment was a useful failure. Simply inflating the low mesh and supplying it as the new baker's cage input produced a flat normal map and white AO, despite a successful cook. I rejected that as a useful bake.

Using the native **Bake Pre-Process** COP, with its low/high/cage outputs and explicit ray offsets, recovered non-flat normal data. AO stayed uniformly white in that control, so I did not replace the primary Surface Normal results with it or claim the cage was better. The [Bake Pre-Process documentation](https://www.sidefx.com/docs/houdini/nodes/cop/bakepreprocess.html) explains why ray attributes matter in this workflow.

## Geometry savings and texture costs are separate budgets

| Prepared browser asset | Mesh triangles | GLB bytes |
| --- | ---: | ---: |
| High reference | 55,296 | 1,571,676 |
| Low, 512 PNG maps | 1,056 | 947,652 |
| Low, 1024 PNG maps | 1,056 | 3,019,048 |
| Low, 2048 PNG maps | 1,056 | 8,538,088 |
| Low, 1024 KTX2 maps | 1,056 | 645,788 |

The 1K PNG asset is larger than the high mesh because the detailed normal image dominates this small fixture. A smaller triangle count did not make the texture payload disappear.

![Prepared GLB file sizes showing the larger 1K and 2K PNG variants and the smaller KTX2 derivative](https://digitalgarden.si/marketing/research/copernicus-20261102/payloads.png)

I used the pinned `gltfpack` image encoder for a UASTC derivative, then transplanted only the encoded normal and ORM images into the corrected original document. Its optimized geometry and materials were discarded. The constant color factor stayed as-is. The first attempt expected a base-color texture that the encoder had optimized away; that failed copy attempt is documented.

Both browser backends selected BC7. The two compressed textures reported **2,796,256 bytes of mip payload**, versus an **11,184,812-byte RGBA8-with-mips estimate** for two 1K textures. Those are texture payload/arithmetic figures, not driver-measured VRAM, total GPU memory or an assertion that 16-bit PNG precision survives browser decoding. UASTC is also not a promise of lossless image equivalence.

## Eighteen browser starts: smaller did not mean faster loading

I tested high, corrected 1K PNG and corrected 1K KTX2 on WebGPU and the forced WebGL2 backend of `WebGPURenderer`, three fresh-profile Chrome processes per cell. This is not a legacy `WebGLRenderer` comparison. Each process used 1280×720, DPR 1, the same camera/lights and no production shadows or post-processing stack.

| Backend / asset | Load + decode median | First GPU completion median | Total median / full range |
| --- | ---: | ---: | ---: |
| WebGPU high | 24.1 ms | 76.0 ms | 101.0 / 100.2–105.1 ms |
| WebGPU 1K PNG | 50.1 ms | 122.3 ms | 172.6 / 168.8–173.9 ms |
| WebGPU 1K KTX2 | 107.9 ms | 86.6 ms | 194.9 / 179.3–207.1 ms |
| WebGL2 high | 16.3 ms | 56.7 ms | 73.1 / 72.9–86.3 ms |
| WebGL2 1K PNG | 53.1 ms | 71.6 ms | 120.4 / 118.2–126.6 ms |
| WebGL2 1K KTX2 | 107.0 ms | 62.2 ms | 169.8 / 160.0–169.8 ms |

Column medians are calculated separately; they are not numbers to add together. Load includes local transfer, parsing, image decode or KTX2 transcoding. Completion uses a WebGPU queue fence or `gl.finish()`, not a monitor presentation timestamp. Fresh profiles do not guarantee cold OS/driver caches. Three repeats on one machine do not establish a universal backend or codec ranking.

All 18 cases actually drew the model. Counters included one mesh draw plus the renderer's output pass: two actual calls and either 55,297 or 1,057 submitted triangles. The extra triangle is the output pass. Across the subsequent 2,160 formal RAF intervals, none exceeded 50 ms; because that window starts after preparation, it is not evidence that startup had no waits.

For this fixture I would keep the exporter inspection and use the corrected 1K asset as a comparison point. I would not call the baked plant automatically faster, smaller at every map size, or visually identical to the high mesh. Alpha cards, real phones, a second GPU/browser, banding/dither fixes and production integration remain untested.

## Data and reproduction

[Reproduction ZIP](https://digitalgarden.si/marketing/research/copernicus-20261102/copernicus-study-20261102.zip) · [Analysis and ranges](https://digitalgarden.si/marketing/research/copernicus-20261102/analysis.json) · [Native CSV](https://digitalgarden.si/marketing/research/copernicus-20261102/results.csv) · [Browser CSV](https://digitalgarden.si/marketing/research/copernicus-20261102/browser-results.csv) · [Source hashes](https://digitalgarden.si/marketing/research/copernicus-20261102/source-manifest.json) · [Environment and limits](https://digitalgarden.si/marketing/research/copernicus-20261102/ENVIRONMENT.md).

The package contains the nine formal native receipts, 18 formal browser receipts, separate visual/control records, prepared GLBs, an editable self-contained HIP with its maps, selected real captures and regeneration scripts. Browser profiles and unrelated production assets are excluded. Original failed setup, naive cage, native tangent validation and encoder-copy limitations are retained rather than relabeled as successful final measurements.

For the broader delivery choices, see the earlier [Houdini glTF optimization test](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/), [Draco/Meshopt/KTX2 measurements](https://digitalgarden.si/blog/draco-vs-meshopt-ktx2-benchmark/) and [vegetation LOD study](https://digitalgarden.si/blog/threejs-webgpu-vegetation-lod/). This experiment does not change their data or dates. The [rendering field guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/) links the related asset, format and renderer work.

Code, fixture, figures and data are by Alicia, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/); dependencies retain their own licenses.
