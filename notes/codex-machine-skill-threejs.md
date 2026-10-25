# I Tested machine-skill with Codex: Two Three.js Pump Simulations

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A hands-on Codex test of machine-skill: a 4:1 gear pump and rain-barrel irrigation rig, with real screenshots, solver checks, fixes and downloadable demos.

- Author: Alicia Lu
- Published: 2026-10-18
- Updated: 2026-10-22
- Canonical HTML: https://digitalgarden.si/blog/codex-machine-skill-threejs/

**Codex can use machine-skill to build an interactive Three.js machine. My two pump tests also needed corrections before their numbers, cutaways and controls told a consistent story.** On October 18, 2026, I used the skill in a Codex desktop conversation on my Windows workstation, built a hand-cranked gear pump and a rain-barrel irrigation rig, and checked the resulting pages in Chromium.

The useful result is an editable explanation: turn the handle, expose a pump chamber, change the water level and inspect the equations. The result does not establish that either pump would perform this way in hardware. The dimensions and operating constants below describe deliberately constructed teaching fixtures.

I make [digitalgarden.si](https://digitalgarden.si/), an [interactive 3D garden](https://digitalgarden.si/virtual-garden/). This experiment ran separately from the live garden. Its renderer, plants and sound were not replaced.

![The actual Codex gear-pump page, with a brass reduction wheel and a computed 0.153 L/min readout](https://digitalgarden.si/marketing/research/machine-skill/gear-pump.png)

## What I actually ran

The original brief suggested Claude Code. I changed the test host to Codex and read the same machine-skill instructions and reference example. I did not run Claude, compare models, or measure a one-prompt success rate.

| Component | Tested setup |
|---|---|
| Agent workflow | This Codex desktop conversation; explicit reading of the cloned SKILL.md |
| Model identifier | Not independently recorded by this running session |
| Upstream | AirsupHQ/machine-skill, commit e0bd066ac5beaf48512935de2fc0f0e55636853a |
| Three.js | 0.170.0, as pinned by the skill; WebGL2 |
| Browser | Playwright Chromium 156.0.8078.4, headless, Windows |
| Graphics | ANGLE / NVIDIA GeForce RTX 5070 Ti / Direct3D 11 |
| Measurement viewport | 1440 × 900, device scale factor 1 |
| Review | Four rounds per machine: whole, cut, exploded, X-ray, two joint views and film stills |
| Timing and cost | Prompt-to-first-page time, token use and cost were not instrumented |

This was an iterative build in one conversation. The prompts supplied the machines and questions; Codex then authored their geometry and solvers, ran checks and repaired the output. A multi-stage build should not be described as an untouched one-line result.

## Using this Claude-oriented skill with Codex

The upstream README explicitly includes Codex among hosts that can read SKILL.md.[^1] I cloned that repository into an isolated experiment folder and had Codex read the file directly. I did not test automatic discovery, a slash command or a global installation.

For a project-scoped setup, OpenAI documents .agents/skills as a Codex skill location.[^2] A reproducible placement is:

```powershell
git clone https://github.com/AirsupHQ/machine-skill.git .agents/skills/machine
git -C .agents/skills/machine checkout e0bd066ac5beaf48512935de2fc0f0e55636853a
```

Then explicitly ask Codex to read that SKILL.md and build the machine. Claude's /reload-skills instruction is not part of this tested Codex workflow.

The page contains the kit and machine code inline, but imports Three.js and fonts over the network. A single HTML file is convenient to move; it is not a fully offline package. These are purpose-written mathematical models and kinematic animations, without a general contact solver.[^1]

## Test 1: a hand-cranked gear pump

The task I used was:

```text
Use machine-skill with Codex: a hand-cranked gear pump with a 4:1 spur gear reduction, slow it down so I can see the teeth mesh, and say what breaks if I crank it dry.
```

The input pinion has 20 teeth and the reduction wheel 80. At module 0.005 m, their pitch radii are 0.05 m and 0.20 m, giving a 0.25 m centre distance. The two pump gears have 16 teeth each. The display slows rotation by 1:8 while leaving the reported rpm and flow on the solver's time scale.

At 60 hand-crank rpm, the model produces 15 pump rpm and **0.153 L/min**. That value follows the declared displacement of 12 mL/revolution and assumed volumetric efficiency of 0.85:

```text
pump rpm = crank rpm / 4
flow (L/min) = 12 × pump rpm × 0.85 / 1000
```

I checked inputs of 0, 30, 60 and 120 rpm, the actual animation's inverse rotation ratio and phase relationship, and the modelled centre distance. Those checks passed. The gear helper itself describes its tooth profile as involute-like: seeing teeth pass one another is not proof of a manufactured involute pair or solved tooth contact.

The first dry-running model had a more revealing error. Its friction and cooling assumptions put the equilibrium temperature below the chosen 85 °C failure threshold, even at maximum crank speed. The failure could never occur. The corrected example declares a smaller illustrative seal thermal mass of 0.5 J/K and cooling of 0.02 W/K, alongside 1.6 N m friction. Removing water now gives zero water flow and a computed temperature that crosses the threshold during the simulated run.

Those constants were selected to demonstrate the thermal equation. The 85 °C threshold is an assumption, not a manufacturer's seal rating, and the test does not predict a real pump's time to damage.

## Test 2: rain-barrel drip irrigation

The second task was:

```text
Use machine-skill with Codex: a garden drip irrigation rig fed by a small centrifugal pump from a rain barrel; show flow at each emitter and what happens when the barrel runs low.
```

![The actual irrigation test in cutaway mode, showing the water barrel, pump and four labelled outlets](https://digitalgarden.si/marketing/research/machine-skill/irrigation-cut.png)

This fixture uses a centrifugal pump curve intersected with a system curve. The system includes lift from the barrel, a lumped pipe resistance and four equal emitters. At full speed and a 0.50 m water level, the result is approximately **0.276 L/min total, or 4.14 L/h at each emitter**.

```text
H_pump = 3 s² - 0.08 Q²
H_system = 0.75 - h + 0.1 Q² + 10.197 (60 Q / 32)²
```

Here H and h are metres, Q is L/min and s is speed divided by rated speed. Each emitter is assumed to deliver 8 L/h at 1 bar, with square-root dependence on pressure. The constants describe this fixture; they were not fitted to a measured pump curve.

The automated checks compare the two heads at the operating point and verify that four branch flows sum to the total. At water levels of 4 cm or below, the suction input is considered uncovered and calculated water flow becomes zero. Just above that threshold, flow returns. The **Expose the intake** button sets the water-level input to 2 cm; it does not bypass the solver and force a decorative failure animation. Reset restores the water and temperature state.

Equal pressure at all four emitters is a simplification. This model does not resolve pressure variations along individual hoses, air entrainment, priming or cavitation. Shaft motion is displayed at 1:80.

## What needed fixing

| Area | Gear pump | Irrigation rig |
|---|---|---|
| Physics | Initial thermal assumptions made the failure unreachable; corrected and disclosed | Pump/system intersection and water cutoff passed; branch pressure is assumed equal |
| Geometry and scale | Added an actual cavity around the pump gears; checked 0.25 m gear centres | Replaced a solid casing with a shell around the impeller; metres are explicit |
| Materials and cutaway | Shared X-ray materials initially hid the base | Same shared-material problem; isolated shell materials while retaining clipping and cap callbacks |
| Performance and layout | Reduced detail without deleting parts; repaired narrow-screen framing | Reduced detail without deleting parts; moved controls off the model on narrow screens |

The follow-up work had three goals: audit the equations and failure thresholds; isolate X-ray materials and inspect solid sections; then halve the geometry budget while keeping every registered part and repair the mobile framing. The review also caught an easy-to-miss implementation detail: cloning a material alone did not preserve the kit's custom shader callback and shared clipping planes. Both needed to remain attached.

The original studio palette is still fairly clean. It reads as a mechanical explanation rather than a weathered prop from the garden. Weathering, detailed valves and seal geometry would be further art and engineering work.

## Measured performance

The upstream screenshot script initially reported about 2 FPS on this Windows setup. A local change to its browser launcher, adding --use-angle=d3d11, restored hardware rendering and roughly 60 FPS. This is an environment correction, not a claimed benefit of reducing triangles.

| Page | Standard triangles | Lightweight triangles | Kit-reported parts |
|---|---:|---:|---:|
| Gear pump | 60,100 | 18,628 | 25 in both versions |
| Irrigation | 37,560 | 7,908 | 33 in both versions |

The lightweight variant reduces curve segments, tooth-flank samples and bevel detail. It retains the tooth counts and the registered part list. Three one-second animation-frame samples per standard page stayed around 60 FPS at the stated viewport, with no page exceptions in those checks. These short, warm desktop samples do not establish sustained phone performance, startup speed or a rendering limit.

The original narrow layout enlarged the model behind its text. I separated the canvas from the control panel and checked 390 px and 320 px desktop-emulated viewports. That is responsive testing, not a physical-phone acceptance result.

## Download, recording and embedding

The [demo ZIP](https://digitalgarden.si/marketing/research/machine-skill/codex-pump-demos.zip) contains both HTML pages, the generator, reproduction notes and the upstream licence. Extract it and open either page in a WebGL2 browser with network access. When served over localhost, add ?low=1 to select the lightweight version. The [test results JSON](https://digitalgarden.si/marketing/research/machine-skill/results-2026-10-18.json) preserves the geometry, solver, mode and responsive checks.

I also tested the upstream recording command locally with a Windows launcher adaptation. Both delivered clips are 20 seconds, 960 × 600, H.264 at 15 fps, without music. They use seconds 8–28 of the generated film, beginning with the assembled machine; the full 28-second takes were retained locally. Full decode and the recorder's pink-frame scan passed. These are local delivery files, separate from the desktop FPS measurements; their sizes and hashes are in the result data.

A plain same-origin local iframe loaded the irrigation page in a 390 px viewport. The site's current Content Security Policy, however, explicitly blocks frames. A local copy of that policy blocked the same embed as expected; the production policy also restricts the external script and font sources used by these pages. **I have not embedded this renderer in the production garden or changed that policy.** The downloadable example and screenshots are the published results.

Merging a generated machine into the garden would require deliberate scene, material and solver integration. For comparison, my [Houdini-to-Three.js OpenUSD test](https://digitalgarden.si/blog/openusd-threejs-usd-web-viewer/) deals with importing authored assets. The earlier [Claude Code for 3D workflow](https://digitalgarden.si/blog/claude-code-for-3d/) describes another agent setup; it is not the host used here. A later [paired Claude Code and Codex task](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/) used one frozen plant-wind starter and checked both untouched outputs on WebGPU and the WebGL2 fallback.

## Would I use it again?

Yes, for an inspectable mechanism study or an article that needs moving parts and equations readers can examine. The kit gave the two tests a useful common structure. The more valuable part of the exercise was catching mistakes that a good-looking screenshot concealed: an unreachable failure, a solid block where a pump chamber should be, and transparency shared with unrelated parts.

For a production garden prop, I would still author and review the asset for that scene. For engineering analysis, I would start from measured dimensions and validated component data. This test establishes a working Codex-assisted explanation workflow, with its assumptions left visible.

The next [img2interior test with Codex](https://digitalgarden.si/blog/img2interior-codex-threejs-garden/) applies that review loop to a Blender-authored room: three output contracts, a 95 MB runtime payload, actual cloth/hem checks and a viewer lifecycle repair.

[^1]: AirsupHQ, machine-skill README and source at the tested commit, including the Codex host note, template, geometry helpers and solver recipes. https://github.com/AirsupHQ/machine-skill/tree/e0bd066ac5beaf48512935de2fc0f0e55636853a
[^2]: OpenAI, Agent Skills documentation, skill invocation and repository skill locations. https://developers.openai.com/codex/skills/
