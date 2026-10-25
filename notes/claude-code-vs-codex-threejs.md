# Claude Code vs Codex: One Reproducible Three.js Wind Task

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A paired Claude Code and Codex workflow test: the same Three.js plant fixture, wind and color-space tasks, unchanged outputs and independent browser checks.

- Author: Alicia Lu
- Published: 2026-10-20
- Updated: 2026-10-21
- Canonical HTML: https://digitalgarden.si/blog/claude-code-vs-codex-threejs/

Claude Code and Codex both completed this small Three.js task and produced builds that passed. Independent browser checks then loaded both unchanged outputs on WebGPU and WebGL2 fallback. This is one paired workflow case study, not evidence that either tool is generally better at 3D work.

For the renderer and asset tests this wind task builds on, see [the garden's Three.js rendering overview](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

The [production garden](https://digitalgarden.si/play/) already has wind. I used a separate frozen starter so that this test would not pretend to add a feature that was already present, expose the private application, or depend on unrelated project history. Codex operated the test and subsequent acceptance checks on 20 October 2026.

![Claude Code's unchanged output, running in the independent browser acceptance wrapper](https://digitalgarden.si/marketing/research/comparisons-20261020/agent-claude.jpg)

## The same starter and request

Both tools received the same source files, fixture hashes, pinned Three.js 0.186.1 and Vite 8.3.1 dependencies, AGENTS.md and task prompt. CLAUDE.md contained an AGENTS.md reference. Each started a new noninteractive session with an isolated copy of the frozen repository.

Task A was a bounded sinusoidal X bend driven by the exported _stiff and _phase vertex attributes. Roots with stiffness zero had to stay fixed; maximum displacement at wind=1 had to stay within 0.10 units. The existing Wind slider had to update a GPU uniform, with one InstancedMesh per source primitive and no per-frame matrix rewrites. Task B was a deliberately incorrect albedo texture colour space: fix the colour texture without treating geometry data as sRGB.

The fixture contains 16 instances of an owned 6,056-triangle procedural plant. Its matrices are translations, not arbitrary scaling or rotated parents. The root/bound argument in this article therefore applies to that controlled transform setup. It does not validate a general production wind solver.

## Which tools actually ran

| Tool | CLI | Model recorded/requested | Execution environment |
| --- | --- | --- | --- |
| Claude Code | 2.1.303 | claude-opus-5-5, reported in the run | Existing authenticated first-party Claude account on a Linux VPS, reached over Tailscale SSH |
| Codex | 0.144.1 | gpt-5.6-sol, successful explicit CLI selection | Existing ChatGPT login on the local Windows workstation |

The first Codex invocation used the desktop's gpt-6.1-sol default and was rejected before editing because that model was unsupported by this CLI account route. I retained the failure and used a model listed by the CLI's local catalogue. This was a setup correction, not an extra coding attempt on a failed solution. The successful task used the original unchanged starter.[^1]

The environments and permission profiles differ. Claude used the VPS's existing configuration and encountered command permission denials while inspecting the GLB. It worked around them with shell utilities, and its report disclosed an inspection script written in a temporary directory outside the intended fixture. That is a scope failure worth retaining: a written instruction is not filesystem confinement. Codex used its Windows workspace-write mode. I did not normalize these into an artificial security or speed score.[^2]

## What the patches changed

Claude added a sine expression with clamped wind and stiffness, a slider input listener, and the sRGB albedo setting. Codex used the same basic bounded multiplication but relied on the verified fixture's stiffness range and the slider's 0–1 range. Both used phase as radians. The prompt left frequency open, so their selected frequencies differ slightly.

For the actual input domain, sin is bounded by one and both wind/stiffness factors lie between zero and one. Multiplying by 0.10 establishes the maximum X offset; multiplying by zero establishes the fixed root. This is source and data reasoning, not GPU vertex readback. Only Claude's additional clamps defend against a later out-of-range input; Codex's code needs its stated input contract to remain true.

Both outputs changed the albedo to THREE.SRGBColorSpace. The fixture assets, camera, placements, dependency versions and protected instruction files remained unchanged. The [paired-task download](https://digitalgarden.si/marketing/research/comparisons-20261020/agents-tests.zip) includes the exact prompt, starter hashes and unmodified source outputs. The public package excludes authentication data, private configuration and raw model transcripts.

## Independent browser acceptance

Neither tool claimed to have verified the browser. Both ran npm run build and disclosed that visual testing remained outstanding. I then appended an independent QA wrapper to copies of their unchanged source. The wrapper displays the active backend, uniform value, instance count and texture colour space, and records three 10-second static-camera observations. Its additions are separated from the saved agent patches.

The browser checks used a 960 × 640 canvas at DPR 1. Both outputs loaded on both backends, retained one InstancedMesh with 16 instances, and reported the albedo as sRGB. Moving the slider to 0 and 1 updated the displayed shared uniform; the rendered plants stopped and swayed as expected. Screenshots and DOM receipts are retained. This verifies the fixture's interactive behavior, not production normals, physically preserved stem length or every device.

The observations stayed near the display ceiling, so no FPS winner is inferred. Wall time and token telemetry are retained in the compact evidence, but the accounts, model families, OS, caching and permission delays differ. Subscription telemetry is not an invoice and is not used to estimate comparative cash cost.

## What I would carry into a real garden change

The useful common behavior was editing a small explicit task, preserving the asset contract, building, and stating validation limits. The additional operator step was essential: a passing build did not verify WebGPU compilation, the fallback, visible motion or the UI.

For a real change, freeze the task and inputs, record the actual model and execution route, review the diff, and independently test each required backend. This paired run does not test Houdini automation, a large migration, long-term maintenance or general model intelligence. Continue with the [full plant submission matrix](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/) or the [Houdini-oriented Claude Code setup](https://digitalgarden.si/blog/claude-code-for-3d/).

[^1]: OpenAI, [Codex noninteractive execution](https://learn.chatgpt.com/docs/developer-commands#codex-exec), checked 20 October 2026. Availability was verified by the actual account response, not inferred from a desktop model label.
[^2]: Anthropic, [Claude Code CLI reference](https://code.claude.com/docs/en/cli-reference), checked 20 October 2026. Permission and print-mode settings describe configuration; they do not substitute for inspecting the recorded actions.
