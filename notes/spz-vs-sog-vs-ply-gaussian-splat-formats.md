# SPZ vs SOG vs PLY: Five Exports of One Gaussian Splat Garden

> One 4,096-splat garden, five encodings and 15 Chrome runs compare PLY, compressed PLY, SPZ v3/v4 and SOG by file size, startup and image error.

- Author: Alicia Lu
- Published: 2026-11-05
- Canonical HTML: https://digitalgarden.si/blog/spz-vs-sog-vs-ply-gaussian-splat-formats/

## Key takeaways

- SPZ v4 was the smallest at 62,011 bytes on a 4,096-splat SH0 garden, versus 278,942 bytes for the original PLY.
- Compressed PLY had the lowest full-frame RGB error against the original PLY across three fixed Spark views.

On one 4,096-splat garden, SPZ v4 was the smallest file at 62,011 bytes, down from 278,942 bytes for the original PLY. Compressed PLY had the smallest image error in the three tested Spark views. SOG landed between compressed PLY and SPZ on that image comparison. The best pick depends on the viewer's supported formats and whether the delivery server applies additional compression.

## One source, one renderer

I started with an owned synthetic scene fitted from Karma renders in the [Houdini procedural splatting study](https://digitalgarden.si/blog/houdini-procedural-gaussian-splatting/). The source has 4,096 splats and SH degree 0. I encoded the exact same PLY with splat-transform 3.10.3, then loaded every result through Spark 2.4.0 and Three.js r187. The browser was Chrome 156.0.8078.80 with WebGL on an RTX 5070 Ti.

```bash
npx splat-transform garden.ply garden.compressed.ply
npx splat-transform --spz-version 3 garden.ply garden-v3.spz
npx splat-transform --spz-version 4 garden.ply garden.spz
npx splat-transform -g cpu garden.ply garden.sog
```

## Size, first submission and image difference

| File | Raw bytes | Brotli q11 bytes | First submit median (ms) | Full-frame RGB PSNR vs PLY (dB, three views) |
| --- | ---: | ---: | ---: | --- |
| PLY | 278,942 | 212,282 | 143.6 | reference |
| Compressed PLY | 67,344 | 66,656 | 147.8 | 54.37 / 53.42 / 54.18 |
| SPZ v3 | 68,140 | 68,145 | 164.1 | 49.38 / 48.73 / 49.35 |
| SPZ v4 | 62,011 | 62,015 | 150.7 | 49.38 / 48.73 / 49.35 |
| Bundled SOG | 69,537 | 63,049 | 144.3 | 50.19 / 49.62 / 50.10 |

Each median covers three fresh browser contexts on loopback, with an uncached asset request. Timing runs from navigation's performance origin until the first `renderer.render()` and blocking `gl.finish()`. The early submission includes module setup and shader work. One raw PLY sample took 414.1 ms, while the other two took 132.6 and 143.6 ms. These three-sample results describe the startup runs and are too sparse for a speed ranking. Brotli measurements are offline byte counts, with no network latency or server configuration in the comparison.

After the scene settled, I rendered three fixed views at 768 × 768. The table compares their full 8-bit RGB images, including the background. It measures encoding differences under this renderer. The two SPZ versions gave identical RGB captures in all three views, with different on-disk sizes. Full-frame PSNR can hide small changes around leaves and other thin edges.

![The fixed oblique view of the encoded SOG garden in Spark](https://digitalgarden.si/marketing/research/splat-formats-20261105/garden-sog-view1.png)

## Where each result applies

- Keep a full PLY master for editing and repeat exports. Check the actual splat schema when receiving a PLY from another tool.
- SPZ v4 saved about 78% against the source PLY in this test. This file uses Zstandard. SPZ v3 uses gzip and remains an option when a viewer needs that version.
- SOG bundles quantized WebP attribute images and metadata. It shrank by another 6,488 bytes under offline Brotli q11 here. Streamed SOG and remote range loading remain outside this comparison.
- A separate 124-splat synthetic SH3 control encoded and decoded as SPZ v4 and SOG, retaining three SH bands in splat-transform's readback. That is a structural check on a different asset. The main five-format timing and visual comparisons are SH0.

The test did not measure per-process GPU memory, CDN performance, large captures or mobile devices. Supported formats depend on both the reader and the format version. For example, PlayCanvas documents its SPZ parser separately from the Model Viewer drag-and-drop list. [Niantic SPZ](https://github.com/nianticlabs/spz/blob/main/README.md), [PlayCanvas SOG](https://developer.playcanvas.com/user-manual/gaussian-splatting/formats/sog/), [PlayCanvas Model Viewer](https://developer.playcanvas.com/user-manual/gaussian-splatting/viewing/).

Alicia is a 3D technical artist, TA and independent developer. Codex assisted with the harness, evidence checks and editing.

![Two matched camera angles across original PLY, compressed PLY, SPZ v4 and bundled SOG](https://digitalgarden.si/marketing/research/splat-formats-20261105/format-contact-sheet.png)

## Related studies

- [Five Three.js splat renderer paths](https://digitalgarden.si/blog/threejs-gaussian-splat-renderer-comparison/)
- [Gaussian splat budget planner](https://digitalgarden.si/tools/gaussian-splat-budget-planner/)

## Data and reproduction

- [format-report.json](https://digitalgarden.si/marketing/research/splat-formats-20261105/format-report.json)
- [splat-formats-reproduction.zip](https://digitalgarden.si/marketing/research/splat-formats-20261105/splat-formats-reproduction.zip)

## Questions

### Which Gaussian splat format was smallest in this test?

SPZ v4 at 62,011 bytes was smallest before HTTP compression. The result applies to one 4,096-splat SH0 garden.
