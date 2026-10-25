# Houdini to Unreal with USD: A Tested Stage, HDA and Material Pipeline

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A Houdini 22 to Unreal 5.8 test of USD Stage, MaterialX, 100 instances and HDA baking. Includes measured units, variant fixes and reproducible fixtures.

- Author: Alicia Lu
- Published: 2026-10-19
- Canonical HTML: https://digitalgarden.si/blog/houdini-unreal-usd/

**Use a USD Stage when you want to retain a composed scene; use Houdini Engine when the deliverable is a procedural tool that should cook and bake inside Unreal.** I tested both with independent fixtures: a small referenced flower and a 100-copy scatter. The units, materials and instance counts below are read from native Unreal components, not inferred from a successful file import.

The test used Houdini 22.0.429, its OpenUSD 26.5 build, Unreal Engine 5.8.3 and Unreal's OpenUSD 26.3 build on Windows. The installed Houdini Engine plugin identifies itself as `3.0 - H22.0.429` with version integer `22000429`. That local descriptor is the relevant compatibility evidence here; it differs from the plugin-version table in the pre-test draft.

The garden at [digitalgarden.si](https://digitalgarden.si/virtual-garden/) renders in Three.js. These Unreal experiments are separate test assets. They do not migrate the live garden or demonstrate the full Project Titan EVO tool collection.

## One fixture per failure

The [test archive](https://digitalgarden.si/marketing/research/briefs-20261019/houdini-unreal-usd.zip) includes the USD layers, a small packed-scatter HDA and source scripts. The USD stage contains:

- A one-meter cube for units and axis conversion.
- A Preview Surface material and an embedded MaterialX standard surface.
- A separate cube with an `unrealMaterial` attribute pointing to a native project material.
- A referenced schematic flower with bud and bloom geometry variants.
- A PointInstancer with 100 placements and one prototype.

Houdini opened this stage through a native Sublayer LOP. A separate SOP network packs a simple proxy and copies it onto a 10 × 10 grid. That network becomes the HDA used for the Engine path. The HDA proxy is intentionally simpler than the USD flower: this is a workflow and instance-count comparison, not a matched performance benchmark.

## What crosses each boundary

| Path | What remains editable | Observed output |
| --- | --- | --- |
| Open a USD Stage | USD composition, source prims and selected variants | Generated transient meshes, material instances and a 100-instance component |
| Native USD asset import | Converted project assets | Preview material instance and static meshes in the test Content folder |
| Interchange USD import | Converted project assets | Static meshes, Preview materials and native MaterialX materials |
| Houdini Engine HDA | Procedural network and exposed controls while cooking | 100 native instances; bake completed successfully |

Epic still labels the USD Stage feature Beta in the current 5.8 documentation.[^1] Keep that shipping caveat separate from the success of a small editor test. Opening a stage and importing assets are different operations, even when both start from the same file.

## Units: the reference cube measured 100 centimeters

The source explicitly declares Y-up and `metersPerUnit = 1`. In Unreal, the one-meter cube had half-extents `(50, 50, 50)` cm and its center was at Z = 50 cm. Its source X position of −1.5 m became −150 cm. The source vertical Y direction became Unreal's vertical Z direction.

That is a stronger acceptance check than reading the source metadata alone. Metadata describes how values should be interpreted; it does not repair geometry authored at the wrong scale. Put a real reference object in your fixture and measure the receiving component.

```python
component = usd_actor.get_generated_component("/World/MeterCube")
origin, half_extent, radius = unreal.SystemLibrary.get_component_bounds(component)
# Expected half_extent for a 1 m cube: 50 cm on each axis.
```

## Materials: render context and schema both matter

| Case | Native observation |
| --- | --- |
| Preview Surface, default context | `MI_Preview` assigned to the cube and flower |
| MaterialX-only material, default context | DisplayColor fallback material |
| Same embedded MaterialX, `mtlx` context | Generated MaterialX material instance assigned |
| `unrealMaterial` path | The expected `/Game/BlogTests/M_Garden` material assigned |
| Interchange, corrected MaterialX schema | Native MaterialX materials and static meshes created; final commandlet reported zero errors |

My first MaterialX definition used a Preview Surface input name: `roughness`. Interchange correctly rejected it because a MaterialX standard surface expects `specular_roughness`. After correcting that authored shader input, the same import finished with zero errors. This was a fixture bug. It is not evidence that Unreal cannot import MaterialX.

The context test is also narrower than a full material-match test. It checks native material creation and assignment for an embedded, constant-color standard surface. It does not establish Karma-to-Unreal pixel parity, external `.mtlx` reference behavior, every texture channel, displacement or a production shader library. Keep a Preview Surface fallback when that is the receiving application's more portable path.

For a native project override, the exact asset path must exist in the receiving project. The test created a green native material in its own Content folder before loading the USD cube. A string that happens to look like a material path is not sufficient evidence of a successful binding.

## Count the instance component, not the scene root

The PointInstancer's generated root was a plain SceneComponent. Its descendant `Prototype_0` was an **InstancedStaticMeshComponent with 100 instances**. Looking only at the root, or only at the actors returned by a commandlet's editor actor list, initially hid that result.

```python
root = usd_actor.get_generated_component("/World/Scatter")
for component in root.get_children_components(True):
    if hasattr(component, "get_instance_count"):
        print(component.get_class().get_name(), component.get_instance_count())
```

The installed version produced ISM, not HISM, for this fixture. Do not turn the documentation's list of supported representations into a promise about the specific class your scene will create. Count it after import and inspect the prototype's actual material assignment.

The HDA path also produced one InstancedStaticMeshComponent containing 100 packed copies. The native material override was authored before packing. This test does not establish arbitrary per-point color propagation across every packing/export combination; a color attribute and a material assignment are different contracts.

## A variant failure that was really a composition failure

The first bud selection did nothing. The source sphere already had a local radius opinion outside the growth variants. That stronger opinion defeated the variant-authored radii.

Clearing the local radius before authoring the variants fixed it. In Unreal, the generated flower's X/Y half-extent changed from 12 cm for bloom to 5 cm for bud. Reloading the source root layer restored the on-disk bloom selection and the 12 cm extent. An actor-level translation of 33 cm survived that reload.

This test authored the temporary variant opinion through the native OpenUSD Python API in the Unreal process. It did not prove that a particular USD Stage panel edit is saved into a persistent override layer. If the requirement is to keep an Unreal-authored variant selection across reloads, explicitly choose an edit target, save that layer and reopen it. The [beginner stage exercise](https://digitalgarden.si/blog/openusd-pipeline-beginners/) explains the opinion stack behind this failure.

## Cook and bake the HDA

The Engine plugin imported the test HDA, started a matching Houdini session, instantiated it and completed synchronous processing. The cooked actor contained 100 instances. `bake_all_outputs()` returned true and produced baked output actors.

SideFX's public API documents session management, asset wrappers, synchronous processing and baking.[^2] The installed API still supports `instantiate_asset`, but marks it deprecated in favor of `instantiate`. Record that distinction if you adapt this fixture; do not copy a tutorial's method names without checking your installed build.

The bake demonstrates that this small tool can produce native output. It is not a packaged-game test or evidence that a shipped game should cook HDAs at runtime. Reopen and inspect the baked assets without depending on a live Houdini session before accepting that delivery contract.

## A checklist that catches more than a successful import

1. Record the Houdini build, receiving UE patch, plugin descriptor and both USD versions.
2. Measure a known object in receiving units and confirm the vertical direction.
3. Test Preview Surface, MaterialX and native overrides separately.
4. Correct shader-schema errors before blaming a render context.
5. Inspect descendant instance components and assert the actual count.
6. Change a variant and measure a geometric consequence.
7. Reload the file and record which source and actor edits survive.
8. Import native assets separately from opening a live stage.
9. Cook and bake an HDA, then inspect the baked deliverable.

All substantive import, component and bake results were obtained in isolated test projects. The production demo project and the live garden's models were not changed. This is an editor pipeline test on one workstation, not a production certification. For the browser destination, the [OpenUSD in Three.js experiment](https://digitalgarden.si/blog/openusd-threejs-usd-web-viewer/) records a different set of material and primitive limitations.

## References

[^1]: Epic Games, [Universal Scene Description in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/universal-scene-description-in-unreal-engine), 5.8 documentation checked 19 October 2026.
[^2]: SideFX, [Houdini Engine for Unreal Public API](https://www.sidefx.com/docs/houdini/unreal/publicapi.html). Runtime signatures were additionally read from the installed plugin.
