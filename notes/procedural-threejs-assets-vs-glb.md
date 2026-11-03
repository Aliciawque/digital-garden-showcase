# Procedural Three.js Assets vs GLB: 148 Local Navigations, Measured

> Three Polyfork assets, four delivery paths and 148 local browser navigations: payload, first GPU completion, instancing, factory calls and rebuild costs.

- Author: Alicia Lu
- Published: 2026-11-03
- Canonical HTML: https://digitalgarden.si/blog/procedural-threejs-assets-vs-glb/

**Short answer:** the module had the smallest gzip asset payload for the tree and car; Draco had the smallest lamp file. Payload size alone did not predict the first completed frame. For the Toromiro tree, the cold-navigation median was 376.5 ms for the module and 321.6 ms for meshopt. Their asset-ready phases were 62.3 and 20.3 ms. After construction, both tree paths used five instanced parts and six renderer calls, including the output pass.

These are independent, local measurements of three [Polyfork](https://polyfork.dev/) low-poly assets. They compare code-built geometry with the official GLB and locally compressed derivatives. The production Digital Garden was not migrated. The [Three.js garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/) places this delivery test alongside the renderer, plant-authoring and splat studies.

![The measured Toromiro tree rendered from its ES module in the independent Three.js WebGPU harness](https://digitalgarden.si/marketing/research/procedural-20261103/tree.png)

## The setup and what “first frame” means

The machine used an RTX 5070 Ti with 16,303 MiB reported by NVIDIA and driver 617.42. The browser was HeadlessChrome/156.0.0.0, headless, at 1280 × 720 and device-pixel ratio 1. WebGPU reported NVIDIA Blackwell and a non-fallback adapter. Forced WebGL2 reported the NVIDIA card through ANGLE/D3D11. Three.js was pinned to 0.186.1.

| Official asset ID | Triangles measured | Mesh parts | Distinct materials | Image textures |
| --- | --- | --- | --- | --- |
| toromiro-tree-a50794 | 906 | 5 | 1 | 0 |
| street-lamp-b4fa26 | 422 | 1 | 1 | 0 |
| convertible-60s-b76f89 | 4,880 | 5 | 1 | 0 |

The current module and raw GLB had matching transformed vertex positions in a bidirectional nearest-vertex comparison. All nine GLBs passed the glTF validator with zero errors. Draco and meshopt introduced nonzero position quantization differences; the largest measured nearest-vertex distance across the three models and both codecs was below 0.00030 source units. This does not establish identical topology, normals, colors or lossless images.

There were 60 fresh browser profiles for the startup study: three assets × four paths × five repetitions. Each profile ran a cold navigation and a paired reload, producing 120 navigations. “Cold” means a new profile's HTTP cache, not a cleared OS disk cache or GPU driver cache. The reload retained both browser-process and HTTP-cache warming, so their effects cannot be separated here. Treatment order rotated between repetitions.

The local server served gzip level 9 with cacheable assets. It imposed no network throttle. I timed navigation through the first render and waited for GPU work completion. That is a submitted-work boundary, not proof of when a physical display presented a frame. The asset-ready phase includes module import/factory work, or loader/decoder setup, fetch and parse. The ES module's import phase includes fetching and evaluating its dependencies; it is not a pure JavaScript-parse timer.

For a separate authored-plant cache experiment, see [GLB loading, cache and the first GPU frame](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/). The broader [Houdini assets versus runtime geometry comparison](https://digitalgarden.si/blog/houdini-vs-code-procedural-threejs-slop-newyork/) compares two complete scenes; this smaller matched-asset study answers a different question.

## Startup: small files still have construction work

All values below are milliseconds, median of five paired trials per asset and path. The full individual rows and ranges are in the data package.

| Asset | Path | Cold nav → GPU complete | Reload → GPU complete | Cold asset-ready phase | Reload asset-ready phase |
| --- | --- | --- | --- | --- | --- |
| Toromiro tree | ES module | 376.5 | 181.0 | 62.3 | 46.8 |
| Toromiro tree | Raw GLB | 333.4 | 135.5 | 17.1 | 2.8 |
| Toromiro tree | Draco GLB | 350.7 | 149.5 | 37.6 | 15.0 |
| Toromiro tree | Meshopt GLB | 321.6 | 138.0 | 20.3 | 4.0 |
| Street lamp | ES module | 314.0 | 139.2 | 11.1 | 3.6 |
| Street lamp | Raw GLB | 325.2 | 134.4 | 15.4 | 2.7 |
| Street lamp | Draco GLB | 343.4 | 147.9 | 35.5 | 14.5 |
| Street lamp | Meshopt GLB | 325.8 | 134.6 | 18.7 | 3.5 |
| Convertible | ES module | 325.5 | 138.6 | 21.5 | 9.2 |
| Convertible | Raw GLB | 341.8 | 137.0 | 22.9 | 3.3 |
| Convertible | Draco GLB | 359.7 | 149.6 | 39.0 | 16.6 |
| Convertible | Meshopt GLB | 328.4 | 135.3 | 21.3 | 4.1 |

![Cold and reload navigation-to-GPU-completion medians and full observed ranges for three assets and four delivery paths](https://digitalgarden.si/marketing/research/procedural-20261103/startup.png)

The tree is the clearest caution against equating bytes with startup speed. Its procedural factory does real CPU work. A ready mesh file can avoid that construction phase even when its payload is larger. The lamp and car have different costs; five trials on this computer do not establish a universal crossover point or statistically significant ranking.

Shader compilation, renderer initialization and browser startup remain inside the navigation metric. The stage columns help explain the path without pretending that subtracting medians produces a complete additive timing breakdown. This study did not time a remote Polyfork CDN navigation, slow mobile networks or texture-heavy scenes. The [Draco, meshopt and KTX2 study](https://digitalgarden.si/blog/draco-vs-meshopt-ktx2-benchmark/) covers separate geometry and texture fixtures.

## Bytes: asset payload and loading code are different budgets

These are exact gzip level 9 file sizes in bytes. The official module and raw GLB were fetched separately; glTF Transform 4.5.1 generated the compressed GLBs with the pinned recipe in the package.

| Asset | ES module | Raw GLB | Draco GLB | Meshopt GLB |
| --- | --- | --- | --- | --- |
| Toromiro tree | 10,663 | 41,883 | 28,647 | 24,051 |
| Street lamp | 5,439 | 8,842 | 2,686 | 5,664 |
| Convertible | 9,828 | 51,130 | 14,542 | 31,417 |

“Module smaller than raw GLB” held for these three assets. The lamp’s Draco file was smaller than its module. The car's actual triangle count also differs from the older example on [Polyfork's performance page](https://polyfork.dev/performance). Its published timings describe the vendor's earlier models, versions and machine; none were copied into my result tables.

A second measurement used Vite 8.3.4 to minify twelve explicit model-loading entries. It recursively counted every static dependency chunk, added the GLB where applicable, and included the Draco wrapper plus WASM binary for Draco. Each entry was counted as a standalone load; an application's renderer and other code were excluded.

| Asset | Module entry, gzip KiB | Raw entry + asset | Draco entry + files | Meshopt entry + asset |
| --- | --- | --- | --- | --- |
| Toromiro tree | 67.4 | 114.1 | 177.4 | 103.8 |
| Street lamp | 63.6 | 81.8 | 152.1 | 85.8 |
| Convertible | 66.6 | 123.1 | 163.6 | 111.0 |

![Asset-only gzip bytes compared with standalone loading-entry, dependency and decoder totals](https://digitalgarden.si/marketing/research/procedural-20261103/payload.png)

This bundle experiment retained the chosen factory and loading function. It is a specific bundling result, not a guarantee that arbitrary procedural source will tree-shake well. Nor are those standalone totals the incremental cost inside an application that already loaded Three.js or a decoder. The decoder and dependency file list is included separately so those budgets can be recomputed.

The official CDN responses were public HTTP 200 with wildcard CORS and a one-day browser cache lifetime, plus longer shared-cache directives. The URLs did not contain immutable version identifiers. The package therefore records SHA-256 values and stops if a later official download differs.

## Instancing: keep every part and its transform

A one-material model does not necessarily contain one mesh. Taking only `meshes[0]` would have discarded four parts of this tree. I constructed one InstancedMesh per static mesh, retained each part's world transform, and used identical deterministic placement for the module and meshopt paths.

This abbreviated pattern preserves the hierarchy transforms for the tested static assets:

```js
source.updateMatrixWorld(true);
const count = 1000;
const side = Math.ceil(Math.sqrt(count));
source.traverse(part => {
  if (!part.isMesh) return;
  if (part.isSkinnedMesh || Array.isArray(part.material)) {
    throw new Error('This example expects a static, single-material part');
  }
  const batch = new THREE.InstancedMesh(
    part.geometry, part.material, count
  );
  const placement = new THREE.Matrix4();
  for (let i = 0; i < count; i++) {
    placement.makeTranslation(
      (i % side) * 4, 0, Math.floor(i / side) * 4
    );
    placement.multiply(part.matrixWorld);
    batch.setMatrixAt(i, placement);
  }
  batch.instanceMatrix.needsUpdate = true;
  scene.add(batch);
});
```

The full harness also normalizes model height and frames the whole grid. These are three repeats per condition, each with 600 measured RAF intervals after 30 warm-up frames. The frame columns are medians of trial means or trial p95 values; CPU submission is the median of the combined per-render timings.

| Backend | Trees | Path | Renderer calls | Mean RAF ms | p95 RAF ms | CPU submit ms |
| --- | --- | --- | --- | --- | --- | --- |
| webgl2 | 1,000 | Meshopt GLB | 6 | 16.7 | 16.8 | 0.2 |
| webgl2 | 10,000 | Meshopt GLB | 6 | 16.7 | 16.8 | 0.2 |
| webgpu | 1,000 | Meshopt GLB | 6 | 16.7 | 16.8 | 0.2 |
| webgpu | 10,000 | Meshopt GLB | 6 | 16.7 | 16.8 | 0.2 |
| webgl2 | 1,000 | ES module | 6 | 16.7 | 16.8 | 0.2 |
| webgl2 | 10,000 | ES module | 6 | 16.7 | 16.8 | 0.2 |
| webgpu | 1,000 | ES module | 6 | 16.7 | 16.8 | 0.2 |
| webgpu | 10,000 | ES module | 6 | 16.7 | 16.8 | 0.2 |

![Measured RAF intervals and CPU submission for 1,000 and 10,000 trees using both delivery paths and backends](https://digitalgarden.si/marketing/research/procedural-20261103/instances.png)

Five part submissions plus the renderer's output pass explain the six calls. All CPU submission medians rounded to 0.2 ms; sub-0.1-ms differences are unresolved by these samples. Both formats retained the same submission count at both densities. Near-16.7 ms RAF scheduling in headless Chrome is a ceiling on what this counter can distinguish; it does not show spare GPU headroom, equal shader cost or a win on a phone. I did not test BatchedMesh, dynamic LOD, skeletal animation or a production garden here. The separate [vegetation LOD study](https://digitalgarden.si/blog/threejs-webgpu-vegetation-lod/) retains its own fixtures and results.

## Factory copies and parameter changes

For the car I timed twenty additional factory calls, then twenty deep clones of the already-built object, five CPU-only repetitions per backend. The clones shared geometry. These temporary groups were not rendered, so this test measures construction and object copying, not GPU upload time or VRAM multiplication.

| Harness backend | 20 factories, median ms | 20 clones, median ms | Distinct factory geometries | Distinct clone geometries |
| --- | --- | --- | --- | --- |
| webgl2 | 54.9 | 0.6 | 40 | 2 |
| webgpu | 61.4 | 0.5 | 40 | 2 |

The first factory that produced the clone source was outside the clone timer. These values therefore answer “how much do twenty additional copies cost after the first model exists?” They are not the vendor's older twenty-copy benchmark or a universal speed ratio.

For the tree I made fifty sequential height changes per backend, reusing the original materials and disposing replaced geometry. These are controlled programmatic steps, not measured human slider gestures.

| Backend | Factory median ms/change | Maximum ms/change | Renderer geometry count after each replacement |
| --- | --- | --- | --- |
| webgl2 | 36.2 | 46.7 | 6–6 |
| webgpu | 35.8 | 46.7 | 6–6 |

The stable geometry counter includes the renderer’s output geometry and describes this loop only. It is not a measurement of total GPU memory, JavaScript heap retention or arbitrary editor leak freedom. Material reuse matters because discarding the last material can also discard its compiled program. Parameter changes should be budgeted alongside the rest of a frame, rather than assumed to be free.

## License and a reproducible package

I read the full [official Polyfork license](https://polyfork.dev/licensing), revised October 2026, on 3 November. It permits personal/commercial project use, modification and research, and restricts redistribution of standalone original or modified asset files. Free official CDN linking is distinguished from hosting copies for other users.

Accordingly, the reproduction package contains my scripts, measurements, charts and source hashes. It excludes Polyfork GLBs, source modules and compressed derivatives. A helper fetches the six official source URLs individually and checks their hashes; compression runs locally after that. Public measurement JSON also omits the vertex-position arrays retained in the private geometry checks.

[Reproduction ZIP](https://digitalgarden.si/marketing/research/procedural-20261103/procedural-vs-glb-study.zip) · [Startup CSV](https://digitalgarden.si/marketing/research/procedural-20261103/startup.csv) · [Instancing CSV](https://digitalgarden.si/marketing/research/procedural-20261103/instances.csv) · [Full summary](https://digitalgarden.si/marketing/research/procedural-20261103/summary.json)

The formal dataset contains 148 navigations across 88 owned fresh browser profiles: 120 startup navigations, 24 instancing runs and four construction/rebuild checks. Pilot runs are separate. The startup and instancing windows contain 14,400 measured RAF intervals each; the four extra checks add 480.

My decision from this sample is to build once and reuse the resulting geometry, choosing the delivery format around construction cost, dependencies, required variants and the actual network. These measurements do not justify replacing the garden's authored Houdini plants.

## Short questions

**Can an ES module avoid GLTFLoader?** Yes. The tested factories returned THREE.Group objects and imported bare `three` plus a Three.js utility. My import map resolved them to the same pinned Three.js build.

**Are procedural models faster to render?** Delivery format alone did not change these tree submission counts. CPU construction, compression, vertex layout and the final scene still need separate measurements.

**Can this ZIP be used as a free Polyfork asset pack?** No. It provides a reproducible experiment and official download links; Polyfork's asset license continues to apply separately.

**AI assistance:** Codex implemented and ran this experiment, checked the results and prepared the draft and plots at Alicia's direction. Vendor claims remain attributed to their sources. The models are Polyfork assets; the test harness is an independent local fixture. Study code and authored charts are CC BY 4.0, while model imagery and dependencies retain their respective licenses.
