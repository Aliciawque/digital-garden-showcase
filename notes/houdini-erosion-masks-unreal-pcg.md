# A Mountain River in Unreal PCG: Testing a Houdini Erosion Mask

> Nine native Unreal PCG runs test Houdini river and flow masks. Both masks remove all 134 in-channel candidates from a 3,844-point test grid.

- Author: Alicia Lu
- Published: 2026-11-04
- Canonical HTML: https://digitalgarden.si/blog/houdini-erosion-masks-unreal-pcg/

## Key takeaways

- The unmasked grid had 134 candidates inside the authored river corridor. Both mask variants reduced that number to zero.
- Adding the Houdini flow rule removed three more candidates. All observed PCG coordinates matched an independent bilinear check.

Houdini's native erosion changed a 1,008 m valley and produced a flow layer. I exported binary placement masks, brought them into Unreal 5.8.4 and ran the same native PCG grid with three controls. Of 3,844 unmasked candidates, 134 were inside the authored river corridor. A channel-only mask and a channel-plus-flow mask both reduced that number to zero. Flow removed three additional candidates near the channel for this grid and threshold.

![Measured Houdini eroded heights, height changes and native flow on the 1,008 m tile](https://digitalgarden.si/marketing/research/pcg-river-20261104/terrain-evidence.png)

## Construct the slope, then measure erosion

I built a 512 × 512 Houdini HeightField on a 1,008 m square, at 1.96875 m voxel spacing. A deterministic procedural surface contains a downward slope, a sinuous channel corridor and noisy valley sides. HeightField Erode 3.0 froze at frame 9. A forced cook of the erosion node took 0.871 s on this machine. Across the tile, the eroded elevation ranged from 0.84 to 185.17 m. Relative to the authored surface, height changes ranged from −28.35 to +10.77 m.

The channel is defined by the authored centerline and an 18 m half-width. The combined exclusion also removes pixels within 60 m of that centerline when normalized native flow exceeds 0.45. Positive flow was divided by its 98th percentile, 7.3656 in the exported layer's units, and clipped. The channel excludes 9,378 of 262,144 source pixels. Adding the flow rule excludes 133 more. This makes the extra contribution measurable without treating the flow layer as an independent water simulation. [SideFX HeightField Erode reference](https://www.sidefx.com/docs/houdini/nodes/sop/heightfield_erode.html).

Houdini's voxel index order put its world Z in the fast axis. I reordered the native data before writing a PNG with X along columns and Z along rows. The PNG alpha carries the allowed mask. The graph reads that channel with bilinear filtering and a 0.5 threshold. The additional 505 × 505, 16-bit heightmap is a center crop intended for future Landscape import. I decoded it independently against the source heights and measured a maximum quantization error of 3.922 mm.

## Native PCG on the same grid

The isolated Unreal map used 62 × 62 points at 16 m spacing. The graph was Create Points Grid → Sample Texture → Density Filter. The Get Texture Data node uses a 100,800 cm texture footprint. The Houdini XZ plane maps to Unreal XY. The mask textures were imported with sRGB disabled, no mipmaps and uncompressed data settings. All three graph variants used the same editor scene and grid.

| Control | Candidate output points | Within authored corridor | Mismatches against independent bilinear check |
| --- | --- | --- | --- |
| All allowed | 3,844 | 134 | 0 |
| Channel exclusion | 3,708 | 0 | 0 |
| Channel and flow exclusion | 3,705 | 0 | 0 |

Each case was forced three times after reopening the saved Unreal map, and each case returned the same coordinates across its three runs. The independent comparison used the actual baseline XY positions and reconstructed sampling from the saved mask. A nearest-texel check differed at two channel-only points and three combined-mask points. The bilinear check matched all retained XY positions. The extra two channel-only removals outside the analytic corridor come from sampling the raster boundary with a threshold.

The first baseline reached the editor's generation event in 424.2 ms. All eight later requests completed in 14.9–18.0 ms in the same editor session. They may reuse compiled graph work and cached data. These are request-to-event latencies, with no claim about packaged runtime, mesh rendering or per-node CPU time. Earlier null-renderer launches timed out; the successful run used offscreen D3D11. [Epic PCG node reference](https://dev.epicgames.com/documentation/en-us/unreal-engine/procedural-content-generation-framework-node-reference-in-unreal-engine).

![Native PCG coordinates over the authored channel for the three cases; red dots mark in-channel candidates](https://digitalgarden.si/marketing/research/pcg-river-20261104/native-pcg-evidence.png)

## Scope

This is an Unreal editor test of placement eligibility. Native PCG returned point coordinates on a plane at Z = 0. It did not import a Landscape, project points onto terrain, spawn production foliage, render water or run a packaged game. The exported heightmap and separate wetness mask are available for that follow-up. In this fixture, the channel mask alone was sufficient for excluding all tested in-channel points. The flow layer changed a narrow part of the surrounding placement budget.

[Reproduction archive and data](https://digitalgarden.si/marketing/research/pcg-river-20261104/pcg-river-reproduction.zip). Codex implemented the fixture, ran and audited the native tests, and drafted the article.

Related study: [Digital Garden field note](https://digitalgarden.si/blog/threejs-forest-performance-budget/).

### Raw measurement files

- [erosion.json](https://digitalgarden.si/marketing/research/pcg-river-20261104/erosion.json)
- [placement-summary.json](https://digitalgarden.si/marketing/research/pcg-river-20261104/placement-summary.json)

## Questions

### Did the test project points onto an Unreal Landscape?

No. The native PCG graphs filtered a fixed flat candidate grid. Landscape projection and packaged runtime were outside the measurement.
