# Google Gemini 4 Argon for Three.js: What Community Tests Actually Show

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Google's Gemini 4 Argon AI model for Three.js: public community tests, the Blender MCP asset route, recurring failures, and what still needs human work to ship.

- Author: Alicia Lu
- Published: 2026-10-14
- Updated: 2026-10-20
- Canonical HTML: https://digitalgarden.si/blog/gemini-4-argon-threejs/

Gemini 4 Argon is an AI model in Google's Gemini family, [released on October 13, 2026](https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-4-argon/). This article collects public community tests of the model generating Three.js scenes, including workflows that drive Blender MCP. It is written for [digitalgarden.si](https://digitalgarden.si/), an [interactive 3D garden](https://digitalgarden.si/virtual-garden/) built with Three.js.

First, the ground rules. I didn't test Argon myself. This post collects tests other people published. Every case below names the author and links to the original post, and I only describe what each person actually showed. If a post didn't mention frame rate or failures, I don't guess.

Second, some context. digitalgarden.si is an interactive 3D garden built with Three.js. I didn't build it with Argon. But because I work in Three.js every day, my question is narrow: how far is an AI-generated Three.js scene from something you could ship?

## What the official post says

I'm only using Google's own announcement here,[^1] and only the parts that touch 3D.

- Google says Argon improves on earlier Gemini models in spatial reasoning, mathematical precision, and "visual coding."
- Googlers use Argon inside Antigravity, with slash commands like `/goal` and multi-agent loops with custom skills, to prototype 3D visualizations, games, and interactive animations. The output is vanilla code that renders directly in the browser.
- One demo, Desk Racer, turned pictures of a real desk into a playable miniature racing game. Blender MCP modeled and collision-checked the desk objects, room, and track. Nano Banana made the textures. Three.js handled the physics and custom shaders, with car engine audio from Gemini Omni Flash.
- Another demo built a pagoda temple complex in Blender, using multi-agent visual critique loops.
- In a third, Argon wrote Python scripts to procedurally sculpt, rig, animate, and PBR-bake a 3D game, then wired it for the web in Three.js with GLSL shaders and shatter physics.
- A Gothic scene started from a single Nano Banana image, with Argon driving Blender MCP in an autonomous loop.

Note the word Google uses: prototype. That's their framing, not mine. And these are Google's own showcase pieces, not independent tests.

## The community cases

### Case 1: Margarita slosh, four models side by side (@ann_nnng)

**Task.** Ann Nguyen (@ann_nnng) tested the same effect, liquid sloshing in a margarita glass, with Argon, Muse Wonder, Claude Opus 5.5, and GPT 6 Astra.[^3]

**Result.** She reports that all four used Three.js with WebGL. Her verdict was one line: Gemini has "improved so much."

**Failure points.** None stated. It's one person's visual comparison. The post gives no frame rate, no code, and no detail on how many runs each model got. It's a useful vibe check, not a benchmark.

### Case 2: A fluffy pumpkin with no Blender (@thebuggeddev)

**Task.** The Bugged Dev (@thebuggeddev) had Argon make a fluffy pumpkin plushie without using Blender at all.[^4]

**Result.** By his description, it's a procedural GLB model with volumetric fur shells, GPU-instanced fur strands, and real-time air-jet physics, built with Three.js and GLSL. You can blow the fur around in real time. In a reply, he explained that Argon generated the meshes mathematically and packed them directly into binary glTF 2.0 with Python.[^5]

**Failure points.** He didn't mention any.

### Case 3: A turbofan engine and a Rubik's cube robot (@ai_for_success)

**Task.** Ashutosh Shrivastava (@ai_for_success) posted two builds. In one, he asked Argon for a high-bypass turbofan jet engine using Blender and Three.js.[^6] In the other, Argon in Antigravity built a whole project: robot arms modeled in Blender, then a 3D simulation in Three.js. He says Gemini then solved the cube through the Interactions API.[^7]

**Result.** Two videos, both following the same route: model in Blender, show it in Three.js.

