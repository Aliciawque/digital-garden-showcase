# Cluster LOD on a dense browser prop: three checks

Read the [complete desktop study, raw measurements and reproducible source on Digital Garden](https://digitalgarden.si/blog/threejs-meshlets-cluster-lod/). This is a short companion; the original contains the full method and results.

A new, deterministic Houdini rock supplied 1,046,528 triangles for an isolated Three.js r187 experiment. Codex tested the official WebGPU MeshletDrawingPipeline, the independent three-nanite cluster hierarchy, manual mesh LOD and single-view cards on one RTX 5070 Ti desktop.

Three integration checks mattered more than a headline FPS figure:

- A requested 300,000-triangle budget selected 300,500 triangles at the middle camera. It was not a strict cap.
- Deliberately disabling WEBGL_multi_draw still produced an image, but one middle-view rock needed 107 calls rather than one batched call.
- The independent preprocessor retained the first material and position/normal/UV data in an authored negative fixture, while dropping color and wind attributes.

![Actual diagnostic normal-shaded rendering of 100 copies of the owned Houdini rock](https://digitalgarden.si/marketing/research/meshlets-20261116/official-near.png)

The experiment covers 42 static configurations, three recorded runs per configuration and separate GPU probes. CPU submission intervals, GPU slices and displayed FPS stay distinct. The routes also use different approximation quality, so their timings are not an equal-quality ranking. Flat single-view cards visibly changed the result.

This was a static-rock desktop test. Production foliage, wind, PBR, shadows and phones remain untested; the garden's production renderer was not replaced. Codex generated the fixture and harness, ran the measurements and assisted with writing at Alicia's direction.

[Continue with the original Meshlets and cluster-LOD study](https://digitalgarden.si/blog/threejs-meshlets-cluster-lod/).
