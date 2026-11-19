# One Houdini plant, 256 samples, and a long WebGPU pause

[Full experiment, figures and runnable lab](https://digitalgarden.si/blog/threejs-webgpu-path-tracing/)

All 12 Chrome runs reached 256 samples per pixel on an RTX 5070 Ti. On this small Houdini scene, the most visible change came from how the path tracer handled compilation.

With `asyncCompile` enabled, the median longest main-thread render call in fresh browser sessions was 83.5 ms. With the default synchronous setting it was 1,724 ms. Both groups used the same 640 × 480 output, worker BVH setup and released three-gpu-pathtracer 0.0.28.

![Three path-traced renders contrast an emissive plane with a rectangular area light at 64 samples](https://digitalgarden.si/marketing/research/webgpu-path-tracing-20261118/emitter-comparison.png)

A small emissive plane left bright speckles at 64 samples. A rectangular area light produced a smoother pool of light. The lighting swap changed the representation and sidedness of the emitter, and power was not normalized.

Alicia authored the plant. Codex assisted with the harness, browser measurements and writing. [See the raw runs and settings](https://digitalgarden.si/blog/threejs-webgpu-path-tracing/).
