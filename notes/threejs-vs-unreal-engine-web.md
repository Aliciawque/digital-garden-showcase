# Three.js vs Unreal Engine for Web 3D: Which Should You Ship in the Browser?

> Choose between Three.js, Unreal Wasm and Pixel Streaming, with 48 desktop Chrome runs covering Brotli, caches and local network shaping.

- Author: Alicia Lu
- Published: 2026-11-11
- Canonical HTML: https://digitalgarden.si/blog/threejs-vs-unreal-engine-web/

**Start with Three.js for a web-authored viewer or interactive page. Consider Unreal's experimental Wasm target when keeping an existing Unreal project is worth the larger client and Preview restrictions. Consider Pixel Streaming when the Unreal application should run on a server.** That is a product decision, rather than a universal performance ranking.

This desktop study was run on 11 November 2026 by Codex under Alicia's direction. Alicia is a 3D technical artist (TA) and independent developer. It adds 48 measured Chrome page loads (24 fresh-context visits and 24 same-context reloads) to the [previous Unreal Wasm build test](https://digitalgarden.si/blog/unreal-engine-wasm-webgpu/), with matched hosting software, Brotli compression and explicit cache policies. Public HTTPS hosting and iPhone Safari remain outside this desktop study. Local bandwidth shaping is not a measurement of a public network.

## Three browser delivery paths

| Path | Where the 3D application runs | What the browser receives | Decision to make |
|---|---|---|---|
| Three.js web client | On the visitor's device | JavaScript and scene assets | Can the experience be authored and maintained as a website? |
| UE 5.9 Preview Wasm client | On the visitor's device | A packaged engine client, WebAssembly and content | Is preserving the Unreal project worth the client download and experimental target? |
| Unreal Pixel Streaming | On a desktop or server running Unreal | WebRTC frames and audio, with input returned to the application | Is a streamed session acceptable, and can the service operate it? |

