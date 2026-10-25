# Houdini Assets vs Runtime Geometry: slop-newyork and My Garden, Profiled

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Desktop WebGPU profiles of slop-newyork and Digital Garden: full download budgets, three 1080p runs, draw calls and the limits of comparing different scenes.

- Author: Alicia Lu
- Published: 2026-10-23
- Canonical HTML: https://digitalgarden.si/blog/houdini-vs-code-procedural-threejs-slop-newyork/

The most useful number in this comparison was the one missing from the initial headline: the library download. My local copy of slop-newyork transferred about 785 KB of compressed project files, close to the 784 KB figure in its launch post. It also fetched about 777 KB of compressed JavaScript from the Three.js CDN. Counting both made the observed encoded payload about 1.56 MB.

That does not make the original number wrong. It makes the boundary important. The same rule applies to my garden: a plant GLB, the first seasonal load and every background-preloaded season are different download budgets.

![Actual slop-newyork city and spring Digital Garden recordings with OpenStreetMap and NYC Open Data credits](https://digitalgarden.si/marketing/research/daily-20261023/city-garden-comparison-preview.jpg)

Tested locally with Codex on 23 October 2026. [Download the measured results](https://digitalgarden.si/marketing/research/daily-20261023/city-garden-results.json). The image is from a separate recording, not one of the timed trials.

## Two real scenes, two mixed pipelines

I pinned [mrdoob's slop-newyork](https://github.com/mrdoob/slop-newyork/tree/0049a497be6e36c79f854771f3018f8f41f07fab) at `0049a497be6e36c79f854771f3018f8f41f07fab`. I inspected its live demo and ran that source locally. It builds city geometry and TSL materials at runtime around generated geographic data; the captured load did not fetch GLB or texture files. “Procedural” does not mean “no data download.”

My garden was built from local Git revision `7a9d1cf29a60d949dbc3e72634eb53112c797182`, using an isolated output directory. Its plant manifests record Houdini 22.0.429 and metre/Y-up GLB assets. But the garden layout is not a whole scene baked in Houdini: deterministic planting is implemented in TypeScript, and the browser handles instancing, LOD, wind, lighting and interaction.

The city loaded Three.js **187dev** from an unpinned `@dev` CDN URL. The garden's package lock pins **0.186.1**. This is a profile of two applications, not an A/B experiment isolating the effect of Houdini, JavaScript or a renderer version.

## What went over the wire

I used the browser's Resource Timing entries, disabled the HTTP cache through DevTools protocol, and served local files with gzip level 9 and `Cache-Control: no-store`. CDN responses retained their real delivery encoding. These are encoded response-body bytes, excluding protocol/header overhead; decimal KB/MB are used below. This is not a public-CDN latency benchmark or a clean device/GPU-cache reset.

The “before ready” budget includes requests **started before the application's loading overlay disappeared**, even if their responses finished later. A small injected readiness probe adds a few hundred HTML bytes. The raw receipts preserve each resource URL and its encoded, decoded and transfer sizes.

| Observed response-body category | slop-newyork | Garden, first spring load |
| --- | ---: | ---: |
| Application JavaScript | 387,234 B | 359,646 B |
| Geographic data | 384,199 B | — |
| Three.js CDN JavaScript | 777,277 B | Included in app bundles |
| GLBs | 0 B | 2,982,529 B across 36 requests |
| HTML, CSS, manifests and other files | 13,710 B | 23,783 B |
| External font CSS observed | 723 B | 0 B |
| **Before-ready total** | **1,563,143 B** | **3,365,958 B** |

The captured city dependency graph included `three.webgpu.js`, `three.tsl.js`, `three.core.js` and addons. Looking only at the repository's JS files would miss those requests. The first-party subtotal is 785,143 B; it is a measured local subtotal, not an exact reproduction of the author's compression process.

The garden continued preloading. At the later recorded snapshot, its cumulative observed response bodies totaled 18,369,320 B, including 72 GLB requests. That is a later network snapshot, not a required first frame payload or a complete four-season download inventory. It would be misleading to put that number beside the city's initial project-only subtotal and call the ratio a pipeline result.

## Same render size, different work

The machine was a Ryzen 9 9950X with an RTX 5070 Ti, Windows and Chromium 155. Both applications actually rendered through WebGPU at **1920×1080, renderer DPR 1**, displayed in a 1280×720 browser viewport. I checked drawing-buffer dimensions; the garden's `?benchmark` mode fixes them to 1080p regardless of the CSS viewport. An earlier 1280×720 city run is retained only as an exploratory record.

I ran three 30-second observations per scene, with one GPU test page active at a time. For the city, the sequence spent ten seconds each on chase, observatory and drone presets. The preset order is repeatable; the moving city and director do not create pixel-identical trajectories. The garden used its resettable 40-second benchmark orbit, sampled for 30 seconds, with the spring scene and default high quality. Each scene kept its own lights and effects.

| Observation | slop-newyork | Spring garden |
| --- | ---: | ---: |
| Average FPS, run 1 / 2 / 3 | 59.79 / 59.96 / 58.89 | 52.25 / 50.84 / 50.01 |
| Median of run averages | 59.79 | 50.84 |
| Median frame interval | 16.7 ms | 16.7 ms |
| 95th-percentile frame interval | 16.8 ms in all runs | 33.4 ms in all runs |
| Median reported draw calls by run | 663 / 675 / 692 | 68 / 68 / 68 |
| Median submitted triangles by run | 10.11M / 9.79M / 10.17M | 9.34M / 9.62M / 9.37M |

Frame intervals came from `requestAnimationFrame`; counters came from each application's existing renderer telemetry. The garden publishes its counters periodically, so repeated samples of one counter value are not independent frame observations. Counters include each application's rendering passes and are not an audited comparison of identical GPU commands. Submitted triangles are not the number of unique mesh triangles in the project.

The city was close to this display's 60 Hz ceiling. I cannot infer its uncapped throughput from 60 FPS. The garden's much lower draw-call count did not guarantee a lower frame time: draw calls alone leave out pixel cost, material complexity, overdraw and rendering passes. This test did not isolate which of those costs caused the difference.

## Claims these measurements cannot support

These measurements do not establish that code procedural is smaller in general, that Houdini is faster, or that my whole garden layout was baked. Neither application is a pure category. No matched hybrid asset replacement was performed, so there is no measured hybrid byte-saving result.

I did capture application-visible readiness during local loading, but did not turn that into a first-frame ranking. Overlay transitions, compilation, HTTP fetches, GPU caches and concurrent setup work have different scopes. The evidence contains exploratory timestamps; it does not contain an accepted device-cold, GPU-fenced first-frame comparison for these applications.

No physical phone, WebGL2 performance comparison, GPU timer-query study, author token cost or untouched AI-agent benchmark was tested here. The launch post's reference to tokens does not establish exactly how every part of the project was authored.

## The production decision I would take from this

I would use compact runtime parameters when many variations can be reconstructed cheaply, and authored assets where shape control and close-up quality justify the geometry payload. For the garden, that means keeping its current mix: procedural plant authoring, explicit web assets, and runtime planting and motion. A further optimization needs a matched element, equal appearance targets and measured quality loss; these two complete scenes cannot supply that result by themselves.

The immediately actionable habit is to report three numbers separately: project files, shared dependencies and the actual initial load. Then report later preloads as a different budget. That makes a small demo's achievement easier to understand without turning it into a universal winner.

## Credits and evidence

slop-newyork code © its authors, MIT. City geography: © OpenStreetMap contributors, [ODbL](https://www.openstreetmap.org/copyright), and NYC Open Data; retain these credits with the screenshots and video. Digital Garden imagery © Alicia. Source, license and provenance are retained with the local clone.

[Results and method](https://digitalgarden.si/marketing/research/daily-20261023/city-garden-results.json). Raw frame observations: [city-1080 run 1](https://digitalgarden.si/marketing/research/daily-20261023/city-1080-frames-1.json), [city-1080 run 2](https://digitalgarden.si/marketing/research/daily-20261023/city-1080-frames-2.json), [city-1080 run 3](https://digitalgarden.si/marketing/research/daily-20261023/city-1080-frames-3.json), [garden run 1](https://digitalgarden.si/marketing/research/daily-20261023/garden-frames-1.json), [garden run 2](https://digitalgarden.si/marketing/research/daily-20261023/garden-frames-2.json), [garden run 3](https://digitalgarden.si/marketing/research/daily-20261023/garden-frames-3.json). The byte table uses Resource Timing rather than the truncated diagnostic network-event log.

Continue with the [Three.js rendering guide](https://digitalgarden.si/blog/threejs-3d-garden-rendering/) or [visit the garden](https://digitalgarden.si/play/).
