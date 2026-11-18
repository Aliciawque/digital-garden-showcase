# What a neural texture and a heightfield taught me about frequency bands

[Full measurements and figures on Digital Garden](https://digitalgarden.si/blog/houdini-22-machine-learning-workbook/)

A neural texture and a heightfield both accept coordinates. That similarity made them a good pair for a Houdini FX 22.0.429 exercise. I could change the representation, measure the error on held-out coordinates, and then ask whether the extra inputs earned their place.

For the texture, I drew 100,000 UV samples and reserved 10% for validation. Two networks of the same width predicted RGB, specular intensity and a mask. Appending eight frequency bands to UV brought the combined RMSE from 0.26167 to 0.01507. Along a shared edge I split the vertices before inference so coincident positions could retain different UVs.

![Raw UV and encoded UV predictions beside the reference RGB texture and mask](https://digitalgarden.si/marketing/research/h22-workbooks-20261117/ml/neural-texture.png)

On the terrain, 1,000 scattered points told a different story. A nine-band encoding raised the height RMSE from 5.56 to 10.42 world units on the same 10,000-point grid. Evaluating the procedural terrain formula directly scored 0.0133. With an allowed error of one unit, the original formula already met the brief.

The model export needed its own check. PyTorch and Houdini agreed within 0.0000363 on those 10,000 coordinates. That answers whether the two executors implement the same prediction. The reference grid answers whether that prediction describes the terrain. Both numbers are worth keeping.

Codex assisted with the experiments and writing. [English study](https://digitalgarden.si/blog/houdini-22-machine-learning-workbook/) · [中文实测](https://digitalgarden.si/zh/blog/houdini-22-machine-learning-workbook/).
