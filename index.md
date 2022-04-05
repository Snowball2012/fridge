# Rendering info

## Books

[PBR Book](https://www.pbr-book.org/) - offline path tracer implementation in the form of a book. Provides deep insights into ray tracing, physics and maths behind it.

## Unreal Engine

- [Global shaders in Unreal without engine modification](https://itscai.us/blog/post/ue-view-extensions/) - Despite the title, the post mainly focuses on SceneViewExtensions, a way to hook into the Unreal rendering pipeline without the need to modify engine source code. You can override SubscribeToPostProcessingPass to hook into the middle of the post processing pipeline. Also note that FSceneView inside a post processing pass does not account for screen percentage, so it's better to get target viewports for passes directly from render targets

- [UE5 Lumen Implementation Analysis](https://blog.en.uwa4d.com/2022/01/25/ue5-lumen-implementation-analysis/) - Good breakdown of the Lumen algorithms. Can't really guide you through the actual code.

## Raytracing

- [RTGI in Metro Exodus](https://developer.download.nvidia.com/video/gputechconf/gtc/2019/presentation/s9985-exploring-ray-traced-future-in-metro-exodus.pdf) - Key takeaways:
  - Use simpler lods for BLAS (Metro has ~4x times simpler geometry for blas)
  - Rebuild TLAS from scratch every frame (doing otherwise affects raytracing performance a lot)
  - ~1GB for acceleration structures
  - Do some extra stuff in raygen shader to hide latency (raymarch terrain heightmap for example, or it can be basically anything else, because TraceRay has a HUGE latency)
  - Don't store ray info for each pixel, reconstruct it in later passes
  - Albedo per instance (!). Color bleeding is really visible if sampled very close to ray origin -> most likely will be captured with a screen trace
  - Extract dominant direction from SH. Then fake uniform light distribution by increasing roughness, then use single BRDF sample with corrected roughness and dominant SH direction

## Denoising

- [ReBLUR](https://www.springerprofessional.de/en/reblur-a-hierarchical-recurrent-denoiser/19538328) - Non-ML denoiser
  - Generate mip chain for pixels that fail temporal reprojection
  - Reccurent blur (noisy input with blurred history)
  - Separate specular and diffuse, they have different kernels and filtering space
  - Bilateral, use roughness, tangent plane distance and specular lobe params for weights

## Useful resources

- [Jendrik Illner's Graphics Programming weekly](https://www.jendrikillner.com/#posts) - An excellent weekly digest of everything that happens in the realtime cg world.
- [Technically Art](https://halisavakis.com/category/technically-art/) - Graphics Programming weekly, but for technical art. Less posts and articles, more tweets and gifs.
