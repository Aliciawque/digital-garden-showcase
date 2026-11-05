# Gaussian Splat Viewers on Desktop: PLY, SOG and SPZ

> Three local Gaussian splat viewer paths tested in Chrome. See which PLY, SOG and SPZ inputs load, and the PlayCanvas release-build worker failure.

- Author: Alicia Lu
- Published: 2026-11-05
- Canonical HTML: https://digitalgarden.si/blog/gaussian-splat-viewers-desktop/

## Key takeaways

- The local PlayCanvas Model Viewer debug build rendered PLY, compressed PLY and bundled SOG with 4,096 decoded splats each.
- The same viewer file input did not load SPZ v3 or v4. SuperSplat standalone SOG and antimatter15 PLY also rendered locally.

I used one 4,096-splat synthetic garden to check three ways to view Gaussian splats locally in Chrome. SuperSplat's generated HTML and antimatter15/splat both rendered the garden. A pinned PlayCanvas Model Viewer debug build rendered original PLY, compressed PLY and bundled SOG. The same viewer did not load SPZ v3 or v4 through its local file input. Its release build hit a worker error before displaying the three supported files.

## Three desktop paths

| Viewer path | Local input actually tried | Desktop result |
| --- | --- | --- |
| SuperSplat Viewer standalone | SOG embedded by splat-transform 3.10.3 | Visible garden in a 960 × 720 canvas |
| antimatter15/splat at ba182b51b7c2 | Original PLY by absolute same-origin URL | Visible garden and a WebGL instanced draw with 4,096 instances |
| PlayCanvas Model Viewer 5.10.0, debug WebGL2 | PLY, compressed PLY and bundled SOG | All three rendered and each decoded resource held 4,096 splats |
| PlayCanvas Model Viewer 5.10.0, debug WebGL2 | SPZ v3 and SPZ v4 | Both left the load screen visible with no decoded splat resource |

![SuperSplat Viewer displaying the garden from the generated HTML with an embedded SOG payload](https://digitalgarden.si/marketing/research/splat-formats-20261105/viewer-supersplat-sog.png)

![PlayCanvas Model Viewer debug build rendering the bundled SOG with its scene panel visible](https://digitalgarden.si/marketing/research/splat-formats-20261105/model-viewer-debug-resource-garden-sog.png)

The screenshots use the viewers' own camera and UI. I checked file loading and actual drawing. The [format comparison](https://digitalgarden.si/blog/spz-vs-sog-vs-ply-gaussian-splat-formats/) uses one renderer and matched cameras for its separate image-error and startup measurements.

## Replaying local files

For SuperSplat Viewer, this command generates a standalone HTML page that embeds the SOG data. My generated file was 3,892,390 bytes including the viewer.

```bash
npx splat-transform -g cpu garden.ply garden.html
```

For antimatter15/splat, serve the upstream HTML and JavaScript from a static server and set the url query parameter to an absolute same-origin PLY URL. A relative value resolves against the viewer's default asset host. My initial relative-URL probe attempted to reach that host, which the harness blocked. The final replay issued only local GET requests.

The PlayCanvas test used the open-source Model Viewer at commit 34df5967df32 with PlayCanvas Engine 2.19.7. I built the release and debug targets from the same source and lockfile. The release build hit a ReferenceError in its blob worker on WebGL2 and WebGPU. The error also appeared after the panel reported 4,096 primitives for PLY. The rendered garden was absent, so those runs did not pass. The upstream debug build ran the three supported formats on WebGL2. Its compressed PLY and SOG scene panels showed zero primitives even when the garden was visibly rendered. I checked the decoded resource count for those rows. This describes the pinned local builds. The hosted viewer and the engine's separate SPZ parser have their own paths. [Model Viewer source](https://github.com/playcanvas/model-viewer/tree/34df5967df32032d34e8833cd29f8d0f19c7a93a), [Model Viewer formats](https://developer.playcanvas.com/user-manual/gaussian-splatting/viewing/), [PlayCanvas Engine SPZ parser](https://developer.playcanvas.com/user-manual/gaussian-splatting/formats/spz/).

## What the desktop check covers

The successful local replays required no login and made only local GET requests in the isolated test. The network evidence covers these specific self-hosted paths, with any nonlocal traffic blocked by the harness. It says nothing about upload and account behavior on third-party hosted services. The rendered screenshots and source file receipts are included. Physical iPhone and Quest 3 operation, immersive modes and hosted service policies remain separate tests. [SuperSplat Viewer](https://github.com/playcanvas/supersplat-viewer), [antimatter15/splat](https://github.com/antimatter15/splat/tree/ba182b51b7c2ad5738cdd6741cd63336d27470fb).

Alicia is a 3D technical artist, TA and independent developer. Codex assisted with the test harness, evidence checks and editing.

## Related studies

- [Five encodings of one splat garden](https://digitalgarden.si/blog/spz-vs-sog-vs-ply-gaussian-splat-formats/)
- [Five Three.js splat renderer paths](https://digitalgarden.si/blog/threejs-gaussian-splat-renderer-comparison/)

## Data and reproduction

- [viewers.json](https://digitalgarden.si/marketing/research/splat-formats-20261105/viewers.json)
- [playcanvas-viewer-debug-resource.json](https://digitalgarden.si/marketing/research/splat-formats-20261105/playcanvas-viewer-debug-resource.json)
- [splat-viewers-desktop-reproduction.zip](https://digitalgarden.si/marketing/research/splat-formats-20261105/splat-viewers-desktop-reproduction.zip)

## Questions

### Did the tested PlayCanvas Model Viewer open SPZ?

The pinned 5.10.0 local file-input path did not load either SPZ version. The PlayCanvas Engine exposes a separate SPZ parser.
