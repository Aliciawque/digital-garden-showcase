# Switching Two Baked Lamps in Three.js Without Reloading Textures

[Full experiment, raw measurements and runnable lab](https://digitalgarden.si/blog/threejs-lightmap-lamp-toggle/) · [中文原文](https://digitalgarden.si/zh/blog/threejs-lightmap-lamp-toggle/)

I turned off the amber lamp on a Houdini garden bench and left the blue pool of light intact. Blender Cycles baked the overhead fill, amber and blue contributions onto the same second UV atlas. A Three.js shader combined the three linear textures. The buttons changed their weights while the bench and path stayed in one draw call.

![The Houdini bench with both baked lamps on, each lamp off in turn, and the independent single-map control](https://digitalgarden.si/marketing/research/lightmap-lamp-20261121/lamp-comparison.png)

The fixture had 116 triangles and 58 non-overlapping lightmap charts. The three layer PNGs totaled 72,528 B. The independent all-on bake was 83,004 B. The layered version made two extra cold-load resource requests. When I pre-summed the same three layers, the displayed result differed by only 0.00045 RGB code values per channel on average.

On this desktop setup (Chrome, WebGL 2, Three.js r187 and an RTX 5070 Ti), 24 switches took a median 13.9 ms from handler entry to return of the next render call. That includes the wait for the next browser callback. The browser test did not measure GPU execution or display latency.

Codex assisted with the fixture, tests and writing. [See the bake settings, controls and 33-file lab](https://digitalgarden.si/blog/threejs-lightmap-lamp-toggle/).
