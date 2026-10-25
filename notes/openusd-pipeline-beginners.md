# OpenUSD for Artists: A First Stage Tested in Houdini Solaris

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Build a five-file OpenUSD scene with references, payloads and season variants. Tested in Houdini Solaris, with downloadable files and composition checks.

- Author: Alicia Lu
- Published: 2026-10-19
- Updated: 2026-10-23
- Canonical HTML: https://digitalgarden.si/blog/openusd-pipeline-beginners/

OpenUSD becomes easier to understand when one asset appears twice in a scene and changing its source updates both copies. This experiment does exactly that: a cylinder and sphere stand in for a plant, two references place it in a garden, and a stronger shot layer changes its season.

**The practical starting point is five small files, not a production scene.** Keep geometry in a payload, give the asset an entry layer with a default prim, reference that entry layer into a layout, and put shot-specific choices in a separate override. The downloadable example was opened and checked with Houdini 22.0.429 and its bundled OpenUSD 26.5 on Windows. The fixture is deliberately schematic; it is not one of the production plants in [Digital Garden](https://digitalgarden.si/virtual-garden/).

![The same two referenced plants, one with the spring look and one with the autumn look, rendered through Houdini's bundled USD Storm renderer](https://digitalgarden.si/marketing/research/briefs-20261019/usd-garden.png)

The image is a render of the actual USD stage through Houdini's bundled Storm renderer. Headless Solaris loaded the files through Sublayer LOPs. Composition evidence was read with OpenUSD's property-stack API; this is not a screenshot of the interactive Scene Graph Details panel.

## The five-file project

The [example archive](https://digitalgarden.si/marketing/research/briefs-20261019/openusd-beginners.zip) contains the five authored layers, the native Component Builder output, the test script and measured results. Start with this structure:

```text
garden_usd/
  garden.usda
  autumn_shot.usda
  autumn_overrides.usda
  assets/lily/
    lily.usda
    payload.usda
```

A **layer** contains opinions: a radius, a color, a reference, a variant selection. A **prim** is an item at a path such as `/garden/lily_01`. A **stage** is the composed result of opening a root layer and resolving those opinions. **Composition** is the process that makes those files act like one scene.

An entry layer needs a useful root. The plant file sets `defaultPrim = "lily"`, so a reference can bring in the intended asset without spelling out its root path. This follows OpenUSD's referencing workflow.[^1] In the layout, the asset root is remapped from `/lily` to `/garden/lily_01`; a second reference maps it to `/garden/lily_02`.

The geometry layer has a 0.3 m cylinder and a small sphere. It uses Y-up coordinates and meters. The entry file carries a payload and the season variants:

```usda
#usda 1.0
(
    defaultPrim = "lily"
    metersPerUnit = 1
    upAxis = "Y"
)
def Xform "lily" (
    kind = "component"
    prepend payload = @./payload.usda@
    variants = { string season = "spring" }
    prepend variantSets = "season"
)
{
    # The complete spring/autumn definitions are in the archive.
}
```

The geometry is gray in `payload.usda`. A season variant authors a stronger display-color opinion on the petals. The spring selection yields approximately `(0.95, 0.95, 0.9)`; autumn yields `(0.8, 0.55, 0.2)`.

## Reference, payload, sublayer and variant: choose by the edit

| Mechanism | Use it for | What this fixture does |
| --- | --- | --- |
| Reference | Reuse an asset at a new scene path | Place the plant twice |
| Payload | Delay loading heavier content | Keep plant geometry behind the asset entry |
| Sublayer | Stack opinions in the same namespace | Put a shot override above the layout |
| Variant | Offer named alternatives on one prim | Select spring or autumn |

A reference is not automatically an instance. The two layout prims reuse the same source, but this particular beginner example does not set `instanceable = true`. That distinction matters when discussing memory or per-instance editing. For measured native and point instancing, see the separate [Houdini to Unreal test](https://digitalgarden.si/blog/houdini-unreal-usd/).

The shot layer lists the override first and the garden second:

```usda
subLayers = [
    @./autumn_overrides.usda@,
    @./garden.usda@
]
```

The override selects autumn for `lily_01`. Opening `garden.usda` gives one spring plant and one autumn plant. Opening `autumn_shot.usda` gives two autumn plants. Both results were asserted in the bundled OpenUSD API and in the stage returned by a Solaris Sublayer LOP.

## Find the opinion that actually wins

When a color appears unchanged, do not start by re-exporting everything. Inspect the attribute's property stack:

```python
attr = stage.GetPrimAtPath(
    "/garden/lily_01/geo/petals"
).GetAttribute("primvars:displayColor")
for spec in attr.GetPropertyStack():
    print(spec.layer.identifier, spec.path)
```

For the autumn shot, the first color opinion came from `/lily{season=autumn}geo/petals` in `lily.usda`. The weaker gray opinion came from `payload.usda`. The shot layer chooses the variant; the selected variant supplies the color. Those are different contributions.

There is a useful trap here: a local attribute value can override a value authored inside a variant. OpenUSD's own variant tutorial explicitly demonstrates clearing the local value before expecting variant colors to win.[^2] I also hit this while preparing the companion Unreal fixture: a radius authored outside the growth variants kept defeating the bud selection. Clearing that local radius made the visible size change work. That was a fixture authoring error, not an Unreal variant limitation.

## Unloading a payload does not mean an empty tree

One claim in the original draft did not survive the test. With `Usd.Stage.LoadNone`, both plant roots remained, but so did `/geo` and `/geo/petals` as over prims introduced by the selected variants. The cylinder stem, defined only in the payload, was absent.

Loading just `/garden/lily_01` brought back its stem while the other stem stayed absent. The useful test is therefore **whether the payload-defined geometry is populated**, not whether every descendant path disappears. Inspect `IsDefined()`, the prim type and payload state before treating a remaining tree item as loaded geometry.

This difference is easy to miss in a Scene Graph Tree. A prim path can exist because a stronger layer contains opinions for it even when the referenced geometry behind that path is unloaded.

## The native Solaris Component Builder version

I built a second version with native nodes: two Component Geometry nodes, Component Geometry Variants and Component Output. Each geometry branch contains the same simple stem and sphere, with a different SOP color. The variant names are `spring` and `autumn` under the `geo` set; these are geometry variants carrying color, rather than a separate MaterialX look-development test.

Component Output wrote `lily.usd`, `payload.usdc`, `geo.usdc` and `mtl.usdc`. Reopening the entry file returned both variant names, with no final node errors or warnings. SideFX describes this node family as a framework for reusable component models with separated geometry, materials, payloads and variants.[^3]

Use forward slashes in Houdini parameter paths. My first Python pass supplied Windows backslashes to an expression-evaluated parameter and produced escape warnings. Switching to `Path.as_posix()` removed them. The final downloadable Component Builder output is the corrected run.

## What the checks establish

| Check | Observed result |
| --- | --- |
| Five authored layers | Opened with bundled OpenUSD 26.5 |
| Base layout | Spring first plant, autumn second plant |
| Shot override | Both plants autumn |
| Property stack | Selected asset variant wins over payload gray |
| Payload control | Loading one plant restores only its payload-defined stem |
| Session variant switch | Spring → autumn → spring returned the expected colors |
| Native Component Builder | Two variants, four exported USD layers, clean final cook |
| Bundled `usdchecker` | Root stage and enumerated season selections passed |

Houdini's installation also includes `usdview.cmd` and `usdchecker.exe`. This test invoked `usdchecker` and used Storm for the render. It did not validate the interactive `usdview` GUI, compress a five-day learning exercise into a claimed learning outcome, or benchmark a large production stage.

For a first exercise, rebuild this folder, change the second reference's placement, add a winter selection, then create a shot that changes only one plant. If that works, you have used the core ideas rather than merely memorized their names. Keep this separate from format support in a receiving application: the [Three.js USD viewer test](https://digitalgarden.si/blog/openusd-threejs-usd-web-viewer/) demonstrates why a valid stage can still lose a material or primitive type in a particular viewer. The [two-plant USD and glTF export](https://digitalgarden.si/blog/houdini-usd-vs-gltf-threejs/) asks that question of Solaris output loaded by Three.js USDLoader. Two composed NVIDIA packages later posed it at a different scale: the [SimReady USD](https://digitalgarden.si/blog/simready-usd-to-glb-threejs-webgpu/) notes record which of Blender, Houdini, usd2gltf and direct loading kept materials.

## References

[^1]: OpenUSD, [Referencing Layers](https://openusd.org/release/tut_referencing_layers.html). Current online tutorial checked 19 October 2026; local results above use the separately recorded 26.5 build.
[^2]: OpenUSD, [Authoring Variants](https://openusd.org/release/tut_authoring_variants.html).
[^3]: SideFX, [Component Output](https://www.sidefx.com/docs/houdini/nodes/lop/componentoutput.html).
