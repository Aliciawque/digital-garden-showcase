# OpenUSD in Three.js: A Houdini Solaris Test of USD Web Viewer

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A tested Houdini Solaris to Three.js workflow with usd-web-viewer 0.1.1: materials, instances, variants, units, warnings and measured local load times.

- Author: Alicia Lu
- Published: 2026-10-16
- Updated: 2026-10-23
- Canonical HTML: https://digitalgarden.si/blog/openusd-threejs-usd-web-viewer/

Can a Houdini Solaris stage reach a Three.js scene without a glTF conversion? **Yes, for the features in this small test.** On October 16, 2026, I exported a purpose-built Solaris stage and loaded its USDA, USDC and USDZ versions with Hugging Face's `@huggingface/usd-web-viewer` 0.1.1. Textured geometry, 100 point instances, two instanceable references and the selected look variant survived. A MaterialX-only material became grey, and a curve disappeared with a warning.

This browser test is one of the delivery notes in the [rendering guide overview](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

I make [digitalgarden.si](https://digitalgarden.si/), an [interactive 3D garden](https://digitalgarden.si/virtual-garden/) built with Three.js and Houdini-authored plants. This test ran in an independent local Three.js scene using the garden's Three.js version. It did not replace the live garden's GLB assets or exercise its WebGPU/TSL rendering pipeline. The results below describe this fixture and this machine.

![The actual browser test: textured panel, two spheres, selected blue cube and a grid of 100 pink instances](https://digitalgarden.si/marketing/research/openusd/solaris-browser.png)

The screenshot shows the local test harness, not the production garden. The grey sphere is an observed material fallback, not an unfinished screenshot.

## What I tested

| Component | Tested version or setting |
|---|---|
| Authoring | Houdini 22.0.429, headless hython; bundled OpenUSD 26.05 |
| Loader | @huggingface/usd-web-viewer 0.1.1, pinned from npm |
| Rendering | Three.js 0.186.1, WebGL2, antialiasing enabled |
| Browser and OS | Codex embedded Chromium 155, Windows 11 |
| GPU | NVIDIA GeForce RTX 5070 Ti, ANGLE / Direct3D 11 |
| Render size | 1265 × 446 pixels, renderer pixel ratio 1 |
| Texture mode | full, with a separate preview-mode check |
| Network | localhost, no throttling; five sequential measurements per format |

The package's published peer range is `>=0.162.0 <0.187.0` for Three.js, so the garden's 0.186.1 fits. Version 0.1.1 is MIT licensed. These are properties of the tested package, not a promise about a future release.[^1]

The fixture is deliberately simple: a UV-mapped quad with a checker texture and flat normal map; two implicit spheres; a colored cube; a cube with two look variants; a 10 × 10 grid of small cube instances; two instanceable references to an external layer; guide, proxy and render-purpose cubes; and a BasisCurves prim. I also authored a USD camera and light to check whether they appeared in the returned group.

## The Solaris export that actually ran

I used **Python Script LOP → Configure Layer → USD ROP**. The node types in this installation were `pythonscript`, `configurelayer` and `usd_rop`. The Python LOP authored the controlled fixture with OpenUSD APIs. Configure Layer explicitly set Z-up and 0.01 meters per unit. The USD ROP used `savestyle = flattenalllayers` and `defaultprim = World`.

This is a reproducible headless Solaris test, rather than a manual Material Library or Karma look-development session. The two materials were authored with UsdShade: one `UsdPreviewSurface` network and one MaterialX-only surface output. I did not test Karma image matching or a production MaterialX library.

There was a useful authoring catch. My first export retained Z-up but came out with `metersPerUnit = 1`. Adding the explicit Configure Layer node corrected the exported metadata to `0.01`. I reopened all three files through OpenUSD and checked the metadata, variant selection, 100 point placements and two native instances before measuring the browser. Merely setting a value in an intermediate LOP was not enough evidence.

The USD ROP wrote USDA and USDC files. I then packaged the USDC with OpenUSD's `UsdUtils.CreateNewUsdzPackage`, including its external reference and textures. The ROP's layer-flattening option retained the authored references and variant selection; this was not a fully flattened stage. SideFX documents USD ROP as the Solaris stage export node.[^2]

| File | Actual size | Contents needed beside it |
|---|---:|---|
| solaris-test.usda | 7,918 bytes | 183-byte reference layer and 385 bytes of PNG textures |
| solaris-test.usdc | 5,724 bytes | Same reference layer and PNG textures |
| solaris-test.usdz | 7,014 bytes | Self-contained package in this test |

You can [download the tested USDZ fixture](https://digitalgarden.si/marketing/research/openusd/solaris-test.usdz) and [inspect the measurements and object diagnostics](https://digitalgarden.si/marketing/research/openusd/results-2026-10-16.json). The fixture is test geometry made for this article; it contains no production garden plants.

## Load USD into an existing Three.js scene

The loader composes USD using Rust compiled to WebAssembly, runs loading in a worker, and returns a Three.js group. The package also offers a ready-made viewer, but I used `loadUsd` so the test scene owned its renderer, camera and lights.[^1]

```bash
npm install @huggingface/usd-web-viewer@0.1.1 three@0.186.1
```

For Vite development, exclude the package from dependency pre-bundling so its worker and WASM asset URLs remain usable.[^1]

```javascript
export default {
  optimizeDeps: { exclude: ['@huggingface/usd-web-viewer'] }
};
```

This is the load pattern used in the test, with cleanup included:

```javascript
import * as THREE from 'three';
import { loadUsd } from '@huggingface/usd-web-viewer';

const placement = new THREE.Group();
scene.add(placement); // scene, camera and lights already exist

const result = await loadUsd('/fixtures/solaris-test.usdz', {
  textures: 'full',
  maxTriangles: 100000,
  maxInstances: 1000
});

placement.add(result.root);
const textures = await result.complete;
console.log(result.info.warnings, result.info.materialKinds, textures);

// When this stage is no longer needed:
// scene.remove(placement);
// result.dispose();
```

The example adds explicit limits suitable for this small fixture; the recorded measurements used the loader's defaults. `complete` waits for textures after geometry becomes available. Keep a separate placement group: the returned root already carries the source-unit scale and up-axis rotation. In my test, its scale was `[0.01, 0.01, 0.01]` and its X rotation was `-π/2`. Overwriting those transforms would undo the conversion.

## What survived, and what did not

All three formats produced the same feature results. The returned diagnostics counted 108 mesh instances, eight geometries, nine materials and 3,182 triangles, with zero missing layers. The 100-copy grid was one actual `THREE.InstancedMesh`; the diagnostic mesh count should not be read as a draw-call count.

| Feature | Observed result |
|---|---|
| Mesh with UVs | The two-triangle checker panel rendered with its UVs |
| Implicit geometry | Both Sphere prims were tessellated and rendered |
| UsdPreviewSurface | Mapped to MeshStandardMaterial; checker texture visible |
| MaterialX-only material | Grey fallback; material-fallback warning |
| PNG textures | full loaded two textures with zero failures; preview loaded one and omitted normalMap |
| displayColor | Green cube and other authored colors appeared in material data |
| PointInstancer | 100 placements in one InstancedMesh |
| Instanceable references | Both referenced objects appeared; external layer resolved |
| Variant selection | The authored cool variant produced RGB [0.25, 0.4, 0.95] |
| Units and axis | Z-up centimeters became Y-up meters; fixture height measured 1 meter |
| Purpose | guide and proxy cubes absent; render cube present |
| Curves | BasisCurves absent, with a prim-unsupported warning |
| USD camera and light | Zero cameras and lights in the returned group; the harness supplied its own |

The flat normal map tests whether the normal-map path is loaded, not the quality of detailed normal shading. The reference test checks that both objects survive; it does not establish that every native-instance pattern is GPU-instanced. The variant test checks an authored selection at load time, not a live JavaScript variant switch. Those references and variants were already composed before export. The [OpenUSD beginner stage](https://digitalgarden.si/blog/openusd-pipeline-beginners/) is the smaller Solaris exercise for that composition: five files, a payload and a season variant, with no browser loader involved.

The loader reported:

```json
{
  "materialKinds": { "preview": 1, "displayColor": 7, "fallback": 1 },
  "upAxis": "Z",
  "metersPerUnit": 0.01
}
```

There were exactly two warning codes: `prim-unsupported` for `/World/UnsupportedCurve` and `material-fallback` for `/World/Looks/MaterialXOnly`. Their full diagnostic messages are preserved in the linked results JSON. They agree with the package's documented warning categories.[^3]

This is why I would check warnings before judging a USD import by its first frame. The stage loaded successfully, but it did not preserve every authored feature.

## Measured load times: small local fixture, not a mobile benchmark

After the first USDC load, I ran five sequential loads of each format. The table shows the median of each measurement. Fetches were local and caches were warm; these numbers are not internet delivery times, mobile results or a ranking of the formats on large scenes.

| Measurement | USDA | USDC | USDZ |
|---|---:|---:|---:|
| Loader totalMs | 11.1 ms | 12.1 ms | 10.0 ms |
| Fetch / parse / compose | 2.1 / 0.9 / 3.9 ms | 2.1 / 0.5 / 4.5 ms | 2.5 / 0.5 / 3.9 ms |
| Await loadUsd, from call | 21.3 ms | 21.0 ms | 20.4 ms |
| Await complete, from call | 23.9 ms | 36.6 ms | 21.5 ms |
| Reported WASM memory | 2,031,616 bytes | 1,966,080 bytes | 1,966,080 bytes |
| Triangles | 3,182 | 3,182 | 3,182 |

The first USDC load in the session reported 39.1 ms for `info.stats.totalMs`, 82.3 ms until `loadUsd` resolved and 87.2 ms until textures completed. I did not clear the browser's entire cache to establish a controlled cold-start experiment. The five raw samples per format are included in the results file.

`totalMs` is the loader's own statistic. My surrounding `performance.now()` measurements include more of the call lifecycle, so I report both. The fetch, parse and compose medians are independent medians and should not be added to reconstruct total time. Likewise, `wasmMemoryBytes` is not total browser or GPU memory.[^3]

For a real garden asset, the next useful test is much larger: plant texture fidelity, many repeated meshes, actual mobile devices and the existing WebGPU/TSL renderer. A seven-kilobyte USDZ cannot answer those questions. Two composed NVIDIA stages later used this same viewer. That [direct OpenUSD load of SimReady assets](https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/) also recorded where Blender, Houdini and usd2gltf stopped preserving materials.

## USD or glTF for the garden?

My practical result is that USD is now worth testing as a direct preview path from Solaris. The external reference, selected variant and instancing cases worked in this fixture. The material and geometry omissions also make the boundary clear.

The live [garden](https://digitalgarden.si/play/) continues to use its existing GLB pipeline. Khronos describes glTF as a format for efficient runtime 3D asset delivery; that remains a useful fit for a public browser experience.[^4] I did not run a matched Houdini-to-glTF benchmark, so this article makes no USD-versus-GLB speed or size claim. Exporting the already-loaded browser group to GLB would carry forward the import's losses and would not be a fair source-format comparison.

Three.js also provides its own `USDLoader` addon for USD, USDA, USDC and USDZ.[^5] I did not benchmark it here. Results for one loader should not be assigned to another. A later [Houdini USD and glTF test](https://digitalgarden.si/blog/houdini-usd-vs-gltf-threejs/) did use that loader on one authored plant pair, exported as GLB, USDC and USDZ. USDLoader kept a single plant and dropped the wind attributes, so the result is a feature failure rather than a speed ranking.

## FAQ

**Can Three.js load USD files from Houdini?**

In this test, yes: Solaris-generated USDA, USDC and a packaged USDZ loaded through usd-web-viewer 0.1.1. The supplied fixture and results let you check the same narrow case.

**Does a successful load mean MaterialX works?**

No. My MaterialX-only sphere rendered grey and produced a warning. The UsdPreviewSurface checker panel worked. Test the actual material network you intend to ship. Unreal accepted an embedded MaterialX surface in a separate [Houdini to Unreal test](https://digitalgarden.si/blog/houdini-unreal-usd/) after the shader input used the MaterialX name and the mtlx context was selected.

**Why do normals disappear in preview mode?**

The package's preview mode omits normal maps.[^3] My full-mode load attached a normalMap; the preview-mode load did not. Request full mode when that map matters.

**Does this show that USD will make the phone garden faster?**

No. These are desktop localhost measurements of a small diagnostic scene. No iPhone, Android, Safari or production-garden frame-rate claim follows from them.

**What would I test next?**

One real Houdini plant, its actual textures, a controlled glTF comparison and both garden rendering backends. The [Claude Code and Houdini workflow](https://digitalgarden.si/blog/claude-code-for-3d/) is how I keep these experiments repeatable: run the script, inspect the output, then decide what belongs in production.

[^1]: Hugging Face, [usd-web-viewer package and README](https://github.com/huggingface/huggingface.js/tree/main/packages/usd-web-viewer). This experiment installed and inspected the published npm 0.1.1 artifact; the repository main branch can change.
[^2]: SideFX, [USD ROP](https://www.sidefx.com/docs/houdini/nodes/lop/usd_rop.html). Node names and parameters in this article were also checked in the installed Houdini 22.0.429.
[^3]: Hugging Face, [load options, diagnostics and timing types](https://github.com/huggingface/huggingface.js/blob/main/packages/usd-web-viewer/src/types.ts). The installed 0.1.1 declaration files were checked alongside the measurements.
[^4]: Khronos Group, [glTF: Runtime 3D Asset Delivery](https://www.khronos.org/gltf/).
[^5]: Three.js, [USDLoader documentation](https://threejs.org/docs/pages/USDLoader.html).
