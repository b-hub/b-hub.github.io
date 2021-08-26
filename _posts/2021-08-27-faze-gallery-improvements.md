---
layout: default
title: "Faze Gallery Improvements"
description: "Adding more information to the display"
date: 2021-08-27 12:00:00 -0000
categories: projects faze
tags: thumbsup
excerpt_separator: <!--excerpt-->
---

This month I have worked on improving the [faze gallery](https://b-faze.github.io/faze/) to give a better understanding of what the visualisation is representing. The change adds structure, tagging and a breakdown of the settings used to construct the image.

<!--excerpt-->

 This work is part of the repository's [gallery info](https://github.com/b-faze/faze/milestone/2) milestone.

## The New Structure

 ![Initial gallery structure](/assets/images/faze-gallery-diagram-initial.png)

Initially, the gallery was simply organised into high-level albums which represented the underling game. This has been reworked to split the albums further into pipeline and variation albums. 

 ![Gallery structure](/assets/images/faze-gallery-diagram.png)

 This allowed for styling pipeline and variation albums differently and in the case of the pipeline, including a link to the code on github.

 ### Pipeline

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

Pipelines can also reference a `DataId` if collecting the data takes a while. E.g. for collecting statistical information from simulating millions of games. 

An example of a pipeline using a `DataId` is below or on [GitHub](https://github.com/b-faze/faze/blob/55354ea577aad7631b826b52d3a8ae8a6f46446b/src/examples/gallery/Faze.Examples.Gallery/Visualisations/EightQueensProblem/EightQueensProblemImagePipeline.cs).

```
ReversePipelineBuilder.Create()
    .GalleryImage(galleryService, galleryMetaData)
    .Render(new SquareTreeRenderer(config.GetRendererOptions()))
    .Paint(new EightQueensProblemPainter(config.GetPainterConfig()))
    .Map(new CommutativePathTreeMerger())
    .LoadTree(DataId, treeDataProvider);
```

### Variations

A variation represents (generally) a specific configuration of a pipeline. However, the same variation may produce several images which have different configurations. This is most common for configurations supporting a depth setting, where an image is produced at each depth but the other settings remain the same.

Depending on the level of configuration a pipeline exposes, two variations may differ by superficial styling changes like border proportion or represent completely different information.

By clicking an image's information button you are able to see a breakdown of the pipeline settings that was used to create it.

 ![Gallery Image Info](/assets/images/faze-gallery-img-info.png)