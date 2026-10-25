# Claude Code for 3D Artists: A Real Houdini Pipeline Setup

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> Claude Code for 3D artists: a Houdini setup with a CLAUDE.md template, a hython fix loop, MCP risks, and a human check list before you trust output.

- Author: Alicia Lu
- Published: 2026-10-14
- Updated: 2026-10-20
- Canonical HTML: https://digitalgarden.si/blog/claude-code-for-3d/

When I'm building [digitalgarden.si](https://digitalgarden.si/), an [interactive 3D garden](https://digitalgarden.si/virtual-garden/), the slow part is rarely sculpting one more flower. Claude Code is what I use on the Houdini side, with hython in the terminal: exports come out with the wrong up-axis, USD layer names don't match, and a batch of assets all need the same attribute changed.

That's where Claude Code for 3D work earns its place. It reads and writes the scripts in your shot folder, runs `hython` in the terminal, reads the error, fixes the script, and runs it again.

People in the English-speaking Claude Code crowd already make this point: if you treat it like a normal coding chatbot, you're wasting it. The difference comes from the system you build around it: persistent conventions, commands it can run, and output you can check.[^1][^12]

It won't click nodes in your viewport for you, and it won't deliver finished shots on its own. You still decide whether the shot looks good. The agent writes the repetitive glue code, gets it running, and fixes it until it passes a minimum check list.[^1][^2]

## A script partner, not a viewport clicker

Compared with asking a question in a browser chat and pasting the answer into a Wrangle, there are three main differences.

1. **It can see your repo.** Your existing exporters, naming rules, and past mistakes become context. It doesn't have to guess from zero every time.[^1]
2. **It can run code and read the errors back.** The script goes through the terminal, the traceback comes back, and it gets fixed. You don't just get a block of code that looks right.[^1]
3. **`CLAUDE.md` pins your conventions to every session.** Write the axis, units, and Houdini version down once, and you get fewer Maya-flavored `hou` calls.[^1][^3]

Houdini users on Reddit are fairly practical about this. With docs for the right version, Claude often writes usable VEX and Python. It still hallucinates, so you need to be able to check node and parameter names. One user tried a Houdini MCP over a weekend and found that most of the time it was faster to ask, then do the work themselves. Their studio's security policy wouldn't allow it anyway.[^4]

For most artists, I'd suggest this order: get the no-MCP, terminal-only setup working first. Then decide whether you want live session control at all.

The agent can't move your mouse. What you can reliably hand off is text: VEX drafts, `hou` scripts, `hython` / `hbatch` / `husk` commands, and written node recipes you wire up yourself.[^2]

In one line: you're the director and the reviewer. Claude Code is a TD assistant that edits scripts, runs commands, and reads logs. It is not an auto-clicker.

## What goes in the `CLAUDE.md` at your repo root

Anthropic's docs describe `CLAUDE.md` as persistent project instructions the agent reads at the start of every session. Specific and short works better than vague lines like "write high-quality code." It's context, not a sandbox. If a path really must be off-limits, you still need permissions, hooks, or similar mechanisms.[^3]

The Retrodeck guide makes a related point. Put your Blender or Houdini version, coordinate system (Y-up or Z-up), units, asset naming, and entry points in the repo root. That's what stops basic failures like Maya-flavored code or assuming centimeters by default.[^1]

Here's a **TEMPLATE**. Change the versions and paths for your own machine. It is not a log from a real run.

```markdown
# CLAUDE.md — Houdini shot / asset pipeline (TEMPLATE)

## Stack (pin versions; don't let the model guess)
- Houdini: 20.5.x (or your actual version)
- Python: the hython that ships with that version
- USD / Solaris: does this project require a layer naming prefix?
- Downstream: Unreal / real-time preview (if any)

## Axis and units (prevents wrong-axis exports)
- Scene up-axis: Y-up (or Z-up — pick one and write it down)
- Units: meters (or centimeters — pick one and write it down)
- Export to Unreal: state whether an extra rotation / scale convention applies
- Never: assume Maya-style transforms by default; if unsure, ask first

## Naming
- Shots: `shots/SH####/`
- Caches: `cache/<asset>/<version>/`
- Suggested USD asset prim: `/World/Assets/<Name>`
- Never ship meaningless names like `geo1` / `null2`

