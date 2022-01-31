# Rendering info

## Books

[PBR Book](https://www.pbr-book.org/) - offline path tracer implementation in the form of a book. Provides deep insights into ray tracing, physics and maths behind it.

## Unreal Engine

- [Global shaders in Unreal without engine modification](https://itscai.us/blog/post/ue-view-extensions/) - Despite the title, the post mainly focuses on SceneViewExtensions, a way to hook into the Unreal rendering pipeline without the need to modify engine source code. You can override SubscribeToPostProcessingPass to hook into the middle of the post processing pipeline. Also note that FSceneView inside a post processing pass does not account for screen percentage, so it's better to get target viewports for passes directly from render targets

- [UE5 Lumen Implementation Analysis](https://blog.en.uwa4d.com/2022/01/25/ue5-lumen-implementation-analysis/) - Good breakdown of the Lumen algorithms. Can't really guide you through the actual code.

## Useful resources

- [Jendrik Illner's Graphics Programming weekly](https://www.jendrikillner.com/#posts) - An excellent weekly digest of everything that happens in the realtime cg world.
- [Technically Art](https://halisavakis.com/category/technically-art/) - Graphics Programming weekly, but for technical art. Less posts and articles, more tweets and gifs.
