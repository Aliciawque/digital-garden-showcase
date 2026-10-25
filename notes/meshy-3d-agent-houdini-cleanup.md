# Meshy Free to Houdini: Testing the Download and Cleanup Path

> AI-assisted disclosure: Codex assisted with research, code and writing. This is a copy of the published article; its original dates, measurements and limitations are retained.

> A real Meshy 6 Lite GLB taken through Houdini: free-download limits, topology checks, a rejected cleanup, 3,000-triangle reduction and browser instancing.

- Author: Alicia Lu
- Published: 2026-10-20
- Updated: 2026-10-21
- Canonical HTML: https://digitalgarden.si/blog/meshy-3d-agent-houdini-cleanup/

The free route worked, with an important boundary: Meshy 3D Agent could not select Meshy 6 Lite, and the Agent's higher-tier generated models were not downloadable on this free account. I used the regular Text to 3D tool with 6 Lite, downloaded a real GLB, and tested the Houdini-to-browser pipeline from that file. This article is a free-workflow test, not a paid Agent export demonstration.

The other plant asset and renderer tests are collected in [the garden's Three.js rendering overview](https://digitalgarden.si/blog/threejs-3d-garden-rendering/).

Codex operated Meshy's web UI, Houdini 22.0.429 and the Three.js acceptance viewer on 20 October 2026. The account was already logged in by its owner. No paid subscription was purchased.

![Actual generated Meshy source compared with the reduced result under the same inspection material](https://digitalgarden.si/marketing/research/comparisons-20261020/meshy-comparison.png)

## Verify the download before spending credits

Meshy's current free-plan help described 100 monthly credits and 10 monthly downloads limited to Meshy 6 Lite. The Agent model selector offered higher-tier models, but no Lite option; its chat answer also directed the free-download workflow to the regular generator. I checked the actual selector and download controls rather than treating the chat answer as proof.[^1]

The regular Text to 3D panel allowed Meshy 6 Lite, quoted 10 credits and selected CC BY 4.0. The prompt requested one stylized grape hyacinth: a slender green stem, five narrow leaves and hanging purple-blue bells, with no pot, soil or ground plane. One generation completed and the GLB download counter changed from 10 to 9. The account also received a separate 30-credit daily check-in reward, so its final balance of 120 is not the generation cost.

The page displayed a notice that Text to 3D would be retired on 31 October. That is a dated UI observation, not a promise that this exact free path will still exist afterward. Recheck access and download entitlement before a new batch; do not spend a session generating models that the account cannot export.

## What the downloaded file actually contained

The GLB was 784,708 bytes, with 21,781 vertices and 43,546 triangles. Its only mesh attribute was POSITION. It contained no UVs, normals, material images or PBR materials. The requested flower colours therefore did not arrive as a textured deliverable. It was a geometry result, not a ready botanical asset with verified species anatomy.

Houdini's imported and unpacked geometry had 13 connected components, zero boundary edges and zero edges shared by more than two faces. Those checks do not prove there are no self-intersections, that all parts form one connected object, or that it is ready for printing. The website's printable badge is not substituted for those measurements.

The original file was preserved. Derived copies were centered in X/Z, placed at ground height and normalized to 0.6 units tall. The later browser inspection applies a common neutral material, with computed display normals where needed; that material is not a generated Meshy texture.

## A failed cleanup made the tolerance problem visible

The first Clean/PolyDoctor chain used defaults after the 0.6-unit normalization. It removed much of the thin geometry: only 23,817 triangles remained, with 11,625 boundary edges and 69 components. I rejected it. A smaller file with more holes is not an accepted cleanup.

The accepted Clean degeneracy tolerance was reduced to 1e-12. Potential overlap/intersection conditions were marked rather than silently repaired. The accepted cleanup preserved all 43,546 triangles and the original closed-edge/component counts. SideFX documents that Clean's degeneracy tolerance is an edge-length measure whose square is compared with area; scale matters.[^2]

PolyReduce then used its actual Output Polygon Count target with finalcount=3000. Merely writing a count while leaving its target on percentage would not activate that target. The remesh branch used target size 0.006 and five iterations. Its loss of some leaf width is visible and consistent with the changed bounds, so it is not called visually identical.

| Geometry stage | Points | Triangles | Boundary edges | Nonmanifold edges | Components |
| --- | --- | --- | --- | --- | --- |
| raw | 21,781 | 43,546 | 0 | 0 | 13 |
| clean | 21,781 | 43,546 | 0 | 0 | 13 |
| reduce | 1,508 | 3,000 | 0 | 0 | 13 |
| remesh | 2,785 | 5,554 | 0 | 0 | 13 |

All derived branches received a six-plane UV unwrap and computed normals before native GLB export. That creates inspectable UVs; it is not a hand-authored low-distortion UV layout. There was no source PBR image to rebake, so I did not invent a texture-baking success. The HIP was saved, reopened and cooked again.

## Validate the browser deliverable

The source and final GLBs passed glTF validation with zero errors and warnings; informational and buffer-view hints remain in the receipts. Three.js 0.186.1 loaded each candidate through five unique localhost URLs. Timings include parsing, compilation and a submitted frame with warm code/GPU pipelines, not public-network or device-cold time.

| Candidate | Bytes | Median local load to submitted frame |
| --- | --- | --- |
| Original download | 784,708 | 33.3 ms |
| clean, with generated UVs/normals | 2,025,332 | 33.3 ms |
| reduce, with generated UVs/normals | 214,740 | 16.6 ms |
| remesh, with generated UVs/normals | 391,480 | 16.6 ms |

Adding UVs and normals makes the full-resolution derived file larger than the position-only download; reduction and attribute creation are different operations. The 3,000-triangle candidate was repeated 1,000 times with explicit InstancedMesh: the receipt records 2 total render calls and 3,000,001 triangles, including the node renderer's output pass. Instancing shares data but does not remove the repeated triangle workload. Both WebGPU and forced WebGL2 fallback were checked.

An independently authored Houdini hyacinth is included as a reference for geometry budget and visible structure, not as botanical ground truth or a blinded aesthetic comparison. The free AI result still needs art direction, material work, appropriate deformation attributes and a scene-specific acceptance check before replacing a production garden asset.

## Reproduce it and retain attribution

The [asset-test package](https://digitalgarden.si/marketing/research/comparisons-20261020/assets-tests.zip) contains the original free download, derived meshes, Houdini scripts, viewer and results. The Meshy source and its derivatives retain CC BY 4.0 attribution: model created with Meshy by Alicia. The package includes the license link and records the modifications. It does not grant rights to the private garden engine or its production models.[^3]

For a procedural alternative with explicit shape controls and wind data, read [the Houdini plant test](https://digitalgarden.si/blog/houdini-procedural-plants-realtime-web/). For the runtime budget, continue with [the nine-path instancing matrix](https://digitalgarden.si/blog/threejs-webgpu-vs-webgl-instanced-plants/).

[^1]: Meshy, [what is included on the free plan](https://help.meshy.ai/en/articles/15696428-what-is-included-on-the-free-plan) and [3D Agent documentation](https://docs.meshy.ai/en/webapp/3d-agent), checked 20 October 2026. Entitlement was also verified by an actual download.
[^2]: SideFX, [Clean SOP](https://www.sidefx.com/docs/houdini/nodes/sop/clean.html) and [PolyReduce SOP](https://www.sidefx.com/docs/houdini/nodes/sop/polyreduce.html), checked 20 October 2026.
[^3]: Meshy, [ownership of generated models](https://help.meshy.ai/en/articles/10137554-what-is-the-ownership-of-the-generated-models), checked 20 October 2026. The model is distributed with [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) attribution and modification notes.