## What the agent may / may not do
- May: edit scripts under `pipeline/` and `shots/*/scripts/`; run `hython` smoke tests on a single asset
- May not: delete caches, touch secrets, submit to the farm without confirmation, edit a `.hip` that has no backup

## How to run
- Smoke test: `hython path/to/script.py`
- One asset first, then the batch
- After every change: print axis check / bbox / whether all material slot names are present

## Minimum human checks
- Do axis and scale look right at a glance?
- Are any materials or attributes missing?
- Was an error swallowed and reported as a fake success?
```

Make each line something you can verify. "Print the world transform before export" beats "be careful with coordinates." The docs also suggest moving multi-step procedures into skills or path-specific rules, so `CLAUDE.md` doesn't grow into an encyclopedia.[^3]

## The Claude Code Houdini loop: write, run hython, read the error, fix

Without MCP, the terminal already covers most of the pipeline glue:

- Write or edit scripts for export, renaming, USD layer checks, and pre-publish validation.
- Run `hython script.py`, or `hython scene.hip`, to get an environment with `hou` loaded.
- Use `hbatch` for HScript. When you need them, `hrender`, or `husk` on exported USD. Check the flags against the docs for your installed version.[^2]

The loop looks like this. Treat it as an **EXAMPLE workflow pattern**, not a measured timing or a hardware spec from any machine.

1. You describe the goal: read the asset list from the shot folder, export Alembic or USD, and print the world transforms.
2. The agent writes the script in `pipeline/`, reusing your existing naming functions where it can.
3. You or the agent run `hython export_one.py`, on one asset first.
4. Read stderr and the exit code: missing node, wrong parm name, path that doesn't exist, license message.
5. Fix it and rerun **the same asset**. Once that passes, widen it to a batch over the folder.

The Terminal Skills page has a useful reminder. Node type names and internal parameter names change between versions. Read them from the current session or the docs for your version. Don't let the model recite them from memory. Python is good for managing scenes and files; big per-point loops belong in VEX.[^2]

### EXAMPLE: fixing an export script with a flipped axis

This is a reusable workflow story pattern. It's made up but common. It is **not** a claim that this ran on a specific machine yesterday.

- **Symptom:** In the downstream engine, garden props are lying on their side, or the scale is off by an order of magnitude.
- **Common causes:** The script hard-codes rotations from another DCC's habits. Units drift between meters and centimeters. `CLAUDE.md` doesn't state the up-axis, so the model falls back on Maya assumptions.[^1]
- **Fix:**
  1. Open the repo-root conventions and confirm the up-axis and units.
  2. Apply the project's conversion once, before export, instead of hand-fixing each asset.
  3. At the end of the script, `print` the rotation, scale, and bbox.
  4. Compare a single hero asset in the Houdini viewport and downstream.
- **Check:** The same asset stands upright at the same scale. Only after one asset passes do you let the batch run.

One artist on X said they aren't very familiar with Houdini but used Claude Code to help run a sim and export Alembic caches for an ML Deformer workflow on the Unreal side.[^6] Same direction: scripts and exports. The agent isn't doing the art direction. Opening that export in Unreal is a different check. The [Houdini to Unreal USD test](https://digitalgarden.si/blog/houdini-unreal-usd/) measures a composed stage and a baked HDA, including units, materials and instance counts. The simulation cook is separate again. A [Houdini ML volume upres](https://digitalgarden.si/blog/houdini-ml-volume-upres-pyro/) log starts from a small Pyro cache, compares pretrained 2× and 3× density, and only then runs a short training pass.

## Optional: Houdini MCP and skills, and the risks

In Claude Code, MCP is a channel for external tools. Once a server is connected, the agent can call whatever that server exposes. Configuration has scopes such as local, project, and user, and you can check status with `/mcp`.[^7]

There are already several open-source projects that connect Houdini to MCP, covering things like creating nodes, changing parameters, running Python or HScript, and even grabbing viewport screenshots. Creators on X have shown Claude Code + MCP + Houdini experiments such as camera-path work.[^5][^8]

**I am not vouching for the security or production readiness of any third-party MCP server.** Their architectures differ (local sockets, RPyC, a built-in web server, and so on), and the permission surface is usually large. If it can execute code, it can change your scene. The Reddit thread also has first-hand reports of "slow" and "my company won't allow it."[^4]

Before you connect one, use this table to check the impulse:

| Can go to the agent (still confirm verbally) | You watch it in the UI / by hand |
| --- | --- |
| Read-only queries of nodes, warnings, error messages | Long sims, large cooks, overnight caches |
| Editing the script repo and `hython` smoke tests | Overwriting a production `.hip` with no backup |
| VEX drafts that you paste into the Wrangle yourself | Render farm submissions, anything that creates a bill |
| A skill that explains how to write `hou` / command-line calls | Any secrets, credentials, cloud storage, or farm accounts |

The skill route packages Houdini command-line and VEX conventions as instructions the agent can read. That's more controllable than remote-controlling the whole desktop, because it mainly changes how the agent writes text. It doesn't hand over full-scene RPC by default. Which skill to install, and whether to trust it, is still on you: check where the files come from. This article doesn't rank vendors.[^2]

## High-return tasks: start with these

Pick work that is tedious, repetitive, clearly specified, and easy to catch when it goes wrong. The Retrodeck guide calls this pipeline plumbing, as opposed to "modeling for you."[^1]

**High return:**

- VEX / Attribute Wrangle drafts (you paste them into the node and cook them yourself)
- Batch renaming, path rewriting, cache folder cleanup
- FBX / Alembic / USD exporters, plus checks that material slots and key attributes are all there
- USD layer and prim naming validation
- Pre-publish checks: scale, naming, empty materials, a policy for obvious broken geometry (a check script, not automatic retopology)
- Small `hython` smoke-test scripts, instead of sending the whole shot to the farm on day one

**Low return or high risk:**

- Asking the agent to "tweak it in the viewport until it feels right"
- Large overwrite-to-disk jobs with no review
- Handing the only copy of a production shot file to an unattended rewrite

Rule of thumb: VEX handles per-element work. Python and `hou` build networks and manage files. You look at the result.[^2]

## Security red lines: paths and secrets the agent stays away from

- **Secrets and accounts:** API keys, farm credentials, cloud storage tokens. Keep them out of the chat and out of any `CLAUDE.md` you commit. When they're needed, use local environment variables or your existing secrets manager, and don't let the agent print them.
- **Write scope:** By default, only `pipeline/`, `scripts/`, and clearly marked temp output. Production caches and publish folders need a human to confirm.
- **Licenses:** `hython`, `hbatch`, and `import hou` take a license. For long-running services, release it the way the docs allow.[^2]
- **Untrusted `.hip` / `.hda` files:** They can carry scripts that run on load. Don't let the agent open files you don't know.[^2]
- **Convention isn't enforcement:** "Don't delete" in `CLAUDE.md` is only a reminder. Actual blocking takes tool permissions, hooks, or OS permissions.[^3]
- **Small before big:** Every batch job starts on one file. Talk about unattended runs only once you can undo or roll back from version control.

## How this splits with Meshy and ComfyUI

Short version: generation belongs to generation tools; glue belongs to the scripting agent.

- **ComfyUI 3D:** Text-to-mesh or image-to-mesh, multi-view shape exploration. That's the front half of the pipeline. A [ComfyUI 3D workflow](https://digitalgarden.si/blog/comfyui-3d-workflow/) test took one TRELLIS.2 draft through remesh, decimation and a GLB check in Blender and Houdini. A generated mesh is not the same as something ready for a game or a [real-time garden](https://digitalgarden.si/play/).
- **Meshy and similar generators:** Shape iteration on the web or generation side. I won't compare vendors here; the point is just to keep it separate from the scripting agent that comes after. A later [Meshy free-to-Houdini cleanup](https://digitalgarden.si/blog/meshy-3d-agent-houdini-cleanup/) downloaded one Meshy 6 Lite GLB, rejected a cleanup that opened holes, and reduced the accepted mesh to 3,000 triangles before instancing it.
- **Claude Code + Houdini:** Batch processing after cleanup, exports, USD packaging, naming validation, publish checks. The back half of the pipeline.
- On YouTube, the more common demos are Claude Code + Blender MCP building scenes or filling them with props.[^9][^10] That's live DCC control plus generation. It complements a script loop in your shot folder. Don't judge them by the same metric. Public tests of [Gemini 4 Argon for Three.js](https://digitalgarden.si/blog/gemini-4-argon-threejs/) include a Blender MCP path toward an AI-generated 3D scene, and they still need a human check before anything ships.
- A [machine-skill with Codex](https://digitalgarden.si/blog/codex-machine-skill-threejs/) test built two Three.js pump simulations from that skill. Codex was the host, not Claude Code, and the solvers still needed corrections before the numbers and cutaways agreed. A later [Claude Code and Codex wind task](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/) gave both tools the same frozen Three.js plant, then checked the unchanged builds on WebGPU and WebGL2.

The garden itself (for example, this [interactive 3D garden](https://digitalgarden.si/virtual-garden/)) makes a good practice ground. Take one plant or prop all the way through script, `hython`, and opening the result to look at it. The [OpenUSD in Three.js](https://digitalgarden.si/blog/openusd-threejs-usd-web-viewer/) note is that kind of check: a headless hython Solaris export, then the browser load. The [OpenUSD beginner stage](https://digitalgarden.si/blog/openusd-pipeline-beginners/) is the smaller file exercise underneath it: references, a payload and a season variant. Then think about batching a full set of seasonal assets. For work in progress, follow [@EmberAtelier on X](https://x.com/EmberAtelier).

## FAQ

### How is Claude Code for 3D different from asking a chatbot and copying the code?

A browser chat gives you a snippet. Claude Code edits, runs, and reads errors against your actual folder, and `CLAUDE.md` remembers your team's axis and naming. The more real context it has, the more useful it gets.[^1][^3]

### Without MCP, which Houdini tasks can Claude Code handle from the terminal?

Anything you can write as Python or `hou` and run with `hython`, `hbatch`, or `husk`: exports, checks, batch renames, smoke-test render scripts, reading cook errors and fixing them. Clicking in the viewport, tuning parameters by feel, and final artistic judgment are still yours. That's the honest scope of a Claude Code Houdini setup.[^2]

### How do I stop the agent from writing wrong-axis or Maya-flavored code?

Pin the up-axis and units in `CLAUDE.md`. Print transforms at the end of export scripts. Compare a single asset downstream first. Explicitly forbid "assume Maya." Read node and parameter names from the session for your version.[^1][^2]

### What's the minimum a human should check?

Axis and scale. Whether materials and key attributes survived. Whether a file reports a successful exit code but is empty. And open the result and look at it. Assembling a scene without looking at it is a common way agents fail.[^11]

---

## References

[^1]: Retrodeck, *Claude Code for 3D Artists: A Real Pipeline Setup* — https://retrodeck.ai/guides/every-3d-artist-should-be-using-claude-code-my-setup
[^2]: Terminal Skills, *SideFX Houdini Skill for Claude Code & AI Agents* — https://terminalskills.io/skills/houdini
[^3]: Anthropic, *How Claude remembers your project (CLAUDE.md)* — https://code.claude.com/docs/en/claude-md
[^4]: Reddit r/Houdini, *Do you use LLMs for Houdini problems?* — https://www.reddit.com/r/Houdini/comments/1l7kkss/do_you_use_llms_for_houdini_problems/
[^5]: X @JADM823 — https://x.com/JADM823/status/2108594805039079637
[^6]: X @melnukoff — https://x.com/melnukoff/status/2102733870219366851
[^7]: Anthropic, *Connect to MCP servers* — https://code.claude.com/docs/en/mcp-quickstart
[^8]: Community Houdini MCP projects (listed only to show they exist; not a security endorsement): https://github.com/atayilgun/Houdini-claudecode-mcp , https://github.com/OtakuSquirrel/houdini-mcp , https://github.com/healkeiser/fxhoudinimcp
[^9]: YouTube, *Claude Code + Blender MCP* — https://www.youtube.com/watch?v=Bdelqytx9oI
[^10]: YouTube, *Claude + 3D Model MCP* — https://www.youtube.com/watch?v=g_XF5l1ighc
[^11]: X @iamrexei — https://x.com/iamrexei/status/2100496481899712802
[^12]: X @Origin_AI_01 (CLAUDE.md as a system workflow) — https://x.com/Origin_AI_01/status/2101541975392657563
