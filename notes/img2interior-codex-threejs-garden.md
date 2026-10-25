# I Tested img2interior with Codex: A Room and a 95 MB Runtime

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A Windows Codex test of img2interior: editable Blender output, an 8.80 MB portable GLB, a 95.10 MB runtime, real cloth checks and a garden import.

- Author: Alicia Lu
- Published: 2026-10-22
- Canonical HTML: https://digitalgarden.si/blog/img2interior-codex-threejs-garden/

**img2interior worked in this iterative Codex test, with local Windows fixes and a viewer lifecycle repair.** It produced an editable Blender scene, a runtime GLB with a Three.js viewer, and a separate portable GLB. The room is a simplified interpretation of one synthetic reference, with no measured reconstruction claim.

The largest practical finding was the payload. The runtime GLB was **2.67 MB**, but its required lightmaps and reflection probe brought the declared total to **95.10 MB**. The standalone portable file was **8.80 MB**. The full desktop workflow passed after review and repair; no physical phone was tested.

![Independent reopening and rendering of the final editable Blender room](https://digitalgarden.si/marketing/research/img2interior-20261022/native.png)

This is an actual render from a fresh process opening the final master. The geometric artwork, uniform stone, simplified upholstery and schematic trees show where the reconstruction stops.

[Inspect the static portable model](https://digitalgarden.si/research/img2interior/), [download the editable study](https://digitalgarden.si/marketing/research/img2interior-20261022/room-study.zip), or read the [machine-readable desktop results](https://digitalgarden.si/marketing/research/img2interior-20261022/results.json). The online preview is a separate static consumer; it does not reproduce the full PBR viewer's cloth simulation.

## One synthetic reference, a new scene builder

The user selected the project's public [synthetic living-room reference](https://img2threejs.io/living-room/assets/interior-modern-source.png): 2752×1536 pixels, with full-height glazing, an ivory sofa, a pale round table, a dark chair and a walnut cabinet. It was not a greenhouse photograph. The reference remains linked at its source and is not redistributed in the download.

img2interior is an agent workflow for authoring a Blender scene, rather than a pretrained image-to-mesh model. In the current Codex desktop conversation, I wrote a scene-specific Python builder using the package's reusable geometry helpers. I did not call its bundled Oak/Linen room recipe or run Claude Code.

The room has actual shell and furniture geometry, glazing, curtains, UV-mapped surfaces and authored lighting: **131 meshes, 46,544 triangles and 12 semantic components**. I estimated dimensions in metres, using a roughly 3.1-metre sofa as an anchor. Hidden construction and the camera remain authored assumptions. Native authoring uses Z-up; the glTF exports use Y-up.

This was not a fresh-session, unchanged-prompt benchmark. I preserved failed renders, changed the builder and checked the results. The [earlier Codex machine-skill test](https://digitalgarden.si/blog/codex-machine-skill-threejs/) follows the same distinction between using a skill and accepting its first output.

## Setup and the Windows interventions

The test used img2interior **0.3.1**, img2 CLI **0.4.0**, Python **3.11.14**, bpy **4.3.0** and NumPy **1.26.4**. The workstation ran Windows 11, a Ryzen 9 9950X and an RTX 5070 Ti. The observed browser was Chromium 155 through ANGLE D3D11. Codex ran in the current desktop conversation; the CLI version readback was 0.144.1.

I registered the package in an isolated harness home and host fixture. Doctor returned zero failures and zero warnings; sync returned zero drift. That verifies the isolated fixture. It does not establish that a newly restarted user Codex host discovered the installed skill.

Three Windows issues needed attention:

- An inherited OCIO setting pointed Blender at an incompatible studio configuration. Clearing OCIO only in the authoring child process fixed preflight while preserving the workstation setting.
- The exporter wrote a backslash in the source-image URL. The viewer rejected it. A local `Path.as_posix()` change fixed that field, with the original retained for rollback.
- Python 3.12 reported different path-stat and file-stat ctime values, triggering the file-change audit. Running the unchanged checks with Python 3.11 passed. The delivery contract also needed an equivalent relative root because its drive path was classified as a URL.

The downloadable study contains the builder, inputs, generated linen maps, packed master, portable model, applicable notices and the viewer lifecycle patch. Install the pinned official toolkit separately to rebuild. The archive excludes the reference image, environments, production garden plants and the large runtime sidecars.

## Render, reject, repair, then bake

The first native render took **5.44 seconds**. I rejected it: the camera was too close, and the table and chair were clipped. Two repair rounds adjusted the framing, hidden front-room extent and furniture proportions. I inspected source, lateral, reverse and textile views before final baking.

Five full/ambient bake groups took **193.17 seconds**, using a base resolution of 1024 and 2048 for textiles and outdoor geometry, 32 samples and OpenImageDenoise. A 1024×512 reflection probe took another **6.61 seconds**. These are measured stages, not a complete setup or generation stopwatch.

The UV/color/coordinate round trip reported zero error. A separate matched 256×192 browser calibration fixture had a mean absolute RGB difference of **0.003818** on a 0–1 scale against the native expected render. That checks lightmap transport, not resemblance to the source sofa.

## Three deliverables with different contracts

| Product | File bytes | What it preserves |
| --- | --- | --- |
| Packed Blender master | 1,037,640 | Editable geometry, cameras, lights and packed PBR source maps |
| Runtime GLB | 2,669,308 | Geometry and bindings for the full viewer, with required sidecars |
| Five EXRs plus reflection probe | 92,429,949 | Baked lighting and reflections required by that runtime |
| Runtime model plus those sidecars | 95,099,257 | Declared file total, before reference image and viewer code |
| Static portable GLB | 8,804,144 | Embedded baked appearance for independent glTF consumers |

These are decimal MB and file-byte totals, not a measured cold network transfer. The online static preview loads the unchanged portable GLB only after an explicit button press. It does not load the 95 MB runtime set.

Fresh Blender processes reopened the master and independently imported the portable model, then rendered each. Both final GLBs passed the Khronos validator with **zero errors and zero warnings**. The full viewer's actual downloads matched the final master and portable file hashes.

![Independent Blender import and render of the portable GLB, with softer static baked surface detail](https://digitalgarden.si/marketing/research/img2interior-20261022/portable.png)

The portable model embeds five static images and uses unlit materials for its baked appearance. Close surface detail is softer than the native/runtime PBR version. It does not carry the current browser fabric finish, live curtain state or dynamic relighting. The [GLB export notes](https://digitalgarden.si/blog/houdini-gltf-threejs-optimization/) cover a separate optimization workflow; I did not run that pass here or invent before-and-after numbers.

## Loading it in the actual Digital Garden renderer

I loaded the portable GLB through the existing Digital Garden Experience and real plant assets, using a local test entry, an inspection camera and derived placement. The garden used Three.js **0.186.1**; the original PBR room viewer used **0.180.0**. The room imported at scale 1, upright, without an extra rotation.

| Path | Observed conditions | Result |
| --- | --- | --- |
| Full WebGL2 room viewer | 1340×748 drawing buffer, wind enabled, preset views | 601 rendered frames in 10.0066 s, about 60.06 FPS |
| Garden WebGPU | 1265×540, DPR 1, low quality, spring/day, 162,420 plants | 600 intervals over 10 s, about 59.99 FPS |
| Garden WebGL2 fallback | Same scene and dimensions | About 59.99 FPS; median 16.7 ms, p95 16.8 ms |

Both garden paths hit the display's roughly 60 Hz ceiling. This does not show which backend is faster. The sampled garden view reported 175 draw calls and about 12.15 million rendered triangles, including rendering passes. For a controlled submission comparison, use the separate [plant instancing benchmark](https://digitalgarden.si/benchmarks/threejs-webgpu-vs-webgl-instancing/).

![Actual local garden integration: existing plants visibly intersect the new room](https://digitalgarden.si/marketing/research/img2interior-20261022/garden.png)

The import exposed a placement problem: existing plants intersected the room. I preserved the scatter. A production installation needs a reserved footprint. The live garden has not been changed to include this experimental room. Its portable baked light also cannot follow the garden's sun.

## Cloth, hems and the lifecycle bug

The desktop follow-up tested low, middle and high fabric points and the lower hem on both panels: eight bidirectional drag/release cases, plus two pinned upper-hem checks. The ray-hit data actually identified `part=hem`; cloth hits were not relabelled as hem results. The far lower hem was initially obscured by the sofa, so I navigated into a clear viewing position instead of weakening occlusion checks.

The [saved hem-grab capture](https://digitalgarden.si/marketing/research/img2interior-20261022/hem.png) shows the full cloth viewer during the test. The online portable preview has static curtains.

A separate vertical pull kept rail coverage at zero while moving the middle probe **76.37 mm**, a higher probe **5.09 mm**, and the pinned top **0 mm**. The maximum observed movable held-point error was **3.86 mm**, below the stated 5 mm test threshold. Camera transforms/projection and UV0/UV1 fingerprints stayed unchanged. Rendered world vertices matched the solver within Float32 precision.

Native pointer-capture loss, release outside the canvas, trusted browser blur, Source-mode changes, real offscreen pause, wind-off sleep, page exit and return all passed. Real navigation produced `pagehide` and hidden events and removed the old canvas. Isolated hidden-pause used a restored temporary getter/event because inactive tabs in the in-app browser stayed visible. Extra-touch/cancellation handlers used DOM PointerEvents. Neither simulation is physical-phone evidence.

The tests found a real lifecycle bug: **a late context-loss event on a detached old canvas could put its replacement viewer into an error state**. I removed the old keyboard/context listeners during disposal and guarded the context handler by session ownership. Dispatching the same old-canvas events after repair left the new viewer ready and its camera unchanged.

Actual WebGL loss, a blocked manifest request and a blocked atlas request each recovered through Retry load. The eight review stages and all three blocking gates passed after repair. The gates check ordering and bytes; the browser observations establish the scoped behavior.

## What I would use it for

The editable project is useful as a starting point for an approximate room, especially when I want a builder and Blender scene to continue editing. Review and repair remain part of the work. The geometric artwork, upholstery, wood, stone, outdoor trees and lighting are visibly simplified.

Before using this room as a production garden asset, I would improve the reconstruction, reserve its footprint, reduce the runtime payload and complete real-device checks. The 390×844 layout emulation had no horizontal overflow, but **no physical phone was connected**. There is also no complete setup timer or per-run token/cost baseline.

### Does img2interior work with Codex?

It did in this Windows test, using a new builder, render iterations, local compatibility changes and a viewer repair. This does not establish untouched first-pass reliability across other images or fresh host installations.

### Which GLB should I put in another Three.js scene?

The portable file is the self-contained static-appearance export. The runtime GLB requires its declared lightmaps, probe and viewer integration. The packed Blender master is the editing source.

### Is it open source?

The tested package is source-available under the [img2 Interior Free Use, Restricted Commercialization License 1.0](https://github.com/img2threejs/img2interior/blob/main/LICENSE), not an OSI open-source license. Read its output/runtime and third-party terms for your use. This test does not relicense the reference image or toolkit.

Related: [Three.js garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/), [Codex machine-skill test](https://digitalgarden.si/blog/codex-machine-skill-threejs/) and the [official img2interior project](https://github.com/img2threejs/img2interior).
