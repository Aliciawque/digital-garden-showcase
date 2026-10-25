# ComfyUI 3D Workflow: From a Million-Triangle Draft to a DCC-Tested Prop

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A real TRELLIS.2 workflow in ComfyUI: generate, remesh, decimate, unwrap and bake a prop, then check its GLB, textures and scale in Blender and Houdini.

- Author: Alicia Lu
- Published: 2026-10-19
- Updated: 2026-10-20
- Canonical HTML: https://digitalgarden.si/blog/comfyui-3d-workflow/

**A useful ComfyUI 3D workflow needs an acceptance stage after generation.** In this test, TRELLIS.2 produced a 1,175,882-triangle mesh from the official example image. Remeshing, decimation, UV unwrapping and texture baking reduced the deliverable to 19,738 triangles in a 4.55 MB textured GLB. Native Blender and Houdini imports then exposed a second problem: an embedded texture that worked in Blender was not directly usable by this Houdini SOP's generated material.

The test ran on ComfyUI 0.37.0, frontend 1.52.7, PyTorch 2.10.0.dev20251209+cu130 and an RTX 5070 Ti with 16 GB VRAM on Windows. The DCC checks used Blender 5.2.0 LTS and Houdini 22.0.429. This was a local TRELLIS.2 run, not a partner-node credit purchase or a comparison with Meshy, Rodin or Hunyuan. The separate [Meshy Houdini cleanup](https://digitalgarden.si/blog/meshy-3d-agent-houdini-cleanup/) follows that other download path: a free Meshy 6 Lite GLB, a rejected cleanup, and a 3,000-triangle instance in the browser. Mesh generation is not the only local model experiment here. [ML volume upres for Pyro](https://digitalgarden.si/blog/houdini-ml-volume-upres-pyro/) applies an ONNX model to cached Houdini smoke instead of turning an image into triangles.

![Actual generated axe before cleanup on the left and the reduced textured result on the right, imported and rendered in Blender](https://digitalgarden.si/marketing/research/briefs-20261019/comfy-raw-clean.png)

The raw shape on the left is shown with neutral clay; the reduced result on the right has the generated and baked PBR material. The camera, lighting and relative display scale are shared. Because the materials differ, this image demonstrates the delivered stages rather than serving as a controlled shading-quality comparison.

## Start with the official graph and one input

I used Comfy's published **Pixal3D & TRELLIS.2: Image to Model** template and its `viking_wolf_rune_axe.png` input.[^1] That keeps the starting asset and graph identifiable. The axe is a workflow fixture, not a production garden asset or an original character design for sale.

The graph was adapted to select TRELLIS.2 and keep the 512-resolution shape stage. The 1024/1536 upsample branch was omitted for this 16 GB test. This matters: the result should not be described as the unmodified highest-resolution template output.

The active generation path was:

1. Load the input and remove its background with BiRefNet.
2. Center/crop the subject and obtain DINOv3 image conditioning.
3. Sample the sparse structure at 12 steps.
4. Decode the structure and sample shape at 20 steps.
5. Decode the raw mesh.
6. Sample the texture stage at 12 steps and decode the material voxels.

All three sampler seeds were fixed to 42. The model was the official `trellis_2_int8_convrot.safetensors` distribution, with the shape VAE, texture VAE and DINOv3 encoder identified by the template. The exact submitted API prompt and model hashes are included with the test evidence.

## Save the raw mesh before improving it

The raw GLB was 20,790,420 bytes. Blender counted 1,175,882 triangles and no UV layer. Houdini read the same primitive count. A default material slot existed, but the raw shape export did not contain the finished baked texture set.

Do not equate the vertex count across importers. Houdini counted 556,025 points in the raw input, while Blender's import with Merge Vertices enabled yielded 554,859 vertices. The triangle count agreed. Importers may weld or split vertices around normals, UV seams and other attributes; report both the importer and its settings when comparing these figures.

The raw asset is useful as a shape/detail source. It is much heavier than the reduced static-prop target, and it still needs UV and texture delivery. That is why a visually successful preview is only the beginning of the acceptance process.

## The cleanup chain that actually ran

| Stage | Local settings or result |
| --- | --- |
| Remesh Mesh | UDF, resolution 256, two smoothing iterations |
| Decimate Mesh | Requested 20,000 faces; output 19,738 triangles |
| Smooth Normals | 180-degree setting from the template |
| Unwrap Mesh | 1024 target atlas; automatic charts |
| Normal bake | From the raw shape onto the reduced mesh |
| Texture bake | Generated base color, metallic and roughness material data |
| Ambient occlusion | Separate bake applied to the delivered material |
| Save GLB | 4,548,732 bytes with UVs and embedded textures |

The exact prompt records all effective inputs, including remaining template values. The requested triangle target is a budget, not a promise of an exact face count. Unwrapping reported 489 charts and an internal atlas of 1039 × 1026 in this run. That is automatic output; it does not establish good artist-authored seam placement.

The first API submission failed at Remesh Mesh because I encoded its dynamic `sign_mode` input as a nested object. The running ComfyUI API expected the template's flat selector and prefixed child widget fields. Restoring that representation fixed execution. This was an API-prompt construction error, not a failed TRELLIS.2 generation.

The successful cleanup/texture submission reported 34.38 s of execution and reused already cached structure and shape nodes. A preceding failed submission had already generated and saved the raw shape. **Do not quote 34.38 s as an end-to-end cold generation benchmark.** Model downloading, startup and cached work are different costs; no isolated peak VRAM measurement was captured.

## Blender: verify the actual exported GLB

Blender imported the reduced GLB with a UVMap and its embedded material. With Merge Vertices enabled it counted 9,792 vertices and 19,738 triangles. The studio render above is from the imported file, not ComfyUI's preview framebuffer.

That distinction catches problems a node preview can hide: missing image payloads, unsupported material channels, a wrong pivot or an exporter that saved a different result from the displayed mesh. At minimum, reopen the exact exported path, inspect the material's image nodes and rotate the prop under a new light.

The visual result is promising as a static prop draft. The wolf-head ornaments, handle ends and small carved details still contain soft or irregular forms. The automatic UV layout has many charts. I would not call it a finished hero prop, a deformation-ready mesh or clean subdivision topology based on this test.

## Houdini: the embedded-texture failure and its fix

Houdini's glTF SOP loaded both meshes without geometry errors. The cleaned GLB exposed point UVs, normals and tangents, but the SOP warned that embedded images could not be used directly as texture filenames for its generated material.

The fix in this experiment was explicit and reproducible: import that same GLB in Blender and export **glTF Separate**, producing a `.gltf`, its binary buffer and external image files. Loading the separate `.gltf` through Houdini's native glTF SOP then returned no import warnings or errors.

The `.gltf` descriptor alone was only 2,202 bytes. That is **not the asset size**: its binary buffer and image files are dependencies. Keep the entire folder together. The original textured GLB remains the convenient single-file web deliverable; the external glTF folder solves this specific Houdini material-import boundary.

After unpacking, I used a uniform transform to make the longest dimension 0.60 m and put the lower Y bound on the ground. Clean and Normal SOPs followed. The final geometry bounds were approximately `0.5994 × 0.6000 × 0.1021` m, with 19,721 primitives after cleanup. The change of 17 triangles is a local cleaning result, not a topology-quality score.

This is one reason to keep the generated raw mesh and a separate cleaned branch. A size or cleanup edit should not destroy the higher-detail source needed for later bakes.

## What I would accept before placing it in a scene

1. The exported file reopens independently in the target application.
2. Triangle count, attributes and every texture dependency are recorded.
3. The object has a deliberate physical size, orientation and pivot.
4. Base color is inspected for baked lighting, and roughness/metallic behave under new lights.
5. Small details survive the target camera distance and the chosen polygon budget.
6. Collision, LODs and performance are checked in the actual destination if those are required.

This experiment completed generation, mesh post-processing, texturing, native imports, a render and a 60 cm geometry cleanup. It did not insert the axe into the production [interactive garden](https://digitalgarden.si/virtual-garden/), build collision or LODs, run animation/deformation acceptance, or compare paid partner models. Those claims require different tests.

For scene packaging after cleanup, the [OpenUSD starter example](https://digitalgarden.si/blog/openusd-pipeline-beginners/) demonstrates references and payloads, while the [Houdini to Unreal test](https://digitalgarden.si/blog/houdini-unreal-usd/) checks units, materials and native instancing. Generation supplies one part; a stable pipeline makes that part reusable.

## Downloads and licensing scope

The [workflow evidence archive](https://digitalgarden.si/marketing/research/briefs-20261019/comfyui-3d-test.zip) contains the exact API prompt, compact measured results and model hashes. The original workflow and example input are linked to their official source rather than presented as my own artwork. The local raw/clean GLBs, external-texture folder and editable DCC scenes are retained for inspection.

Comfy's documentation identifies TRELLIS.2 as the local model for this template.[^1] The upstream project publishes its license in its repository.[^2] Paid partner services and other model families have separate access conditions and terms; this run does not establish their cost, availability or commercial permissions.

## References

[^1]: ComfyUI, [TRELLIS.2 Image to 3D Model workflow](https://docs.comfy.org/tutorials/3d/trellis2), including the official graph, input image and model links; checked 19 October 2026.
[^2]: Microsoft, [TRELLIS.2 repository](https://github.com/microsoft/TRELLIS.2), primary source for the model and license.
