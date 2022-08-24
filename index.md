# Rendering info

## Books

[PBR Book](https://www.pbr-book.org/) - offline path tracer implementation in the form of a book. Provides deep insights into ray tracing, physics and maths behind it.

## Unreal Engine

- [Global shaders in Unreal without engine modification](https://itscai.us/blog/post/ue-view-extensions/) - Despite the title, the post mainly focuses on SceneViewExtensions, a way to hook into the Unreal rendering pipeline without the need to modify engine source code. You can override SubscribeToPostProcessingPass to hook into the middle of the post processing pipeline. Also note that FSceneView inside a post processing pass does not account for screen percentage, so it's better to get target viewports for passes directly from render targets

- [UE5 Lumen Implementation Analysis](https://blog.en.uwa4d.com/2022/01/25/ue5-lumen-implementation-analysis/) - Good breakdown of the Lumen algorithms. Can't really guide you through the actual code.

- [Journey to Lumen](https://knarkowicz.wordpress.com/2022/08/18/journey-to-lumen/) - Software raytracing methods analysis.

## Raytracing

- [RTGI in Metro Exodus](https://developer.download.nvidia.com/video/gputechconf/gtc/2019/presentation/s9985-exploring-ray-traced-future-in-metro-exodus.pdf) - Key takeaways:
  - Use simpler lods for BLAS (Metro has ~4x times simpler geometry for blas)
  - Rebuild TLAS from scratch every frame (doing otherwise affects raytracing performance a lot)
  - ~1GB for acceleration structures
  - Do some extra stuff in raygen shader to hide latency (raymarch terrain heightmap for example, or it can be basically anything else, because TraceRay has a HUGE latency)
  - Don't store ray info for each pixel, reconstruct it in later passes
  - Albedo per instance (!). Color bleeding is really visible if sampled very close to ray origin -> most likely will be captured with a screen trace
  - Extract dominant direction from SH. Then fake uniform light distribution by increasing roughness, then use single BRDF sample with corrected roughness and dominant SH direction

- [Hybrid Raytracing in Far Cry 6](https://www.youtube.com/watch?v=nTZpKD600eQ)
  - [Example with source code](https://gpuopen.com/learn/hybrid-reflections/)
  - Separate ray gen from HW raytracing. Allows to classify screen tiles by complexity and only HW raytrace areas which needs raytraced reflections (puddles in front of the player mostly)
  - Use different screen-space tracing methods for mirror and glossy reflections (linear high-quality tracing for mirror, Hi-Z cone tracing for glossy)
  - Async compute for blas/tlas generation
  - Put only near geometry in TLAS
  - Use compute shaders for BLAS vertex generation, geometry pipeline is excessive and has vertex cache misses
  - Per-vertex lighting for raytraced reflections. Vertex buffers for lighting are generated beforehand and placed in a global buffer to reduce shader tables. FC6 uses only one shader table (1 hit shader and 1 empty miss shader). Metro Exodus GI has a similar idea (but stores only albedo per instance)
  - Multiple shaders in shader tables cause a lot of divergence. Avoid them if possible
  - There may be a problem when switching from HW raytracing to SSR for a tile. So don't switch to SSR if previous 2 frames used DXR for the tile. To avoid DXR takeover you can just turn DXR off for random tile pixels over time. This allows to gracefully blend 2 methods:
<img width="924" alt="fc6-raytracing-method-switch" src="https://user-images.githubusercontent.com/2443670/162637915-53f0b5c5-5ca6-45bd-9ea7-8218e2c0f926.png">

## Denoising

- [ReBLUR](https://www.springerprofessional.de/en/reblur-a-hierarchical-recurrent-denoiser/19538328) - Non-ML denoiser
  - Generate mip chain for pixels that fail temporal reprojection
  - Reccurent blur (noisy input with blurred history)
  - Separate specular and diffuse, they have different kernels and filtering space

  - Bilateral, use roughness, tangent plane distance and specular lobe params for weights

## Deferred texturing

- [Short review of current visibility buffer techniques](https://github.com/Snowball2012/fridge/blob/gh-pages/resources/DeferredTexturing.pptx?raw=true)
- [Deferred Texturing in Dawn Engine](https://www.eidosmontreal.com/news/deferred-next-gen-culling-and-rendering-for-dawn-engine/)
  - Fat VBuffer (4x32bit)
  - Has extra info for lighting (TBN, vertex normals for better SSAO and easy anisotropy)
  - No GBuffer
  - Almost no vertex attributes (1 UV channel and that's it)
- [Geometry Rendering Pipeline At Activision](https://www.youtube.com/watch?v=NoTUzzmxPo0)
  - Hybrid. Meshes can be rendered with either F+ or V+ depending on geo density and number of attributes
  - Indexing is disabled to get triangleID
  - Only 2 32bit VBuffers (1 for most geo, seconds is used only for implicit geo like foliage and deformables)
  - Uses Z-test trick to shade materials
  - Reconstructs full triangle
  
## SSR
- [Stochastic Screen-Space Reflections (SIGGRAPH 2015)] (https://www.youtube.com/watch?v=AzXEao-WKRc)
  - conceptually 2 passes (1 rpp, importance sample ggx, store intersection point, resolve on the second pass, reusing neighbour rays)
  - resolve 4 pixel simultaneously
  - raytrace in halfres, resolve in full res
  - reproject virtual reflected points for temporal stability
  - can pick best samples out of 4 for every pixel
  - cone trace mip prev frame mip chain

## Math

- [Stupid Spherical Harmonics (SH) Tricks](http://www.ppsloan.org/publications/StupidSH36.pdf) - good introduction to SH

- [Sampling the hemisphere](https://alexanderameye.github.io/notes/sampling-the-hemisphere/) - several hemisphere sampling derivations. Cone sampling can be derived easily from this

## Useful resources

- [Jendrik Illner's Graphics Programming weekly](https://www.jendrikillner.com/#posts) - An excellent weekly digest of everything that happens in the realtime cg world.
- [Technically Art](https://halisavakis.com/category/technically-art/) - Graphics Programming weekly, but for technical art. Less posts and articles, more tweets and gifs.
- [Physically Based](https://physicallybased.info/) - PBR material database. Any material can also be extracted as UE material graph
- [Krzysztof Narkowicz's blog](https://knarkowicz.wordpress.com/) - Graphics Programmer at Epic Games
