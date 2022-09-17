---
layout: page
title: "Skip SSAO on areas lit by direct lighting"
permalink: /ssao-skip-direct/
---

# Skip SSAO on areas lit by direct lighting

SSAO is usually used as a relatively cheap occlusion term for indirect lighting. There are some techniques that leverage ssao term for direct lighting contact shadows approximation. But for these kinds of features prebaked occlusion maps are usually more preferrable than gbuffer-based SSAO.
In most scenes direct lighting is much more intensive than indirect bounces (may be orders of magnitude higher in some outdoor scenes). That means we don't have to calculate high-quality indirect lighting in directly lit areas if that lighting is bright enough. SSAO term can be omitted on these areas without noticeable visual difference.

## The algorithm

1. Compute direct lighting as usual, but keep it separated from indirect lighting.
2. At the start of SSAO pass sample direct lighting intensity, early out if direct is much more intensive than indirect

## Results
SSAO buffer (before blur):

baseline:
![](https://github.com/Snowball2012/fridge/blob/gh-pages/resources/ao_skip_direct_lit_areas/classic_hbao.png?raw=true)
optimized:
![](https://github.com/Snowball2012/fridge/blob/gh-pages/resources/ao_skip_direct_lit_areas/optimized_hbao.png?raw=true)

Resulting frame:

baseline:
![](https://github.com/Snowball2012/fridge/blob/gh-pages/resources/ao_skip_direct_lit_areas/classic_frame.jpg?raw=true)
optimized:
![](https://github.com/Snowball2012/fridge/blob/gh-pages/resources/ao_skip_direct_lit_areas/optimized_frame.jpg?raw=true)

Timings (1080p SSAO with 4k output on NVidia GTX 1080Ti):

baseline:
![](https://github.com/Snowball2012/fridge/blob/gh-pages/resources/ao_skip_direct_lit_areas/classic_timings.png?raw=true)
optimized:
![](https://github.com/Snowball2012/fridge/blob/gh-pages/resources/ao_skip_direct_lit_areas/optimized_timings.png?raw=true)

we can see 25% speedup in the main SSAO pass on a relatively well-lit outdoor scene, which is good for such a simple optimization

## Caveats

These results does not actually estimate direct to indirect intensity ratio, I just early out in every pixel with direct lighting intensity above some constant threshold. Apparently sampling 2 4k RGBA16 textures is so bad bandwidth-wise, that you don't gain any speedup if you sample them both in HBAO shader. But this can be avoided if at some point "ssao mask" is generated, e.g. in forward pass.
Another issue with this approach comes from the fact that you need final direct lighting before computing SSAO, and that means it can't be used to fake contact shadows anymore. Plus that moves SSAO forward quite a bit, so you can't use async compute to overlap AO with shadowmaps, for example. Maybe it is possible to build ssao mask on reprojected previous frame, but I expect some visual artifacts at the shadow edges
