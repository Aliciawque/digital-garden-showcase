# FLIP bubble experiment: measured notes and source

Read the [complete experiment, code and evidence on Digital Garden](https://digitalgarden.si/blog/houdini-flip-deforming-bubbles/). This is a short companion note; the original article contains the full method and measurements.

A bubble-shaped particle selection seemed like a manageable exercise after studying Entagma's viscous-fluid tutorial. I built a separate 48-frame pour in Houdini 22.0.429 and compared five ways to carry its distance attribute through the cache.

Three checks changed the implementation:

- On frames with no new centers, my distance-node configuration returned -1. Without a guard, the next threshold test selected the whole fluid.
- Stable IDs preserved identity when point order changed. Neighbor repair needed its own second Wrangle to read the completed inheritance result.
- Surfaced cavity shapes needed clipping against an inset liquid SDF. The containment checks covered 53 frame/radius outputs.

![Native FLIP cavity meshes shown as an explanatory cutaway](https://digitalgarden.si/marketing/research/flip-bubbles-20261113/cutaway-sequence.png)

The limits are useful too. This is cached-material post-processing, not a second air-phase simulation. The larger-radius final mesh retained two local face-direction exceptions; the tests do not establish production-ready glass rendering or continuous collision freedom. The explicit CUDA probe fell back to CPU, so formal neural surfacing used DirectML.

The original write-up includes all five comparisons, the two-pass VEX, discrete mesh checks, an actual Solver SOP comparison, figures, CSV and a small source package. Codex executed the Houdini tests and assisted with numerical checks and writing.

[Continue with the full Houdini FLIP bubble study on the original website](https://digitalgarden.si/blog/houdini-flip-deforming-bubbles/).
