---
layout: default
monthly_highlight: true
highlight_banner_url: /assets/images/faze-video-banner.gif
title: "Introducing Faze Video"
description: "A new Faze.Rendering.Video.FFMPEG library"
date: 2021-08-28 10:00:00 -0000
categories: projects faze
tags: dev ffmpeg
excerpt_separator: <!--excerpt-->
---

A new NuGet package has been added [Faze.Rendering.Video.FFMPEG](https://www.nuget.org/packages/Faze.Rendering.Video.FFMPEG/).

This package extends the faze pipeline to add video rendering! The pipeline can now be split up and iterated over in order to produce video frames. The frames are piped to an externally installed ffmpeg command, so no filling up directories with hundreds of images - just straight to video!

<!--excerpt-->

The faze gallery also supports video, [so check them out!](https://b-faze.github.io/faze/)

# Pipeline recap

![image pipeline](/assets/images/faze-pipelines-image.png)

In its simplest form, a pipeline will look like the above diagram. Faze has ways of short-cutting some of the steps by loading a tree from file, but it is essentially:

1. Some arbitrary tree (usually a game state tree)
2. Tree is mapped to a painted tree (colour tree)
3. Painted tree is drawn by a renderer
4. Renderer writes image to a stream
5. Stream is a FileStream.

In code, such a pipeline looks like:

```
ReversePipelineBuilder.Create()
    .GalleryImage(galleryService, galleryMetadata)
    .Render(new SquareTreeRenderer(config.GetRendererOptions()))
    .Paint(new GoldInterpolator())
    .Map<double, WinLoseDrawResultAggregate>(t => t.MapValue(v => v.GetWinsOverLoses()))
    .LoadTree(OXDataGeneratorExhaustive.Id, treeDataProvider);
```

A common way to map from a state tree to a painted tree is to sample results of playing random games and assign a colour based on the likelihood of winning. 

What if we want to produce a video that shows how the visualisation changes as more random games are played and the probabilities start to converge?

# Introducing the Video Pipeline

![video pipeline](/assets/images/faze-pipelines-video.png)

The pipeline now has three new methods: 

- **Iterate** splits the pipeline output.

- **Map** applies a sub-pipeline to each of the new outputs.

- **Merge** joins the output of all the sub-pipelines into a single output.

Here it is in code ([GitHub](https://github.com/b-faze/faze/blob/master/src/examples/gallery/Faze.Examples.Gallery/Visualisations/OX/OXGoldVideoPipeline.cs)):

```
ReversePipelineBuilder.Create()
    .GalleryVideo(galleryService, galleryMetadata)
    .Merge()
    .Map(builder => builder
        .Render(new SquareTreeRenderer(config.GetRendererOptions()))
        .Paint(new GoldInterpolator())
        .Map<double, WinLoseDrawResultAggregate>(t => t.MapValue(v => v.GetWinsOverLoses()))
    )
    .Iterate(new WinLoseDrawResultsTreeIterater(new GameSimulator(), config.LeafSimulations))
    .LimitDepth(config.MaxDepth)
    .GameTree(new SquareTreeAdapter(3))
    .Build(() => OXState.Initial);
```

The `WinLoseDrawResultsTreeIterater` is responsible for keeping track of the game results and aggregating them for each simulation - returning an `IEnumerable<Tree<WinLoseDrawResultAggregate>>`.

The **Map** function is the same as the earlier image pipeline example, as from its perspective it is dealing with a single image (a video frame in this case).

The new `Video` pipeline method expects a single Stream which is the concatenation of all the frame data. The **Merge** method iterates over renderers and writes each of their images to the same stream.

# Piping to FFMPEG

This is surprisingly simple, although it was confusing to get there as I am unfamiliar with FFMPEG and piping to other programs in C#.

You can take a look at the full code [here](https://github.com/b-faze/faze/blob/55354ea577aad7631b826b52d3a8ae8a6f46446b/src/libraries/Faze.Rendering.Video.FFMPEG/Faze.Rendering.Video.FFMPEG/Extensions/ReversePipelineBuilderExtensions.cs#L23). 

As I am writing this, the package is still in pre-release and needs some improvements but the code is fine to illustrate how it works:

```
...

 var startInfo = new ProcessStartInfo
{
    FileName = "ffmpeg",
    Arguments = $"-y -f image2pipe -vcodec {imgVCodec} -r {fps} -i - -codec:v mpeg4 -q:v 2 -r {fps} \"{filename}\"",
    RedirectStandardInput = true,
    ...
}

using (Process process = Process.Start(startInfo))
{
    streamer.WriteToStream(process.StandardInput.BaseStream);
}

...

process.WaitForExit();
```

The two main things you need to be aware of are:
- `RedirectStandardInput` needs to be set to true, then it is as simple as writing to the `process.StandardInput.BaseStream`
- Be careful with any encoding applied to the bytes sent over the stream (I don't use any, it goes straight from SkiaSharp to FFMPEG). Using stream reader/writers can encode the bytes in a way that will corrupt the video or cause ffmpeg to error. 

# Future Plans

There are a lot of use cases for rendering videos!

### Exploratory Videos
For pipelines that are not fast enough for real-time rendering, it allows for creating high-quality zooming and exploration videos.

Reverse-zoom videos, where you start at a specific game state or end result and then zoom back out of the image, showing where is lies compared to all other games and how it is only a tiny fraction of the ways to play.

### Animations
Effects could be shown like colour changes and pulsing effects to highlight certain things or just make it look fancier.


### Rendering Short-circuit
Being able to split and merge the pipeline allows for creating new generic methods. 

Imagine wanting to create a visualisation by sampling game results over and over again (like with the example above), but instead of giving a specific number of samples it keeps sampling until it has reached some convergence threshold.