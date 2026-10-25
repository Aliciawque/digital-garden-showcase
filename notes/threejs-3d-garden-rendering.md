# Rendering an Interactive 3D Garden in Three.js: WebGPU, Plants, Formats and Splats

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A guide to the tested Three.js notes behind Alicia's interactive 3D garden: renderer choice, instanced plants, glTF and USD delivery, splats and AI meshes.

- Author: Alicia Lu
- Published: 2026-10-21
- Updated: 2026-10-25
- Canonical HTML: https://digitalgarden.si/blog/threejs-3d-garden-rendering/

This guide collects the notes behind the [interactive 3D garden](https://digitalgarden.si/play/) at digitalgarden.si. The hands-on notes record experiments with pinned versions, owned fixtures and explicit limits. The Gemini note is a separate community case summary. Read them in the order below if you are building a plant scene in Three.js, or jump to the question you have now.

The notes and the open benchmark dataset come from one desktop setup; they are case studies, not general performance claims. Where a result depends on a fixture, a renderer path or a file encoding, the linked page says so.

## Renderer and performance

Start with [how the interactive 3D garden is built](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/). It follows the production garden from Houdini assets to Three.js, with the current code tested on both WebGPU and its WebGL2 fallback, and explains where the scene budget goes.

When the question is [which renderer and submission path to choose for plants](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/), read the controlled 1,000 plant fixture next. It separates the submitted representation, such as separate meshes, InstancedMesh or BatchedMesh, from the API name, and compares CPU render time with the actual draw counters. The first note is a project record; the second is a controlled measurement.

For larger scenes, the [open plant instancing data at 1k, 10k and 100k](https://digitalgarden.si/benchmarks/threejs-webgpu-vs-webgl-instancing/) is a separate v1 dataset: 240 attempts across 48 cells, three renderer paths, static and moving cameras, and per-run CSV and JSON downloads. It uses a longer protocol than the 1,000 plant note, so the two sets of numbers are not pooled. No physical phone was measured.

## Plant assets: procedural, optimized and instanced

For [scattering objects on uneven terrain without floating or clumping](https://digitalgarden.si/blog/threejs-scatter-objects-uneven-terrain/), a separate 70-trial fixture compares plain and BVH rays, weighted surface sampling, hard spacing and native Houdini layouts. Placement accuracy, distribution and render pacing are kept separate.

A plant for the web needs repeatable geometry and data that survives export. The note on [authoring wind-ready plant geometry](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/) builds a small seeded Houdini plant with two levels of detail, custom wind attributes and alpha tested leaf cards.

[Compressing and instancing plant GLBs](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/) measures Draco, Meshopt and KTX2 on that kind of asset, and checks whether the attributes needed by wind survive compression before repeating the plant a thousand times.

Generated meshes need the same scrutiny. [Making a generated mesh safe to instance](https://digitalgarden.si/blog/meshy-3d-agent-houdini-cleanup/) takes a real Meshy 6 Lite download through topology checks and a Houdini reduction before it is placed in the browser.

## Delivery formats: glTF and OpenUSD

For [choosing glTF or USD for browser delivery](https://digitalgarden.si/blog/houdini-usd-vs-gltf-threejs/), the same two-plant pair was exported from Houdini as GLB, USDC and USDZ, then loaded in Three.js. The note records which representation kept both plants, their colors and their wind data.

[Loading a Solaris stage in the browser](https://digitalgarden.si/blog/openusd-threejs-usd-web-viewer/) tests a Houdini Solaris scene in a USD web viewer, with materials, instances, variants, units and warnings. If USD composition is new to you, the [first OpenUSD stage for artists](https://digitalgarden.si/blog/openusd-pipeline-beginners/) explains references, payloads and variants with downloadable files.

## Follow-up delivery checks

Three follow-up tests cover different parts of the pipeline. [MuTracer with a Houdini plant](https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/) checks colors, alpha, instances, denoising downloads and fixed-sample convergence. [GLB cache and first-frame measurements](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/) publish twenty controlled loading rows. [Houdini and Geometry Nodes plant export](https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/) compares editable native files, triangle correspondence and retained wind attributes, including two rejected export cases.

## Gaussian splats

Splats are not part of the production garden. To see [where splats fit next to mesh plants](https://digitalgarden.si/blog/gaussian-splats-threejs/), read the native Three.js test with an owned synthetic fixture, opaque mesh references and the SPZ v4 correction.

[Choosing a splat library for Three.js](https://digitalgarden.si/blog/spark-vs-gaussiansplats3d-threejs/) then loads that same fixture through three integrations and records which renderer and encoding combinations actually drew.

For [a generated million-point garden in Three.js](https://digitalgarden.si/blog/viggle-jst2-spz-threejs-garden/), a free PINOC / JST-2 export in SPZ v4 is placed beside an opaque Houdini plant. It loaded in native Three.js on WebGPU and its WebGL2 fallback, and in Spark 2.4.0 with the classic renderer; Spark with WebGPURenderer failed to initialize. That input file and those package versions differ from the synthetic fixture above.

## Coding agents for rendering code

[Handing a shader task to a coding agent](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/) gives two coding tools the same Three.js plant wind and color-space task, keeps their outputs unchanged, and checks them independently on WebGPU and WebGL2. For further agent experiments, see the [machine-skill pump simulations with Codex](https://digitalgarden.si/blog/codex-machine-skill-threejs/) and the [community tests of Gemini 4 Argon](https://digitalgarden.si/blog/gemini-4-argon-threejs/).

## How to use these notes

The [slop-newyork and garden profile](https://digitalgarden.si/blog/houdini-vs-code-procedural-threejs-slop-newyork/) compares full download budgets and three 1080p observations of two different WebGPU scenes. The [SimReady USD-to-GLB test](https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/) follows two NVIDIA assets through Blender, Houdini, usd2gltf and direct USD loading, including the failed paths and material-preservation limits.

The [img2interior room test with Codex](https://digitalgarden.si/blog/img2interior-codex-threejs-garden/) follows a single synthetic reference through Blender authoring, PBR/runtime delivery, a static portable GLB and actual garden import. It records Windows fixes and the difference between a 2.67 MB model and its 95.10 MB required runtime payload.

The [Spirula Studio to Spark sample test](https://digitalgarden.si/blog/spirula-studio-garden-splat-threejs/) adds a measured training-to-browser handoff: the official Mip-NeRF 360 Garden sample, supplied camera poses, Vulkan training, PLY export and SPZ conversion. It documents 100,000 splats and a mixed WebGL render while keeping phone capture, fresh SfM and mobile performance outside the tested scope.

Each note links its downloadable fixtures and machine-readable results where they exist. Rerun the experiment on your own hardware before relying on a number, and treat a screenshot as evidence of one renderer path, not of every device. Then [walk through the garden](https://digitalgarden.si/play/) to see the production scene these tests support.
