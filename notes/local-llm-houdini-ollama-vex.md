# Ollama in Houdini: 60 VEX Tests on a 16GB RTX 5070 Ti

> Sixty native Houdini VEX trials compare local Qwen2.5-Coder 7B and 14B in Ollama, with unedited code, error feedback, timing and reproducible evidence.

- Author: Alicia Lu
- Published: 2026-11-02
- Canonical HTML: https://digitalgarden.si/blog/local-llm-houdini-ollama-vex/

**A local Ollama endpoint worked from Houdini 22, but fast VEX replies were often wrong.** In 60 frozen trials on an RTX 5070 Ti, Qwen2.5-Coder 7B and 14B each passed 5/30 first attempts. With at most two compiler or semantic feedback repairs, they reached 8/30 and 11/30. I kept all 156 generated snippets unchanged.

This is a controlled point-attribute study in **headless Houdini 22.0.429**, using owned synthetic fixtures. It does not measure an open garden viewport, artist productivity, production assets, or every local code model. Codex assisted the implementation, execution, checks and writing; the counts come from saved native results.

![Local VEX correctness: both models pass 5 of 30 first attempts; feedback raises 7B to 8 and 14B to 11](https://digitalgarden.si/marketing/research/ollama-vex-20261102/pass-rates.png)

## What actually ran

The machine was Windows 11, Ryzen 9 9950X, 64 GB RAM and an RTX 5070 Ti with 16,303 MiB reported by driver 617.42. Houdini ran its own Python 3.13 through `hython`; the API bridge required only Python's standard library. Ollama was **0.13.2**, with CUDA 13 selected by its own runtime. I retained that version for this study rather than installing the downloaded updater.

I started a separate Ollama service at `http://127.0.0.1:11435`, with a dedicated model directory, one parallel request and one resident model. The ordinary default port is 11434. These scripts use 11435 deliberately and do not change the workstation's global Ollama or proxy settings.

| Tested model | Quantization | Registry size, bytes | First correct | Correct after ≤2 repairs |
|---|---|---|---|---|
| qwen2.5-coder:7b-instruct | Q4_K_M | 4,683,087,561 | 5/30, 16.7% | 8/30, 26.7% |
| qwen2.5-coder:14b-instruct | Q4_K_M | 8,988,124,298 | 5/30, 16.7% | 11/30, 36.7% |

These are the two models tested, not a recommendation or a claim about the newest model releases. Full manifest digests and `/api/show` responses are in the reproduction package. For reference, the pinned digest prefixes are `dae161e27b0e` and `9ec8897f747e`.

## Ten tasks, frozen before inference

The formal plan records SHA-256 hashes of the task specification and native validator before model inference. Each model saw ten tasks with seeds 11, 23 and 47: **30 trials per model**. Task order rotated identically for both models. Settings were temperature 0.2, top-p 0.9, context 8192 and at most 768 generated tokens per attempt.

The tasks were a height mask, height-driven color, safe normal normalization, an integer-ID scale pattern, radial smoothstep fade, a specified right-handed Y rotation, root-pinned wind, a sine terrain height, point-group membership and bounding-box UVs with a zero-extent fallback. Every constant and convention appeared in the prompt. These are mathematical attribute exercises inspired by plant work, not tasks extracted from the production garden.

Each snippet ran unedited in a point Attribute Wrangle against three fixtures: a 45-point grid, 37 deterministic jittered points and nine edge cases. The last fixture includes a degenerate X bounding box; normals include zero vectors. Time was fixed at frame 13 and 24 fps, giving `@Time = 0.5`.

An independent Python oracle checked the requested values within absolute tolerance `2e-5`, tuple widths, finite values, point count, preserved inputs and unrequested attributes or groups. All three fixtures had to pass. Ten known-correct controls passed first; four deliberately wrong controls covering values, position changes, extra attributes and syntax were rejected.

The prompt prohibited loops, geometry creation/deletion, I/O, channel references and external geometry. A single code fence could be removed; the code inside was not repaired by hand. Failure feedback contained native compiler messages or numeric mismatches, capped at 3500 characters. There were at most three attempts in the same conversation.

I call the second result **success after feedback**, because these are dependent repair attempts. It is not the independent-sampling pass@k estimator, and three seeds per task are too few to establish broad model rankings.

![Correct outcomes per task after up to two feedback repairs, with all three repeats shown for each model](https://digitalgarden.si/marketing/research/ollama-vex-20261102/task-matrix.png)

## Compiling and being correct are separate checks

On first attempts, 7B compiled on all fixtures in 7/30 trials but was correct in 5/30. The 14B model compiled and passed in 5/30. A compile-only check would have accepted two extra 7B failures.

Failures included undefined unprefixed variables, invented function calls and incorrect attribute assignment. One 7B height response declared a local `height_mask`, attempted `point(...).y` without resolving the overloaded return type, and modified `P` despite being asked to preserve it. Some repair attempts repeated the same faulty code. A rotation response called `rotate` as if it accepted and returned a point vector; Houdini rejected that signature.

The terrain task passed all three first attempts for both models. Neither model solved the radial fade, root-pinned wind or UV task in any of the three repair trials. That is a result for these exact prompts and checks, not evidence that local models can never solve those tasks.

The larger model added six repaired successes after its five first-attempt successes; 7B added three. A plausible-looking snippet and a fast answer were insufficient grounds to apply code to an active scene.

## Latency and memory

| Metric | 7B | 14B |
|---|---|---|
| Median first response after warm-up | 0.474 s | 0.865 s |
| Median first nonempty streamed token | 66.3 ms | 79.4 ms |
| Median summed model-response time per task, including repairs | 1.553 s | 2.461 s |
| Ollama resident allocation after load | 4.975 GiB | 10.118 GiB |
| Separate cold warm-up wall time | 32.068 s | 2.031 s |

The first two latency rows use all 30 first attempts per model, including failures. Summed response time includes model requests but excludes native validation and interaction time. Ollama's prefix caching remained enabled, and these snippets were short. This is not a fresh-process startup measurement for every request.

The cold warm-ups were separate setup observations. Model order and system state differed, so the 32-second versus two-second observations do not establish a general load-time advantage for 14B.

![Measured warm response latency and Ollama resident allocation, with distinct units and scope](https://digitalgarden.si/marketing/research/ollama-vex-20261102/latency-memory.png)

The memory numbers are `/api/ps` `size_vram` divided by 2³⁰. They are **Ollama resident allocation, not total driver peak or spare viewport capacity**. A parallel 200 ms `nvidia-smi` logger produced malformed CSV rows because its asynchronous appends were not serialized. I preserved that failed diagnostic and excluded it from driver-peak claims. Both models unloaded successfully afterward, with an empty `/api/ps` model list. No Houdini GUI responsiveness conclusion follows from this headless study.

## The Python client that worked in Houdini

Ollama's [chat API](https://docs.ollama.com/api/chat) returns text in `message.content`; the [generate API](https://docs.ollama.com/api/generate) uses `response`. Both can stream. For a small blocking client, explicitly disable streaming:

```python
import json
import urllib.request
import urllib.error

API = "http://127.0.0.1:11435"
opener = urllib.request.build_opener(urllib.request.ProxyHandler({}))

def chat(model, messages, timeout=180):
    body = {
        "model": model, "messages": messages, "stream": False,
        "keep_alive": "5m",
        "options": {"temperature": 0.2, "num_ctx": 8192,
                    "num_predict": 768},
    }
    request = urllib.request.Request(
        API + "/api/chat", data=json.dumps(body).encode("utf-8"),
        headers={"Content-Type": "application/json"},
    )
    try:
        with opener.open(request, timeout=timeout) as response:
            data = json.load(response)
    except urllib.error.HTTPError as error:
        raise RuntimeError(
            f"Ollama HTTP {error.code}: "
            + error.read().decode("utf-8")
        ) from error
    if data.get("error"):
        raise RuntimeError(data["error"])
    if not data.get("done"):
        raise RuntimeError("Incomplete response")
    return data["message"]["content"], data
```

The direct opener matters on this workstation: the inherited Python proxy configuration did not bypass `127.0.0.1`. A controlled GET to `/api/version` returned 502 through the inherited opener and 200 with version 0.13.2 through the direct opener. The change is scoped to this loopback client; it does not disable the user's OS proxy.

I also reproduced three distinct failure cases in native Houdini Python:

| Test | Actual result | Correct handling |
|---|---|---|
| Read `reply["response"]` from a successful chat response | `KeyError: 'response'` | Read `reply["message"]["content"]` |
| Call `json.loads` on the entire streamed generate body | `JSONDecodeError: Extra data` | Parse individual lines, or request `stream: false` |
| Request an intentionally missing model | HTTP 404 with an error body | Report the HTTP error before reading a text field |

Those are local reproductions. They do not establish what caused an error in somebody else's tutorial. The [streaming documentation](https://docs.ollama.com/api/streaming) describes the newline-delimited response contract.

## Apply code through a checked controller

For the formal experiment, a controller set `snippet` on a point Attribute Wrangle, requested its geometry and checked the native result. A Python SOP also made a real localhost request and returned `READY` while preserving 60 input points. The separate demonstration replayed the accepted, unedited 14B color snippet from formal trial 1:

```c
float t = clamp(@P.y / 1.5, 0, 1);
@Cd = lerp(set(0.05, 0.20, 0.03), set(0.80, 0.30, 0.10), t);
```

![Sixty native Houdini points colored by the accepted unedited VEX snippet, plotted from HOM readback](https://digitalgarden.si/marketing/research/ollama-vex-20261102/native-points.png)

The editable HIP was saved and reopened, and its point count and color attribute were read back. Its saved Python SOP replays the verified HTTP reply instead of automatically starting inference on opening. The figure is a plot of native point data with a linear-to-sRGB display conversion, not a viewport screenshot.

Synchronous HTTP calls block the Python cook until completion. I did not test a threaded GUI controller here. A production tool needs its own UI handling, timeouts, cancellation and scene-specific validation; this package demonstrates the bounded headless path.

## FAQ

**Can a local model write working Houdini VEX?** Yes, some snippets passed native semantic checks. Both tested models passed 5/30 first attempts; feedback increased the totals to 8/30 and 11/30.

**Does 14B fit on this 16GB card?** It loaded with a reported 10.118 GiB Ollama resident allocation during this headless study. An open production scene and responsive viewport were not tested.

**Why does `/api/chat` produce `KeyError: 'response'`?** A successful chat response uses `message.content`. I reproduced the wrong-key exception. Streaming parse failures and HTTP errors are separate cases.

**Does this code upload the scene?** The tested client sends only the declared messages and options to loopback. It does not serialize geometry or call a cloud inference endpoint. Model downloads are a separate network operation; this is not a complete workstation network audit.

**Should I paste generated VEX into my active garden?** These results support checking it in an isolated scene first. Even compiling was not sufficient to prove correctness in this task set.

## Data and reproduction

Download the [reproduction ZIP](https://digitalgarden.si/marketing/research/ollama-vex-20261102/ollama-vex-study-20261102.zip), [analysis JSON](https://digitalgarden.si/marketing/research/ollama-vex-20261102/analysis.json), [all-attempt CSV](https://digitalgarden.si/marketing/research/ollama-vex-20261102/results.csv) and [README](https://digitalgarden.si/marketing/research/ollama-vex-20261102/README.md). The package includes the frozen prompts and validator, every request and response, separate setup/integration failures, model metadata, figures and editable HIP. Model weights and user settings are excluded.

Related measured work: [RTX 5070 Ti in Houdini](https://digitalgarden.si/blog/rtx-5070-ti-houdini-benchmark/), [Claude Code versus Codex for Three.js](https://digitalgarden.si/blog/claude-code-vs-codex-threejs/) and [procedural Houdini plants for the web](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/).
