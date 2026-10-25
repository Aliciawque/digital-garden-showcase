# Houdini to Three.js: Testing glTF Compression and Instancing

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Measured Houdini GLB, Draco, Meshopt and KTX2 results: file size, texture memory, attribute survival and 1,000 plant draw calls in Three.js.

- Author: Alicia Lu
- Published: 2026-10-19
- Updated: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/

For this Houdini to Three.js test, texture compression changed file size far more than geometry compression. The larger catch was semantic: the tested gltfpack outputs lost the custom attributes needed by wind. I measured export, browser loading and repeated placement separately so a small GLB could not silently pass as a usable plant.

The renderer, format and splat notes that build on this export test are collected in [the full rendering guide for the interactive 3D garden](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

The fixture is an original procedural grape hyacinth built for this experiment, with 6,056 triangles, one material and an intentionally noisy 2048 × 2048 PNG test texture. It is not a production garden asset or a representative survey of plant textures. Codex executed the scripts and browser checks on 19 October 2026.

![The original procedural plant loaded in the browser with its test texture](https://digitalgarden.si/marketing/research/webgpu-series-20261019/plant-raw.png)

## Start with the native export

Houdini 22.0.429 used the `gltf::2.0` ROP. A single quad exported as two triangles without a separate Divide SOP. Ten packed copies exported as ten meshes and ten nodes, totaling 60,560 triangles. Packing inside Houdini did not create `EXT_mesh_gpu_instancing` in this output. If the website needs an InstancedMesh, create and verify that runtime representation explicitly.

Custom point attributes used the underscore prefix required by the exporter. The raw GLB contained `_STIFF` and `_PHASE` alongside position, normal, color and UV attributes.[^1] An initial validator failure exposed zero length normals at degenerate leaf endpoints. I fixed the generator’s normal fallback, exported again and validated the final candidates. That repair is part of the tested result; the first export was not accepted.

## File size and attribute survival

The native Draco export used Houdini. The other candidates used the official Windows gltfpack 1.3.0 binary or glTF Transform CLI 4.5.1. Sizes are actual file bytes, shown as decimal MB. Every final GLB passed the glTF validator with zero errors; nonblocking unused data and extension warnings are retained in the evidence.

| Candidate | File MB | Triangles | Wind attributes survive? | Median local fetch to submitted frame |
| --- | --- | --- | --- | --- |
| Native GLB + PNG | 4.438 | 6,056 | Yes | 65.7 ms |
| Native Draco + PNG | 4.367 | 6,056 | Yes | 49.8 ms |
| gltfpack Meshopt + PNG | 4.191 | 6,056 | No | 53.0 ms |
| gltfpack Meshopt + KTX2, 2048 | 0.793 | 6,056 | No | 33.4 ms |
| gltfpack Meshopt + KTX2, 1024 | 0.238 | 6,056 | No | 16.7 ms |
| glTF Transform Meshopt + PNG | 4.208 | 6,056 | Yes | 55.6 ms |
| Native vertex color, no texture | 0.290 | 6,056 | Yes | 16.7 ms |

The browser used Three.js 0.186.1, its Draco and Basis decoders, and MeshoptDecoder on WebGPU. Each row used five distinct local HTTP URLs. Timing covered fetch, parse, decode, compile and the first submitted frame. Decoder code and GPU pipelines were warm. These are localhost observations with frame scheduling granularity, not production network, device cold start or GPU completion timings. A separate WebGL2 check also loaded the 2048 KTX2 candidate and reported the same 12 BC7 mip levels and 5,592,432 uploaded bytes.

Draco reduced this texture dominated file by only about 1.6%. KTX2 plus a smaller texture had a much larger effect, but those gltfpack candidates failed the wind attribute requirement. The `-kv` experiment did not rescue the unknown attributes. The glTF Transform Meshopt candidate retained them, using generic attribute quantization at 12 bits. Attribute presence still does not guarantee sufficient numerical precision for every shader; inspect the decoded values and appearance before release. A later [Three.js GLB loading](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/) test separated cache, decoding and the first GPU frame on a texture-free plant. Meshopt reduced that file from 319,648 to 130,296 bytes with identical decoded attributes, and the fresh-URL first-frame median fell from 35.7 ms to 17.6 ms. Those twenty rows use a different protocol from this warm, texture-dominated table.

## Commands and their limits

The tested gltfpack commands were:

```sh
gltfpack -i plant-raw.glb -o plant-pack.glb -cc -ce ext
gltfpack -i plant-raw.glb -o plant-ktx2.glb -cc -ce ext -tc -tj 4
gltfpack -i plant-raw.glb -o plant-1024.glb -cc -ce ext -tc -tj 4 -tl 1024
```

The tested texture size option was `-tl 1024`. These commands use `EXT_meshopt_compression` explicitly and the binary’s built in texture encoder.[^2] I did not install or benchmark a separate toktx conversion. The alternate Meshopt command was:

```sh
gltf-transform meshopt plant-raw.glb plant-transform.glb --level high
```

The vertex color candidate removed the image entirely and encoded phase/stiffness into red/green channels. It is a data transport probe, not an accepted visual replacement: a material that displays those channels as ordinary vertex color will tint the plant. A deliberate vertex color palette can suit stylized plants, but it cannot preserve details that existed only in the texture. A later [Houdini versus Geometry Nodes plant export](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/) kept three vertex-color groups on one shared material. The first Blender Geometry Nodes file wrote a valid GLB that had lost its green stem and foliage colors; the corrected export kept all three groups.

## Uploaded texture data is a separate budget

The 2048 PNG expands to an estimated 22,369,620 bytes for an RGBA8 texture and full mip chain. The actual BC7 KTX2 mip arrays totaled 5,592,432 bytes at 2048 and 1,398,128 bytes at 1024. Those totals describe uploaded mip data. They do not measure total process VRAM, decoder scratch space, staging allocations or driver overhead.

The visual check matters as much as this arithmetic. Downsizing the artificial high frequency texture changes its fine detail; the smaller candidate should not be described as visually identical. Real foliage needs its own close and distant views, especially around alpha edges.

## One thousand plants: count the draws

A second test repeated the plant 1,000 times at 1280 × 720, DPR 1, with a 30 second orbit after warmup. All paths used the same simple material settings and stayed close to the display’s 60 Hz ceiling on the RTX 5070 Ti.

| Runtime representation | Total render calls | Total triangles |
| --- | --- | --- |
| InstancedMesh, LOD0 | 2 | 6,056,001 |
| BatchedMesh, LOD0 | 1,001 | 6,056,001 |
| Merged geometry, LOD0 | 2 | 6,056,001 |
| InstancedMesh, all plants at LOD1 | 2 | 2,116,001 |

The totals include a final output pass, which adds one call and one triangle. In the pinned WebGPU backend, the tested BatchedMesh path loops over draw commands. Its class name did not imply a single GPU draw. Merging reduced calls but duplicates vertex data; instancing shares geometry. The LOD row replaces the whole patch with its authored lower detail model, not a per instance LOD algorithm. The same question, across classic WebGL and the node renderer, is in the [nine-path WebGPU and WebGL comparison](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/).

The practical acceptance rule is to check file size, decoded attributes, material appearance and runtime counters together. A later [Houdini USD to GLB](https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/) test made the same distinction on NVIDIA SimReady geometry: optimizing a broken export cleared the reported index errors only by changing the mesh, so the untouched geometry snapshot was the file that passed first. The [small reproducible experiment](https://digitalgarden.si/marketing/research/webgpu-series-20261019/webgpu-garden-tests.zip) includes the measurements and owned generator. Continue with [procedural plants and wind](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/) or the [complete garden pipeline](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/). Production models were not replaced by these test outputs.

[^1]: SideFX, [glTF ROP](https://www.sidefx.com/docs/houdini/nodes/out/gltf.html), including custom attribute and export behavior.
[^2]: meshoptimizer, [gltfpack documentation](https://meshoptimizer.org/gltf/). Also see the official [glTF Transform CLI](https://gltf-transform.dev/cli). Sources checked 19 October 2026.
