# Houdini GLB to USDZ with Blender, Three.js and Three-Live

> One Houdini plant, three desktop USDZ export paths. Compare geometry, texture retention, world bounds, double-sided leaves and OpenUSD readback.

- Author: Alicia Lu
- Published: 2026-11-05
- Canonical HTML: https://digitalgarden.si/blog/threejs-glb-to-usdz-desktop/

## Key takeaways

- All four USDZ files reopened with 1,056 triangles. Blender 5.2 kept the source bounds, two textures and double-sided leaves.
- The r187 Quick Look option did not change this file. Phone playback remains a separate test.

I converted the same textured Houdini plant with Three.js r187, Three-Live 1.2.3 and Blender 5.2 LTS. All three exports reopened in Houdini with 1,056 triangles. Blender preserved the original world bounds, two textures and the double-sided leaves. Three.js preserved the bounds and textures. Three-Live produced the smallest package, with a shifted bounding box and no linked textures.

![Houdini plant in Chrome before USDZ conversion, showing the stem and three leaves](https://digitalgarden.si/marketing/research/three-live-usdz-20261105/plant-source.png)

## One input and three export paths

The 947,652-byte input is the owned 512-pixel GLB from the [Copernicus bake study](https://digitalgarden.si/blog/houdini-copernicus-bake-glb-threejs/). It has 628 vertices, 1,056 triangles, a normal map, a packed occlusion-roughness-metallic map and a double-sided material.

- **Three.js r187.** In Chrome 156 on an RTX 5070 Ti, I loaded the GLB with GLTFLoader and ran USDZExporter with maxTextureSize 1024. I exported three times per Quick Look option.
- **Three-Live 1.2.3.** The verified Windows x64 CLI read the same GLB and exported USDZ directly.
- **Blender 5.2.0 LTS.** A factory-startup background session imported the same GLB and exported a Y-up, meter-scale USDZ with USD Preview Surface. A fresh Blender scene then imported the USDZ, recovering 628 vertices, 1,056 faces and two packed images.

```powershell
three-live-cli.exe convert input/houdini-plant.glb -f usdz -o evidence/plant-three-live.usdz
blender --background --factory-startup --python-exit-code 2 --python export_blender.py
```

```js
const { scene } = await new GLTFLoader().loadAsync('/input/houdini-plant.glb');
const bytes = await new USDZExporter().parseAsync(scene, {
  quickLookCompatible: true,
  maxTextureSize: 1024
});
```

## What survived when I reopened the packages

| Path | USDZ bytes | Stored ZIP members | Resolved image files | Points / triangles | Double-sided | World Y range (m) |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| Three-Live 1.2.3 | 66,910 | 1 | 0 | 628 / 1,056 | false | −0.9923 to 1.1887 |
| Three.js r187, option off | 364,858 | 4 | 2 | 628 / 1,056 | false | 0.0000 to 2.0287 |
| Three.js r187, option on | 364,858 | 4 | 2 | 628 / 1,056 | false | 0.0000 to 2.0287 |
| Blender 5.2 LTS, Y up | 957,384 | 3 | 2 | 628 / 1,056 | true | 0.0000 to 2.0287 |

Houdini 22.0.429's OpenUSD 0.26.5 reopened all four packages. All entries passed CRC, store-method and 64-byte alignment checks. Each stage declared Y up and one meter per unit. Three.js and Blender preserved the GLB world bounds to within 0.000001 m per component. The Three-Live file had one USDA layer and a bound UsdPreviewSurface with no linked texture files.

The Blender package used a binary USDC layer and two PNG textures. Its normal PNG accounts for most of its 957,384 bytes. The result supports a desktop handoff where the leaf backs and floor position matter. The smaller Three.js result is another candidate when the mesh's single-sided export is acceptable for the intended camera.

## What the Quick Look option changed on this plant

All six Three.js exports were byte-identical, across both settings. The exporter warned about the double-sided input in each run. Every Three.js USD mesh reopened with doubleSided false. The result is specific to this asset and exporter build.

The included device checklist links the saved USDZ files. A physical iPhone can then check texture appearance, back-facing leaves, initial scale and placement. This release reports the desktop export and reopen. Apple documents the device playback path separately. [Apple Quick Look](https://developer.apple.com/quick-look-gallery/), [Three.js USDZExporter](https://threejs.org/docs/#USDZExporter), [Three-Live 1.2.3](https://github.com/google/three-live/releases/tag/v1.2.3), [Blender USD manual](https://docs.blender.org/manual/en/latest/files/import_export/usd.html).

Alicia is a 3D technical artist, TA and independent developer. Codex assisted with the test harness, evidence checks and editing.

## Related studies

- [Houdini Copernicus bake for Three.js](https://digitalgarden.si/blog/houdini-copernicus-bake-glb-threejs/)
- [Houdini USD vs glTF for Three.js](https://digitalgarden.si/blog/houdini-usd-vs-gltf-threejs/)

## Data and reproduction

- [openusd.json](https://digitalgarden.si/marketing/research/three-live-usdz-20261105/openusd.json)
- [blender-reopen.json](https://digitalgarden.si/marketing/research/three-live-usdz-20261105/blender-reopen.json)
- [three-usdz-desktop-reproduction.zip](https://digitalgarden.si/marketing/research/three-live-usdz-20261105/three-usdz-desktop-reproduction.zip)

## Questions

### Which exporter retained the double-sided Houdini leaves?

Blender 5.2 exported a double-sided USD mesh in this desktop test. Both Three.js settings and Three-Live reopened as single-sided.
