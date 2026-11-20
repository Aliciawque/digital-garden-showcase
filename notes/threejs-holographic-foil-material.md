# What Changes When a Houdini Foil Plaque Tilts?

[Read the full experiment and download the runnable lab](https://digitalgarden.si/blog/threejs-holographic-foil-material/)

I used a 288-triangle Houdini plaque to see how foil changes with angle. Prism went from bright silver and blue at 0° to a dark face with a bright rim at 60°. Nebula kept a speckled blue look. A separate scanline shader on the same plaque kept its cyan stripes.

![One Houdini plaque at three angles with Prism, Nebula and a cyan scanline shader](https://digitalgarden.si/marketing/research/holographic-foil-20261119/foil-tilt.png)

On an RTX 5070 Ti, I ran Prism and Nebula at each of three WebGL 2 quality tiers. Across the 36 runs, first-page readiness medians ranged from 2.6–3.4 seconds for Prism and 7.2–12.3 seconds for Nebula. A second page in the same browser context was ready in about 0.15–0.18 seconds. The timings started after the GLB loaded.

Chrome initialized an actual WebGPU backend in a separate test. The pinned Holospot constructor then rejected Three.js WebGPURenderer. The foil trial used Three.js r187 and WebGL 2, outside the garden renderer.

Codex assisted with the fixture, browser measurements, and writing. [See the captures, timing data, and GLB](https://digitalgarden.si/blog/threejs-holographic-foil-material/).
