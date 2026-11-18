# A lavender glTF can look right and still lose its wind inputs

[Full experiment, screenshots and runnable kit on Digital Garden](https://digitalgarden.si/blog/pixijs-3d-vs-threejs-webgpu/)

A still image looked like a successful renderer migration. All three lavender clumps appeared, and a real click reached the HUD above them. The first sign of trouble came from the loaded geometry.

The same 53,561-triangle GLB went through Three.js 0.186.1 and official @pixi/3d 0.1.0. Three.js retained the eight custom plant attributes. Both Pixi backends warned that each one was skipped. The stem still had position, normal and UV data, enough to render the static picture. A wind shader depending on stalk IDs, anchors and bend weights would need those inputs restored.

![Three lavender clumps from the fixed Pixi WebGPU camera](https://digitalgarden.si/marketing/research/pixi3d-20261117/pixi-gpu.png)

The Chrome test used 15 fresh contexts on one desktop. Across the three routes the median animation callback interval rounded to 16.7 ms. That cadence tells us little about spare GPU capacity. The missing attributes are a more useful first estimate of the porting work.

Alicia authored the garden assets. Codex assisted with the harness, measurements and writing. [Inspect the source data and repeated runs](https://digitalgarden.si/blog/pixijs-3d-vs-threejs-webgpu/).
