# Blender 5.3 Gaussian Splats: Import, Crop and Export Tested

> Blender 5.3 imported four splat formats and preserved a crop, but native exports lost data. A 12,000-splat desktop test checks the Three.js return path.

- Author: Alicia Lu
- Published: 2026-11-12
- Canonical HTML: https://digitalgarden.si/blog/blender-5-3-gaussian-splats-threejs/

**Blender 5.3.0 imported the same 12,000-splat plant through PLY, SPZ, glTF and USD. A Geometry Nodes crop preserved its surviving attributes. None of the native exports tested here returned a complete web-readable splat.** File creation, point preservation and splat preservation were three different results.

Codex ran this desktop study under Alicia's direction on 12 November 2026. Alicia is a 3D technical artist (TA) and independent developer. The fixture is the owned procedural plant from our [Houdini 22 GSplats study](https://digitalgarden.si/blog/houdini-22-gaussian-splats-threejs/), trained previously from 24 Karma views for 1,200 steps. It is a synthetic plant fit, not a photographic scan, a newly trained Blender model or the live Digital Garden scene.

![The owned 12,000-splat plant rendered in Blender 5.3 Cycles before cropping, with an orange flower and green leaves](https://digitalgarden.si/marketing/research/blender53-20261112/source-cycles.png)

## What Blender officially promises

The [5.3 rendering notes](https://developer.blender.org/docs/release_notes/5.3/rendering/) explicitly say: “There is currently no export support.” They describe native PLY/SPZ import, a Gaussian-splat point-cloud type, and limitations around color and applying transforms. The [pipeline and I/O notes](https://developer.blender.org/docs/release_notes/5.3/pipeline_io/) separately document glTF/USD splat import and generic point-cloud improvements. Generic point-cloud export is not a promise to preserve a Gaussian-splat field.

The [Blender developer article](https://code.blender.org/2026/10/splatting-the-gaussians-into-blender/) explains the different rendering approaches and rudimentary relighting. We checked these claims against the installed release, rather than treating release footage or a successful export notification as a round trip.

## Machine, source and measurement boundary

| Item | Recorded setup |
|---|---|
| Blender | 5.3.0 Release, build `02f0899d78e3`, built 10 November 2026 |
| Workstation | Windows 11, Ryzen 9 9950X, RTX 5070 Ti, driver 621.14 |
| GPU memory | Adapter reports 16,303 MiB total; one pre-test snapshot used 2,607 MiB; no per-import allocation or VRAM peak measurement |
| Native images | 800 × 600, fixed orthographic camera; source/crop share a camera; transform pair uses a separate shared camera |
| Native color setup | Process-scoped ACES 2.0 OCIO config, sRGB display, Un-tone-mapped view, no look, exposure 0, gamma 1 |
| Browser lab | Desktop Chrome 157, headed, 800 × 600 canvas, DPR 1; Three.js 0.186.1; Spark 2.3.1 |
| Source properties | 12,000 splats; degree-three SH storage, with degree-one training active in the historical fit |

The original PLY is **3,121,605 bytes**, its lossy SPZ v3 derivative **235,748 bytes**, and the native USD **3,121,407 bytes**. Their hashes accompany the evidence. A local glTF repack uses the same PLY records, linear scale, normalized opacity, xyzw quaternions and the [KHR_gaussian_splatting](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_gaussian_splatting) attribute schema, including `kernel: ellipse` and `colorSpace: srgb_rec709_display`. That repack tests import; it is not a Blender export.

The historical Karma color pipeline and Blender's documented sRGB assumption differ. The glTF color-space field is an explicit structural-test declaration; no original Karma-to-sRGB conversion was validated. These pictures establish visible output and structural behavior, not colorimetric equivalence across native and browser renderers.

## Four imports, one source

All four paths created `POINTCLOUD` objects with type `GAUSSIAN_SPLAT`, 12,000 points, `scale`, quaternion `rotation`, four-component `radiance:base`, and 15 `radiance:sh_N` vectors.

| Input | Single operator duration | Orientation used for this source | Other observation |
|---|---:|---|---|
| Standard 3DGS PLY | 2.79 ms | Imported identity; rotate object X +90° for Blender Z-up inspection | Additional source normals/albedo scalar attributes retained |
| SPZ v3 | 4.13 ms | Importer already changed this file's coordinates to Z-up | Anti-aliasing support warning in the native log |
| USD particle field | 33.44 ms | Importer supplied X +90° object rotation | Native splat type detected |
| glTF repack | 43.55 ms | Importer converted glTF Y-up data | Native splat type detected |

These are one operator call each in already running background processes. They exclude process startup, interface painting and viewport readiness. They are diagnostics, not a format speed ranking. SPZ versions other than this v3 derivative were not tested. An axis correction for this writer is not a universal SPZ rule.

## Cropping works; inspect evaluated geometry

On a duplicate of the PLY object, a Geometry Nodes group connects **Position → Separate XYZ → Compare (X > 0) → Delete Geometry**, on the point domain. It retains X ≤ 0 in source coordinates. A `source_id` integer attribute lets the test match each survivor back to its exact input record.

| Crop check | Result |
|---|---:|
| Base datablock points | 12,000 |
| Evaluated modifier output | 5,906 |
| Surviving source IDs match the intended selection | Yes |
| Maximum absolute difference, surviving position/scale/quaternion/base radiance | 0 |
| Maximum absolute difference, all 15 stored SH vectors | 0 |
| Fresh-process `.blend` reopen | Base 12,000 / evaluated 5,906 |

Reading only `object.data` would incorrectly report that the modifier had done nothing. The tested code obtains `object.evaluated_get(depsgraph).data` before checking counts and attributes. Edit Mode deletion and more complex cleanup were not measured.

![The same fixed Blender camera after the X-positive crop, retaining 5,906 splats and removing much of the right side](https://digitalgarden.si/marketing/research/blender53-20261112/crop-cycles.png)

## Applying transforms changed the appearance

A separate duplicate had X +90° and Z +90° object rotations plus uniform scale 2. Before and after applying rotation and scale, the same camera viewed the same intended world-space arrangement.

World-space point centers agreed within **5.95 × 10⁻⁸** after application. However, the `scale` and quaternion `rotation` attribute arrays did not change at all. The first three SH vectors contained nonzero values before the operation and also remained unchanged. Point centers moved into the datablock while the stored splat shape and directional radiance did not follow the object transform.

![The transformed plant before Apply Transform, with continuous green leaves and an orange flower](https://digitalgarden.si/marketing/research/blender53-20261112/transform-before.png)

![The same transformed plant after applying rotation and scale, showing a visibly fragmented flower and leaves](https://digitalgarden.si/marketing/research/blender53-20261112/transform-after.png)

The paired renders visibly fragment after application. This reproduces the documented transform limitation for this fixture. Keep object transforms unapplied in this workflow; a general transformed-splat exporter would need to handle covariance and SH rotation correctly. Our reference crop writer does neither because it deliberately preserves the original coordinate system.

## Rendering and a bounded relighting probe

| Image operation | Settings | One wall-clock duration |
|---|---|---:|
| Original, Workbench | Flat lighting | 0.83 s |
| Original, EEVEE | 128 samples | 3.42 s |
| Original, Cycles | OptiX GPU, 64 samples, denoising off | 0.87 s |
| Cropped, Cycles | Same 64 samples and source camera | 0.29 s |
| Diffuse probe, EEVEE | Gray Diffuse BSDF plus point light, 128 samples | 0.47 s |
| Diffuse probe, Cycles | Same material/light, 64 samples | 0.46 s |

These sequential single renders include different startup/cache costs and different rendering work. They do not establish an engine ranking or viewport FPS. The diffuse probe was a deliberately gray material replacement. It produced a lit grayscale plant, rather than recovering the learned plant's albedo. A retained copy and both probe images are in the package; baked illumination removal was not tested.

## Export matrix: success messages are insufficient

Only the cropped object was selected. Generic attribute export was enabled where available. Every written artifact was independently reopened in a fresh empty Blender scene.

| Native export | Bytes written | Reopen result | Complete splat preserved? |
|---|---:|---|---|
| PLY, evaluated modifier enabled | 156 | Header says `element vertex 0`; native reimport fails | No |
| GLB, default `export_apply=false` | 144,664 | 12,000 ordinary mesh vertices; crop and splat attributes absent | No |
| GLB, `export_apply=true` | No output | Raises `Object does not have geometry data` | No |
| GLB, modifier baked first, `export_apply=false` | 71,540 | 5,906 ordinary mesh vertices; splat attributes absent | No |
| USD | 2,666,502 | 5,906-point `POINTS` cloud; scale, rotation and SH retained; base radiance missing | No |
| Alembic | 736,250 | 5,906-point `POINTS` cloud; scale and renamed SH retained; rotation and base radiance missing | No |
| SPZ | Not available | No native SPZ export operator found | Not tested through a native exporter |

The USD and Alembic logs explicitly warn about unsupported `radiance:base`. Their ordinary point-cloud type and missing data prevent calling this a splat round trip. Simply changing the point-cloud type back cannot recover a missing base radiance/opacity vector. The GLB default also illustrates why counting bytes or relying on a saved filename can conceal a modifier-evaluation problem.

## Actual browser return path

The local lab ran eight functional browser starts, one per backend/file pair. Native Three.js selected WebGPU; Spark used `WebGLRenderer`. Each successful case settled for 45 frames before capture. There is no FPS benchmark here. Three's native SPZ path needed an X = π object rotation for this particular file; Spark used its own decoded convention.

| File | Native Three.js GaussianSplat | Spark 2.3.1 |
|---|---|---|
| Original PLY | Visible, 12,000 splats | Visible, 12,000 splats |
| Original SPZ v3 | Visible, 12,000 splats | Visible, 12,000 splats |
| Reference source-record crop PLY | Visible, 5,906 splats | Visible, 5,906 splats |
| Blender's native crop PLY | Rejected: required splat properties missing | Rejected: `scale_0` missing |

The **reference crop** is a separate diagnostic writer: it copies the original PLY records selected by the same source-X rule. The count and IDs match the evaluated Blender crop. It is **not** a native Blender export, a general exporter for edited attributes, or evidence that transformed SH were handled correctly. It proves the viewer accepts a complete cropped splat when the original schema and records survive.

![The original SPZ plant visibly rendered by Spark 2.3.1 in the local desktop browser](https://digitalgarden.si/marketing/research/blender53-20261112/web-spark-plant-spz.png)

![The original PLY in native Three.js WebGPU, showing a visibly more sparse rendering in this setup](https://digitalgarden.si/marketing/research/blender53-20261112/web-native-plant-ply.png)

The native Three.js image is visibly sparser than the Spark image. We retain that difference rather than claiming matched image quality. These two functional captures do not explain every renderer difference; our [renderer comparison](https://digitalgarden.si/blog/threejs-gaussian-splat-renderer-comparison/) treats renderer behavior separately. glTF, USD and Alembic were not forced through PLY/SPZ-only loader entries. The exported GLBs have no Gaussian-splat extension, and these native/Spark lab paths supply no equivalent USD/Alembic splat import.

## Practical workflow and reproduction

Use native Blender import to inspect, combine and crop a splat, then preserve the `.blend` as the editable result. Before selecting a delivery tool, ask whether it preserves the **evaluated count, radiance/opacity, scale, rotation, SH convention and coordinate system**. Reopen the artifact and test it in the intended viewer. A third-party exporter is a separate pipeline with its own version, license and limitations; no PlayCanvas account upload was performed here.

For a working web deliverable today, keep the source standard PLY/SPZ or use a separately verified export/conversion path such as our bounded [Houdini attribute round trip](https://digitalgarden.si/blog/houdini-22-gaussian-splats-threejs/). Compression and viewer choice remain separate decisions; see the [PLY, SPZ and SOG study](https://digitalgarden.si/blog/spz-vs-sog-vs-ply-gaussian-splat-formats/).

The [machine-readable summary](https://digitalgarden.si/marketing/research/blender53-20261112/analysis.json) and [reproduction package](https://digitalgarden.si/marketing/research/blender53-20261112/blender53-splat-reproduction.zip) include owned inputs, the editable compressed `.blend`, scripts, raw attribute arrays, native artifacts, screenshots and browser receipts. Source code is MIT; owned fixture data, text and captures are CC BY 4.0, Alicia / digitalgarden.si. Blender and browser/dependency binaries, user settings, account data and full copied vendor pages are excluded. Dependencies retain their upstream licenses.

The first browser harness incorrectly attempted a WebGL `finish()` call on the native WebGPU canvas context. An initial glTF repack also copied nonunit source quaternions; that invalid diagnostic input was excluded, then normalized and reimported for the reported glTF result. The maximum float32 unit-norm error was 1.79 × 10⁻⁷. Pilot receipts remain separate from the corrected results. Public HTTPS hosting, iPhone Safari, large photographic scans, native SPZ v4, generic edited-attribute export and viewport FPS remain untested. Codex assisted execution, numerical checks and writing.

## Questions

### Does Blender 5.3 support Gaussian splatting without an add-on?

Yes. This Blender 5.3.0 desktop test imported the same owned 12,000-splat fixture through PLY, SPZ v3, glTF and USD as GAUSSIAN\_SPLAT point clouds. It did not train a new splat from photos.

### Can I crop Gaussian splats in Blender 5.3?

Yes in this fixture. A Geometry Nodes Delete Geometry selection removed positive-X points, leaving 5,906 evaluated splats. Surviving position, scale, rotation, base radiance and all 15 stored SH vectors matched their source records exactly.

### Can Blender 5.3 export a standard Gaussian-splat PLY?

The official release notes say there is currently no export support. Our native PLY exporter returned FINISHED but wrote a 156-byte file with zero vertices. This is one tested build and fixture, not a claim about every add-on.

### Does a successful GLB export preserve Gaussian splats?

No in this test. The default export retained 12,000 ordinary vertices without the crop or splat attributes. Baking the crop first retained 5,906 ordinary vertices, still without splat attributes. The export\_apply option raised a geometry error.

### Can Blender relight a splat like a scanned material?

Only rudimentary relighting is documented. A gray Diffuse BSDF and point light produced a lit grayscale result here. That did not recover albedo or remove baked illumination from the learned radiance.
