---
layout: default
title: "Faze Gallery Improvements"
description: "Adding more information"
date: 2021-08-27 12:00:00 -0000
categories: projects faze
tags: thumbsup
excerpt_separator: <!--excerpt-->
---

The [faze gallery](https://b-faze.github.io/faze/) has been improved to give a better understanding of what the visualisation is representing. The change adds structure, tagging and a breakdown of the settings used to construct the image.

<!--excerpt-->

 This work is part of the repository's [gallery info](https://github.com/b-faze/faze/milestone/2) milestone.

# A New Structure

 ![Initial gallery structure](/assets/images/faze-gallery-diagram-initial.png)

Initially, the gallery was simply organised into high-level albums which represented the underling game. This has been reworked to split the albums further into pipeline and variation albums. 

 ![Gallery structure](/assets/images/faze-gallery-diagram.png)

 Images are still shown for each album, but the new structure allowed for created sub-albums to group the pipeline and variations. It was also possible to change the styling/content depending on if the album was at game, pipeline or variation level.

![Gallery Image Info](/assets/images/faze-gallery-album-game.png)

# Breakdown

In the following section I break down what a pipeline and variation is, in the context of the gallery. How they are formed and how they are used.

## Pipeline

 A pipeline represents all of the steps taken to create the visualisation and can expose its own configuration settings to allow for variations.

Take a look at the example pipeline below or view the code on [GitHub](https://github.com/b-faze/faze/blob/55354ea577aad7631b826b52d3a8ae8a6f46446b/src/examples/gallery/Faze.Examples.Gallery/Visualisations/PieceBoards/PieceBoardImagePipeline.cs).

 ```
ReversePipelineBuilder.Create()
    .GalleryImage(galleryService, galleryMetadata)
    .Render(new SquareTreeRenderer(config.GetRendererOptions()))
    .Paint(new PiecesBoardPainter())
    .GameTree(new SquareTreeAdapter(config.TreeSize))
    .Build(() => state);
```

Pipelines can also reference a `DataId`, if collecting the data takes a while. E.g. for collecting statistical information from simulating millions of games. 

An example of a pipeline using a `DataId` is below ([GitHub](https://github.com/b-faze/faze/blob/55354ea577aad7631b826b52d3a8ae8a6f46446b/src/examples/gallery/Faze.Examples.Gallery/Visualisations/EightQueensProblem/EightQueensProblemImagePipeline.cs))

```
ReversePipelineBuilder.Create()
    .GalleryImage(galleryService, galleryMetaData)
    .Render(new SquareTreeRenderer(config.GetRendererOptions()))
    .Paint(new EightQueensProblemPainter(config.GetPainterConfig()))
    .Map(new CommutativePathTreeMerger())
    .LoadTree(DataId, treeDataProvider);
```

... and the pipeline with corresponding `DataId` ([GitHub](https://github.com/b-faze/faze/blob/55354ea577aad7631b826b52d3a8ae8a6f46446b/src/examples/gallery/Faze.Examples.Gallery/Visualisations/EightQueensProblem/DataGenerators/EightQueensProblemExhaustiveDataPipeline.cs))

```
ReversePipelineBuilder.Create()
    .SaveTree(dataId, treeDataProvider)
    .Map(new EightQueensProblemSolutionTreeMapper(MaxEvaluationDepth))
    .GameTree(new SquareTreeAdapter(BoardSize))
    .Build(() => new PiecesBoardState(new PiecesBoardStateConfig(BoardSize, new QueenPiece(), onlySafeMoves: true)));
```

Currently, there is no link to a data pipeline from the gallery, but there will be in the future.

## Variations

 <img src="/assets/images/8QP-var1.png" />
 <img src="/assets/images/8QP-var3.png" />

A variation represents (generally) a specific configuration of a pipeline. However, the same variation may produce several images which have different configurations. This is most common for configurations supporting a depth setting, where an image is produced at each depth but the other settings remain the same.

Depending on the level of configuration a pipeline exposes, two variations may differ by superficial styling changes like border proportion or represent completely different information.

The following bit of code illustrates how a variation is created. In this case, only the depth changes. See code on [GitHub](https://github.com/b-faze/faze/blob/55354ea577aad7631b826b52d3a8ae8a6f46446b/src/examples/gallery/Faze.Examples.Gallery/Visualisations/EightQueensProblem/EightQueensProblemVis.cs).

```
new GalleryItemMetadata<EightQueensProblemImagePipelineConfig>
{
    FileId = $"8 Queens Problem Solutions depth {depth}.png",
    Album = Albums.EightQueensProblem,
    PipelineId = EightQueensProblemImagePipeline.Id,
    Variation = "Var 1",
    Depth = depth,
    Config = new EightQueensProblemImagePipelineConfig
    {
        TreeSize = 8,
        ImageSize = 600,
        MaxDepth = depth,
        BlackUnavailableMoves = true,
        BlackParentMoves = true
    }
};
```

By clicking an image's information button you are able to see a breakdown of the pipeline settings that was used to create it. The config section of the image information of created from the GalleryItemMetadata.Config property.

 ![Gallery Image Info](/assets/images/faze-gallery-img-info.png)

# Future plans

The faze gallery is intended to be a way of quickly browsing the visualisations, to document what is going on and serve as a reproducible example. Progress has been made against the first and last targets so the next focus will be improving the explanation. 

Some ideas so far:
- Add a description for an image
- Add the colour scale used
- Album descriptions (for game, pipeline and variation)
- Add interactive zooming (may need to move away from a static gallery)