**Failure points.** Neither post mentions any.

### Case 4: A flying dragon (@ashen_one)

**Task.** @ashen_one made a flying dragon test and said everything was custom-made, with no premade animations and no video generation.[^8]

**Result.** The mesh came from Tripo 3D. He's specific that he drove Tripo through browser control, not MCP. Then Blender MCP and Argon took it from there. His own take: "not bad for one shot?"

**Failure points.** None stated, but note what this shows: Argon didn't make the model. The mesh and the motion came from different tools.

### Case 5: A storm lighthouse in one HTML file (@YFarmX)

**Task.** @YFarmX had Argon build a storm-lashed lighthouse in a single HTML file.[^9]

**Result.** Real-time Three.js with no image files. The island, waves, and lightning are all generated in code. By the author's own count, the whole chat used about 87k tokens.

**Failure points.** None mentioned in the post.

### Case 6: A sneaker from primitives (Creative AI News)

**Task.** The Creative AI News staff gave Argon and Gemini 3.1 Pro the same prompt, once each: a single web page with a 3D sneaker on a round pedestal, soft studio light, slow rotation, and drag to look around. No 3D files allowed, so the shoe had to be built from simple shapes in Three.js.[^10]

**Result.** Argon produced a rounded sole, a tongue, laces, and a heel loop on an orange-trimmed pedestal. It also added a product title nobody asked for. Both pages opened without errors.

**Failure points.** The toe is boxy and the spin is very slow. The writers said themselves that neither shoe "would pass for a real product render," and called it a first look, not a benchmark.

## Recurring problems

Six cases is a small set, but a few patterns show up.

1. **Failures are mostly unreported.** Of the six, only the one written up by a publication, Creative AI News, lists concrete flaws.[^10] The X posts show the result and stop. That doesn't mean nothing went wrong. It means you can't tell from the post.
2. **Close isn't finished.** The sneaker test is the clearest look at quality: recognizable, but boxy in places and not product-render grade.[^10]
3. **Argon didn't always make the assets.** The dragon mesh came from Tripo.[^8] Before you judge a demo, work out which step the model actually did.
4. **The samples are tiny.** Every test here is a single post or a single run per model. Nobody published repeat runs.
5. **Setups are hard to reproduce.** XiaoHu wrote a step-by-step guide to Google's Desk Racer demo and found that the headless Blender MCP configuration used in that demo wasn't published. The guide uses the community mcp-for-blender add-on instead.[^2] If you try to repeat a demo, expect your setup to differ.

## The Blender MCP to Three.js asset route

Two routes come up again and again.

- **Route A: through Blender.** Argon runs Python through Blender MCP, models in Blender, exports GLB, and Three.js loads it. That's how Google describes Desk Racer,[^1] and the XiaoHu guide spells out the same split: Blender MCP models through Python scripts and exports GLB props, and Three.js loads, renders, and handles driving physics.[^2] Cases 3 and 4 also go through Blender.[^6][^7][^8]
- **Route B: skip Blender.** Argon writes Python that generates glTF directly,[^5] or builds everything in code inside one HTML file.[^9][^10]

Here's how I'd choose. If the asset needs to stay editable, or another artist will pick it up later, use Route A. You get a .blend file you can open and fix. If it's a one-off effect or an atmosphere piece, Route B is fine. There's no file to hand off, but there's also less to set up.

**Checklist before you start:**

- [ ] Which Blender MCP implementation, and which port? Google's demo setup wasn't published; the XiaoHu guide's add-on defaults to local port 9876.[^2]
- [ ] Do the exported GLB's units, orientation, and up-axis match your Three.js scene?
- [ ] Are colliders built separately, or are you colliding against the render mesh?
- [ ] Where do textures come from: Nano Banana, procedural, or hand-painted?

## Performance and accessibility: still human work

This part is my own checklist from Three.js projects. It doesn't come from the tests above. None of the cases reported mobile performance or accessibility, so treat these as open items for any AI-generated scene.

**Performance:**