This three-path frame is our decision aid. Epic documents the experimental [Wasm target in the 5.9 release notes](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-5-9-release-notes) and describes [Pixel Streaming](https://dev.epicgames.com/documentation/en-us/unreal-engine/pixel-streaming-in-unreal-engine) as remotely rendered frames and audio delivered over WebRTC. Pixel Streaming latency, server costs and concurrent sessions were not measured in this study.

The [MegaJam Preview announcement](https://forums.unrealengine.com/t/unreal-engine-5-9-megajam-preview/2852840) calls the browser target experimental, identifies WebAssembly, WebGPU and the mobile renderer, and recommends Chrome. Its source-build/C++ guidance should be read alongside the known issues. Our earlier Windows Launcher result is evidence for one installed build, not a guarantee for every platform or project. Existing production projects were preserved separately.

## What the desktop test actually compares

The machine was Windows with an NVIDIA RTX 5070 Ti; the browser was Chrome 157.0.8103.50, headless, at 1280 × 720 and device pixel ratio 1. The Unreal package was UE 5.9.0 MegaJam Preview CL 60351404, Shipping, using the forward WebGPU tier. The Three.js control used 0.186.1, reported revision 186, and actually selected its WebGPU backend.

Both fixtures have a floor, two upright boxes, a rotating center cube and a fixed camera. The original fixture sources and UE Shipping package were reused unchanged. Materials, lighting, shaders and engine behavior differ. The screenshots show that difference; this is not matched visual quality or a garden-scale scene.

![UE Shipping four-box fixture rendered through WebGPU on the tested desktop.](https://digitalgarden.si/marketing/research/threejs-vs-unreal-20261111/ue-desktop.png)

![Three.js WebGPU control with the same box layout but different materials and lighting.](https://digitalgarden.si/marketing/research/threejs-vs-unreal-20261111/three-desktop.png)

Each stack ran three fresh browser contexts, each followed by a reload in the same context, for each of two network profiles and two cache policies: 48 measured page loads in total, using two Playwright browser launches overall, one per cache-policy dataset. Stack order alternated between pairs. Fresh-context means fresh browser storage, not a reboot, a cold GPU driver or a reset of every browser-process and OS cache.

Both stacks used Epic's packaged local server, loopback HTTP, the same isolation headers and Brotli siblings. The second profile used its server-side 50 Mbit/s shared-bandwidth and 40 ms response-delay setting. Server-side shaping also covers the UE service worker's fetches; the profile still does not reproduce a VPS, TLS, DNS, routing or a mobile radio.

For the primary table, versioned assets used the server's immutable-cache option and entry documents retained its no-cache policy. This is appropriate only for fixed, versioned test artifacts; a production site must change asset URLs when their content changes.

## Desktop startup and transfer results

Every cell in the primary startup table below is the median of three runs. Times are milliseconds from navigation start. Response-body bytes were summed from the server ledger after 120 sampled post-ready animation intervals and an observed zero-in-flight response count, before screenshots. They exclude HTTP headers, TLS and transport overhead. These are bytes for that observation window, not a folder size and not a count stopped at READY.

The common hook records the first successful GPU queue submission after a configured canvas acquires a texture. It provides one instrumented API boundary across stacks; it does not establish GPU completion, display scanout or that the first submission contains the final scene. The separate readiness column ends at UE's page READY event or the Three.js control's first rendered submission. Those readiness signals do different work.

| Stack | Network profile | Visit | First canvas submission, ms | Stack readiness, ms | Completed response bodies, B |
|---|---|---|---:|---:|---:|
| Three.js r186 | Unshaped loopback | Fresh context | 199.07 | 199.08 | 533,231 |
| Three.js r186 | Unshaped loopback | Same-context reload | 95.13 | 95.14 | 124 |
| Three.js r186 | Local 50 Mbit/s + 40 ms | Fresh context | 483.83 | 483.84 | 533,231 |
| Three.js r186 | Local 50 Mbit/s + 40 ms | Same-context reload | 114.26 | 114.27 | 124 |
| UE 5.9 Wasm | Unshaped loopback | Fresh context | 2043.05 | 2049.62 | 50,111,928 |
| UE 5.9 Wasm | Unshaped loopback | Same-context reload | 1688.59 | 1701.52 | 40,838 |
| UE 5.9 Wasm | Local 50 Mbit/s + 40 ms | Fresh context | 15493.15 | 15507.72 | 57,537,511 |
| UE 5.9 Wasm | Local 50 Mbit/s + 40 ms | Same-context reload | 1607.65 | 1612.54 | 40,838 |

The small Three.js control had the smaller transfer and earlier submission in these conditions. That observation supports a lightweight web-client option for this fixture. It does not give a speedup ratio for equally rendered scenes, predict the full garden, or settle the cost of porting an existing Unreal project.

The UE cold transfer changed with the shaped profile even though the package was identical. On unshaped loopback it was 50,111,928 B. On the shaped profile it was 57,537,511 B. The shaped cold ledgers record 14 Range responses totaling 7,425,583 B alongside whole-file prefetch responses; the unshaped cold ledgers record no Range responses. Count the actual requests when estimating network cost. A Brotli file's size alone misses this observed overlap.

The earlier build article reports its own before-READY byte count and an uncompressed Three.js control. Those historical figures remain unchanged. This table uses a different, explicitly defined observation window and compressed control, so it should not silently replace the earlier numbers.

## Cache policy is part of the comparison

The server's default policy is no-cache and it provides no validators for these responses. In the default-policy control, the Three.js files were fetched again on reload. UE also used its own service-worker content cache. Enabling the same immutable option on both servers changed the warm byte totals:

| Stack | Server policy | Reload response bodies, B | Reload readiness, ms |
|---|---|---:|---:|
| Three.js | Default no-cache, no validators | 533,231 | 116.46 |
| Three.js | Versioned assets immutable | 124 | 95.14 |
| UE Wasm | Default no-cache, no validators | 109,591 | 1506.18 |
| UE Wasm | Versioned assets immutable | 40,838 | 1701.52 |

These rows use unshaped loopback and medians of three reloads. The default-policy control did not persist its in-flight response counter; its transfer column is the ledger total recorded at collection. They show why “warm” needs a definition. The difference between the two server policies is a deployment result for these files, not a built-in advantage or defect of a JavaScript library. The UE warm page still initialized the engine; caching did not make startup free.

Across all 48 measured page loads the fixtures rendered moving pixels at the expected canvas size, and the page-error collector recorded zero JavaScript page exceptions. The runs sampled 5,760 post-ready requestAnimationFrame intervals in total. The raw evidence retains four console resource notices from missing favicon requests. Those notices are not silently relabeled as clean network logs. Animation-frame spacing on this tiny, refresh-limited fixture is not a maximum-throughput benchmark.

## What the memory readings do and do not say

The 11 November runner attempted a Wasm linear-memory sample every 100 ms, but its hook did not obtain a memory handle in any of the 24 UE page loads. Those raw fields are null. They mean not measured, not zero memory usage. The startup and transfer results do not supply a new Wasm-memory measurement.

The separately published 10 November test recorded a sampled peak of 536,870,912 B, or 512 MiB. That historical result belongs to that test. It is a linear-memory allocation, not total browser RAM, not all transient allocations and not GPU VRAM; it is not presented as a newly observed peak for these 24 page loads.

The data includes Chrome's sampled JavaScript heap counters for both stacks. They cover another memory category and are not comparable to the Wasm allocation as a total-memory ranking. No total-memory or VRAM ranking is presented. A larger scene needs its own resource and memory study.

## Which path should a project choose?

For an experience built around a web page, Three.js keeps the scene in the front-end code and allows assets to be chosen and loaded at the feature level. Digital Garden already uses that structure: the website shell is separate from the [playable Three.js garden](https://digitalgarden.si/play/). The [Houdini-to-Three.js garden article](https://digitalgarden.si/blog/webgpu-garden-houdini-threejs/) describes that pipeline. This study's four-box scene is a separate fixture.

For an existing Unreal level, reusing the editor, code and assets may matter more than the initial download. Test the particular features the project relies on in the experimental target, then measure that package on the intended host and client devices. A successful box scene does not validate an arbitrary Blueprint project, large level, deferred rendering path or mobile browser.

For a project that should remain on a server, assess Pixel Streaming as a service architecture. Its browser client receives a stream rather than downloading the Wasm engine package, but a machine still has to run the Unreal application for the session. Streaming quality, network latency, input behavior and service capacity require separate tests. This article supplies no invented prices or session counts.

Three.js's [WebGPURenderer documentation](https://threejs.org/docs/pages/WebGPURenderer.html) describes a WebGL 2 fallback when WebGPU is unavailable. All measured control runs here selected WebGPU; fallback behavior and Safari were not tested. An engine's renderer being called “mobile” does not prove it works on an iPhone. Unity, Babylon.js and PlayCanvas were not measured either.

## Evidence and reproduction

The [desktop summary](https://digitalgarden.si/marketing/research/threejs-vs-unreal-20261111/desktop-summary.json) and [reproduction package](https://digitalgarden.si/marketing/research/threejs-vs-unreal-20261111/threejs-vs-unreal-desktop-reproduction.zip) contain both 24-run datasets, the harness, authored fixture sources, request ledgers, screenshots and the measurement definitions. The package excludes Unreal Engine binaries, engine scripts and the generated UE runtime; reproduce that side using a licensed engine installation and the build steps linked from the earlier article. The portable runner requires explicit paths to the existing Shipping package, Epic server and Python.

The first setup attempt encountered Windows' inactive Python alias before a server or measured trial started. It is retained separately as a setup failure. The completed datasets use the installed engine's Python and report no measured-run failures. This distinction matters when counting the 48 measured page loads.

The [Three.js r186 license](https://github.com/mrdoob/three.js/blob/r186/LICENSE) and [Epic's Unreal Engine terms](https://www.unrealengine.com/en-US/license) remain the authoritative license references. No royalty or seat interpretation is made here.

Author: Alicia, 3D technical artist (TA) and independent developer. Codex assisted with desktop execution, instrumentation, data checks and this manuscript. Public-host HTTPS, iPhone Safari, Pixel Streaming and larger-scene validation remain untested.

## Questions

### Can Unreal Engine run in a browser without Pixel Streaming?

Experimentally, yes. Unreal Engine 5.9 Preview adds a WebAssembly target with WebGPU rendering. Our specific Windows Launcher Shipping fixture rendered locally in Chrome. Public hosting and iPhone Safari remain untested here.

### Are these Three.js and Unreal timings a visual performance ranking?

No. The layout matches but the materials, shaders, runtimes and readiness signals differ. The shared submission hook records a CPU-side WebGPU queue submission, not visible display time or equal rendering work.

### Does a warm Unreal Wasm page still initialize the engine?

Yes in this fixture. Cached resources reduced response-body transfers, but each measured reload still initialized the engine. A warm download is not a zero-cost startup.

### Does Unreal Wasm work in iPhone Safari?

This study has no iPhone Safari result. Desktop Chrome, a mobile renderer name, and local network shaping do not establish iPhone compatibility.
