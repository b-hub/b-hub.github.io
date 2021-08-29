---
layout: default
title: "thumbsup Custom Theme & Metadata"
description: "How to add custom metadata to a thumbsup gallery"
date: 2021-08-29 12:00:00 -0000
categories: projects faze
tags: dev thumbsup
excerpt_separator: <!--excerpt-->
---

This is how to integrate custom metadata into a thumbsup gallery and how I used the methods to include and display custom data in the faze gallery.

<!--excerpt-->

# The Problem

The standard thumbsup themes do not meet the new styling / content requirements of the Faze gallery. 

In the blog post [Faze Gallery Improvements](/projects/faze/faze-gallery-improvements.html), I describe the new structure of the gallery. It has two main requirements:
- Albums rendered differently depending on what they represent (E.g. Game, Pipeline or Variant)
- Custom information be displayed for an image.

In this post I will take you through how thumbsup was used to achieve it.

# Custom Theme

thumbsup themes can specify their own set of options which can be used to customise the final output. 

The options are specified in a settings JSON file, which you can configure thumbsup to point to using the setting `theme-settings: "path/to/settings.json"`.

The Faze gallery uses the [theme-flow](https://github.com/thumbsup/theme-flow), which has a limited number of customisable options.

In order to render additional content, I needed to create a custom thumbsup theme and used the theme-flow as a base.

## Running thumbsup Locally

The [installation options](https://thumbsup.github.io/docs/2-installation/options/) section of the thumbsup documentation lists two methods for running:
- As a npm package
- As a Docker container

The npm package boasts faster speeds, however I found the docker container to be sufficient and it didn't require me to install all the other dependencies separately.

I used the following command to run thumbs up with docker:

`docker run -t -v "%~dp0:/work" thumbsupgallery/thumbsup /bin/sh -c "cd /work/ && thumbsup --config gallery-config.json"`

### **`-v "%~dp0:/work"`**

Maps the current directory to the /work directory inside the container. `%~dp0` is a Windows batch file command to get the current directory - you may have to use a different one depending on your environment.

### **`thumbsupgallery/thumbsup`**

The docker image.

### **`/bin/sh -c "cd /work/ && thumbsup --config gallery-config.json"`**

Runs a shell process in the docker container, along with the command issued to it. The gallery-config.json exists in the current directory, which we find within the container by navigating to the '/work' directory.

From here you can play around with the theme and read more about it in the [thumbsup documentation](https://thumbsup.github.io/docs/4-themes/create/).

# Custom Metadata in thumbsup

 The [thumbsup data model](https://thumbsup.github.io/docs/4-themes/data-model/) only supports a predefined set of metadata properties for an image. 
 
 ```
{
  date: Date,
  caption: String,
  keywords: [String],
  video: Boolean,
  animated: Boolean,
  rating: Number,
  favourite: Boolean,
  width: Number,
  height: Number,
  exif: Object
}
 ```
 
 `exif` contains additional information provided by a camera - E.g. location the image was taken. 
 
It is possible to hijack the existing metadata properties to hold custom information (you can use the same [exiftool](https://exiftool.org/)). However, there is no support for album-level metadata. 

In this case we can leverage the theme settings JSON file to provide image, album and any other custom metadata.

```
{
  "itemMetadata": {
    "Pawn Depth Painter 3.png": {
      "config": {
        "TreeSize": 3,
        "ImageSize": 500,
        "BorderProportion": 0.0,
        "MaxDepth": 6,
        "Piece": "Pawn",
        "OnlySafeMoves": false
      },
      "fileId": "Pawn Depth Painter 3.png",
      "album": "Piece Board",
      "pipelineId": "PB Depth",
      "variation": "Pawn",
      "depth": 6
    },
    ...
  },
  "pipelineMetadata": {
    "PB Depth": {
      "id": "PB Depth",
      "dataId": null,
      "relativeCodePath": "Visualisations/PieceBoards/PieceBoardImagePipeline.cs"
    },
    ...
  }
  ...
}
```

The theme settings above show how image and pipeline metadata are stored for the Faze gallery.

Within the custom theme, these settings can be referenced by using `@root.settings.itemMetadata` or `@root.settings.pipelineMetadata`. 

Metadata for specific files or albums (i.e pipeline albums) can be fetched by using the filename or album name properties. E.g. `@root.settings.itemMetadata[filename]`

These commands can be wrapped up in a helper (item_metadata.js):

```
module.exports = (settings, filename) => {
    var info = settings.itemMetadata[filename];
    if (!info) return {};

    return info;
};
```

and called using .hbs (in the context of a file):

```
{{'{{'}}item_metadata @root.settings filename}}
```

See further documentation [here](https://thumbsup.github.io/docs/4-themes/create/).

# Hacking Image Information

The final complication was using the existing image panel for displaying the custom metadata.

![faze gallery image info](/assets/images/faze-gallery-img-info-edit.png)

The thumbsup flow theme uses the library [lightgallery](https://www.lightgalleryjs.com/) with the lg-exif.js plugin. 

I ended up editing the lg-exif.min.js file, found in public/lightgallery/js/lg-exif.min.js (unminifed):

```
...

(n.prototype.renderExif = function () {
    (this.html = this.exifHtml), ...;
}),
(n.prototype.update = function () {
    let e = this.core.$items.get(this.core.index),
        i = e.dataset.filename;
        this.exifHtml = $('.itemInfoHtml', e).html();
    if ((this.renderExif(), "undefined" != typeof USE_DATA_ATTRIBUTES)) {
        let e = "Image Information";
        (e += i ? " (Data)" : " (EXIF)"), t(".lg-exif h3").html(e);
    }
}),

...
```

Creating a new property exifHtml and assigning it the content of the element with 'itemInfoHtml' css class.

I then added the following div to thumbnail.hbs:

```
<div class="albumList" ...>
...

    <div class="itemInfoHtml" style="display:none;">
        <div class="itemInfo">
            {{'{{'}}>itemInfo (item_metadata @root.settings filename)}}
        </div>
    </div>
</div>
...
```

### `{{'{{'}}>itemInfo (item_metadata @root.settings filename)}}`

Renders a partial with the file's metadata as the context.

You can see the full code [here](https://github.com/b-faze/faze/tree/master/src/pages/theme).