- [ ] Test the frame rate on mid- and low-end phones, not just the author's desktop recording.
- [ ] Count draw calls. Merge meshes where you can and use instancing for repeated objects.
- [ ] Compress textures (KTX2) and geometry (Draco or Meshopt).
- [ ] Dispose geometry, materials, and textures when the page is left or the scene unloads.
- [ ] Make post-processing like bloom something you can turn off.

**Accessibility:**

- [ ] Respect `prefers-reduced-motion`: reduce or stop automatic animation.
- [ ] Support keyboard controls, not just mouse dragging.
- [ ] Give the canvas a text description, and don't put key information only inside the 3D view.
- [ ] Limit how often things flash. The lightning in a storm scene is exactly where this matters.

## How it splits with a Claude Code workflow

I wrote earlier about [Claude Code for 3D artists and a real Houdini pipeline setup](https://digitalgarden.si/blog/claude-code-for-3d/). Looking at these community cases, here's how I'd split the work for now:

- **Argon:** go from a photo, sketch, or one-line idea to a playable prototype fast, to check whether an idea works. Use it to run Blender MCP for a first pass on assets.
- **Claude Code:** refactoring inside an existing codebase, pipeline scripts, performance work, and anything that needs long-term maintenance.
- **A person:** lighting, materials, how interaction feels, the performance budget, and accessibility.

Image-to-mesh is a separate front end from either of those tools. A [tested ComfyUI prop workflow](https://digitalgarden.si/blog/comfyui-3d-workflow/) still had to remesh, decimate and reopen the GLB in Blender and Houdini before the generated draft counted as a prop. A downloaded [Meshy 6 Lite model](https://digitalgarden.si/blog/meshy-3d-agent-houdini-cleanup/) needed the same kind of Houdini cleanup before Three.js instanced it.

This split is a suggestion based on public material. It is not a head-to-head test of the two tools. A separate [machine-skill with Codex](https://digitalgarden.si/blog/codex-machine-skill-threejs/) test built two Three.js pump simulations; Codex was the host, not Argon, and those pages still needed human corrections. The [Claude Code versus Codex wind comparison](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/) is a paired run of those two coding tools on one Three.js plant, with WebGPU and WebGL2 checked independently.

## Verdict: prototype yes, production not yet

From the public tests, Argon's strength in Three.js is getting from nothing to something you can see and play with, fast. Google itself frames it as a prototyping tool.[^1] The one test that listed flaws found a scene that looked close but not finished,[^10] and none of the posts covered performance or accessibility. Those are still on you.

So I'd use it to sketch, not to ship. Take the prototype, then do the boring parts yourself.

If you want a feel for what a browser scene built with Three.js can be, walk around the [virtual garden](https://digitalgarden.si/virtual-garden/) or try [Play](https://digitalgarden.si/play/). Neither was built with Argon.

## References

[^1]: Google, "Gemini 4 Argon: Built to solve your hardest problems" — https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-4-argon/
[^2]: XiaoHu, Desk Racer reproduction guide — https://best.xiaohu.ai/en/article/desk-racer-photo-to-game/
[^3]: @ann_nnng, margarita slosh four-model comparison — https://x.com/ann_nnng/status/2110234900616126769
[^4]: @thebuggeddev, fluffy pumpkin — https://x.com/thebuggeddev/status/2110043093466743194
[^5]: @thebuggeddev, how the GLB was generated — https://x.com/thebuggeddev/status/2110205444836950118
[^6]: @ai_for_success, turbofan engine — https://x.com/ai_for_success/status/2110040065800097793
[^7]: @ai_for_success, Rubik's cube robot arms — https://x.com/ai_for_success/status/2110040948638126557
[^8]: @ashen_one, flying dragon — https://x.com/ashen_one/status/2110192210964521388
[^9]: @YFarmX, storm lighthouse — https://x.com/YFarmX/status/2110093177088885008
[^10]: Creative AI News, sneaker scene test — https://www.creativeainews.com/articles/gemini-4-argon-launch-price-3d-test-2026/
