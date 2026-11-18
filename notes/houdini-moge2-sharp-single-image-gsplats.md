# Why a portrait camera changed a million-splat render

[Full measurements and figures on Digital Garden](https://digitalgarden.si/blog/houdini-moge2-sharp-single-image-gsplats/)

I built an asymmetric 512 × 768 still life with a known 24 mm camera to check what survived a single-image reconstruction. The foreground sphere makes a stretched image easy to spot, and the background stripes give a second cue when the camera moves.

The FP32 SHARP conversion returned 1,179,648 Gaussians. I first checked the Houdini volume layout with four known channel values. On this machine the RGB image arrived as one three-component volume, and the four probe values agreed using ZYX with separate channel collation.

![Square assumption, calibrated portrait camera and comparison overlay](https://digitalgarden.si/marketing/research/h22-workbooks-20261117/splats/aspect.png)

The render still depended on the camera. Restoring the portrait intrinsics to the centers and ellipsoid covariances brought mean absolute RGB error down from 0.04538 to 0.01249. In a mirror control the error rose to 0.07596. All three comparisons used the same source pixels.

MoGe-2 also produced a depth map, normals, positions and 417,534 points. The two reconstructions were rendered from the front and with a small lateral camera move in Karma XPU. These views show image agreement and parallax for one synthetic scene. Absolute scene depth and surfaces hidden behind the foreground were outside the test.

Codex assisted with the experiments and writing. [English study](https://digitalgarden.si/blog/houdini-moge2-sharp-single-image-gsplats/) · [中文实测](https://digitalgarden.si/zh/blog/houdini-moge2-sharp-single-image-gsplats/).
