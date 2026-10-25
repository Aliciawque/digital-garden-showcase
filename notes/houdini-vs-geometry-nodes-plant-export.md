# Houdini vs Geometry Nodes: Exporting the Same Plant Data

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A controlled plant assembled in Houdini and Blender Geometry Nodes: editable files, 6076 matching triangles, wind attributes, color-loss fixes and GLB validation.

- Author: Alicia Lu
- Published: 2026-10-24
- Canonical HTML: https://digitalgarden.si/blog/houdini-vs-geometry-nodes-plant-export/

Houdini 22 and Blender 5.2 both exported the controlled plant with 6,076 triangles, three vertex-color groups and two wind attributes intact. Comparing every triangle in world space gave a maximum position difference of 0.000000082 metres. Getting there exposed two failures that a successful export button did not reveal: degenerate leaf tips and lost colors in my first Blender material setup.

This comparison starts from shared prototypes and placements. It tests editable assembly and deliverable data; it does not measure which application models a plant faster or offers the better general procedural system.

![The corrected Geometry Nodes plant export reopened in the Three.js browser reference](https://digitalgarden.si/marketing/research/followup-20261024/blender-reference.png)

Tested locally with Codex assistance on 23 October 2026. [Download both editable projects and reproduction scripts](https://digitalgarden.si/marketing/research/followup-20261024/followup-lab.zip), [triangle and attribute comparison](https://digitalgarden.si/marketing/research/followup-20261024/native-comparison.json), and the [glTF validation report](https://digitalgarden.si/marketing/research/followup-20261024/asset-validation.json).

## The shared recipe and the different native graphs

I generated a small, owned plant recipe: one stem, five leaves and forty bells, with fixed placements, phase seed 23 and a height of 0.62 metres. Both applications received those prototypes and transforms. This deliberate control removes modeling differences from the export comparison.

Blender used an editable 25-node Geometry Nodes graph. Prototype objects and placement points feed Instance on Points; rotation and scale fields set placement, Realize Instances prepares a single export mesh, and Store Named Attribute retains the wind fields. The accepted graph uses one shared vertex-color PBR material because the three color groups have the same material parameters.

Houdini used an editable Python SOP that embeds the same recipe, a Normal SOP and a glTF ROP. Height and phase seed are exposed as spare parameters. It is a scripted SOP assembly, not a claim of equivalent node-for-node modeling workflows.

Each native file was saved, reopened in a separate native background process and then evaluated and exported. I also unpacked the download into a second directory and reopened both packaged projects there: each still evaluated to 6,076 triangles, and Houdini's relative export path resolved inside the relocated folder. The active GUI projects were not used. Manual GUI editing on another workstation remains untested.

## What survived export

| Property | Blender Geometry Nodes | Houdini SOP |
| --- | --- | --- |
| Native runtime | 5.2.0 LTS | 22.0.429 |
| Native file reopened | Yes | Yes |
| Native points | 3,858 | 3,858 |
| GLB triangles | 6,076 | 6,076 |
| GLB primitives | 1 | 1 |
| Explicit GLB materials | 1 | 0; loader default used |
| Vertex-color groups | Three, preserved | Three, preserved |
| Custom attributes | `_STIFF`, `_PHASE` | `_STIFF`, `_PHASE` |
| glTF validator | 0 errors, 0 warnings | 0 errors, 0 warnings |
| GLB file bytes | 804,592 | 279,724 |

The files have different normal and vertex-splitting policies. Blender exported flat face normals with split vertices; Houdini's Normal SOP produced its own smoothing and packing. Houdini also omitted an explicit material while Blender wrote one. File size therefore does not isolate exporter efficiency or a software-wide advantage.

I matched triangle centroids one to one, then tested all six vertex permutations for each matched triangle. All 6,076 triangles passed a tolerance of 0.000001 metres. The largest position difference was 8.20 × 10^-8 metres. The maximum `_STIFF` error against height-derived weights was below 6.2 × 10^-8, and `_PHASE` error was below 7 × 10^-9.

These checks establish static data preservation. The two native files do not contain a demonstrated production wind rig, and carrying wind weights does not prove an animation system uses them correctly.

## Failure one: my degenerate leaf tips

The first shared leaf grid collapsed several tip vertices to the same position. It created twenty degenerate triangles; the first Houdini GLB then reported fifty zero-normal validation errors. That was a defect in my fixture generation, not a general Houdini failure.

I rejected triangles whose squared area was at most 10^-20, compacted unused vertices, and rebuilt both native projects. The accepted recipe has 6,076 triangles rather than the earlier 6,096. Both accepted exports then passed with zero errors and warnings. The rejected validation receipt and source correction are included separately in the package.

## Failure two: a valid file with white leaves

My first Blender setup used multiple materials that referenced the same color attribute. The evaluated Geometry Nodes mesh had the expected colors, but the exported GLB lost the green stem and foliage colors. It still passed format validation. I confirmed the failure in accessor data and in the browser image; retrying both named and material-based vertex-color export settings did not repair that case.

![The earlier rejected Blender export has white leaves despite passing format validation](https://digitalgarden.si/marketing/research/followup-20261024/blender-color-loss.png)

All three parts already shared the same PBR settings, so I changed this fixture to one vertex-color material. The accepted GLB retained all three colors, and its browser reopening showed green leaves again. I did not patch Blender's exporter or establish that every multi-material export loses color. A minimal upstream reproducer and independent confirmation would be needed for that broader conclusion.

## Inspect structure, then inspect appearance

The package contains `inspect-gltf.mjs`, a read-only preflight. It checks the JSON and glTF structure, reports material and image counts, lists custom attributes, and flags prototype-like names as hints. It does not delete objects or assume every repeated mesh reference is accidental duplication.

For this plant I also required triangle correspondence, attribute checks and actual browser images. The [SimReady conversion test](https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/) shows why source visibility and material networks need their own acceptance checks: a file can be valid while containing duplicate hidden prototypes or losing its intended appearance.

The practical choice here is about the editable source you need to maintain. Both controlled assemblies delivered the required static plant data after explicit corrections. The [MuTracer experiment](https://digitalgarden.si/blog/mutracer-houdini-plant-path-tracing/) rendered both accepted native exports, while the [loading experiment](https://digitalgarden.si/blog/threejs-glb-loading-cache-first-frame/) used a separate fixed 6,056-triangle asset to isolate compression and cache behavior. More pipeline tests are linked from the [garden rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).
