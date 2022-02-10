---
layout: post
title: Making a model of an island using plywood
date: '2022-02-10 20:11'
categories:
  - en
tags:
  - 3d-printing
  - laser cutting
comments: true
lang: en
ref: 3d-relief-svg-preparation
---

A month ago, I've gotten a great [Snapmaker 2.0 A150 3-in-1](https://snapmaker.com/snapmaker-2/specs) that can print with plastic as a 3D printer, cut and engrave with a laser, and has a milling module that can also be used to make three-dimensional models or make circuit boards.

The package included a kilogram spool of black PLA thread for 3D printing, 2 pieces of plywood measuring 150 * 150 mm to test the capabilities of the laser and an acrylic plate to test milling.

First, I put the module for 3D printing and was impressed. Impressed by the process itself, how the finished product appeared layer by layer. Having no previous experience, I managed to print several items from the first attempt (almost the first).

<center>
  <iframe width="560" height="315" src="https://www.youtube.com/embed/cX_YODc6Eek" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

The next step was to test the laser module. I didn't want to cut out a simple test box that is available in the Luban project library. I wanted something interesting.

## In search of inspiration

On the advice of [Fedor Gontsa](https://www.behance.net/gontsa), I decided to make a relief model by cutting out parts of the two pieces of test plywood that came with the rig. I have already seen several examples of relief models created from layers of plywood in the [post of Eric Gunderson](https://blog.mapbox.com/3d-laser-printing-maps-in-glowforge-54007f9736e1), founder of Mapbox, a few years ago and now it's time to do something similar myself.

![Eric blogpost](/images/2022/02/eric-blogpost.png)

To do this, I needed to find data on a digital terrain model of an area, for example, raw data that requires processing by specialized geospatial data processing systems, such as [QGIS](https://qgis.org/en/site/), or ready-to-use isolines data.

![Mapbox Outdoors](/images/2022/02/mapbox-outdoors.png)
<sub>_[Mapbox Outdoors](https://api.mapbox.com/styles/v1/mapbox/outdoors-v11.html?title=true&accesstoken=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4M29iazA2Z2gycXA4N2pmbDZmangifQ.-gvE53SD2WrJ6tFX7QHmA#14.46/27.72792/-17.95366/0/1) - There are isolines on one of the map styles from Mapbox_</sub>

As an option, you can go to [Mapbox Studio](http://studio.mapbox.com) and take only the isolines, using the data set Mapbox Terrain v2 (<https://docs.mapbox.com/data/tilesets/reference/mapbox-terrain-v2/>), which contains exactly what we need. This is easy to do given the availability of detailed documentation with code examples and alike.


## Getting isolines

However, I came across another tool that already had everything I was going to do myself. <https://contours.axismaps.com/#12/27.7434/-18.0547>

![Contour Axismaps Preparation](/images/2022/02/countour-axismaps-preparation.png)


It remained only to find an area with a clear relief. My choice fell on the Canary Islands, the island of Hierro (or as it is also called Isla del Meridiano). The height of the highest point of the island is 1501 m, it has steep slopes and expressive bays - just what I need.

Choosing a step of 500 feet, I got 10 layers that I needed to cut from two pieces of plywood.

I had only two pieces of plywood, but how to make 10 layers out of them? The solution was quite simple - to cut layers of 500, 1500, 2500, 3500 and 4500 feet from one plywood, and 1000, 2000, 3000 and 4000 feet from the another. I would have layers that overlap - just what I needed, I had only two pieces of plywood and I had to be frugal and get as little leftovers as possible.

![Contour Axismaps Download](/images/2022/02/countour-axismaps-download.png)

## Processing *.svg, preparation for cutting

I saved the isolines to the `*.svg` file and tried to open it directly in Luban. There I had the first problem. The thing was, I was hoping that Luban would allow me to extract each line from the `*.svg` file separately and assign it the appropriate instructions for cutting and engraving. On one piece of plywood I was going to cut isolines 500, 1500, 2500, 3500 and 4500 feet and engrave isolines 1000, 2000, 3000 and 4000 feet, on the other - vice versa. It turned out that in Luban you can set parameters only for all lines from the file at once - it does not have the ability to select individual lines from the file for cutting or engraving.

![Contourlines Inkscape Original](/images/2022/02/contourlines-inkscape-original.png)

Here Inkscape helped me. I divided the file into three parts: in the first part I put a coastline isoline of 0 feet; in the second, isolines of 500, 1500, 2500, 3500 and 4500 feet; in the third, isolines of 1000, 2000, 3000 and 4000 feet. In addition, I've shrinked the files so that their side was 150 mm, the side of my pieces of plywood. It was also useful to simplify the lines, it would relieve my device a little from cutting too small details and reduce the file size from 400 kB to 17-25 kB each. Remember to save files in **Plain SVG (*.svg)** format.

Shoreline | Odd isolines | Even isolines
-- | -- | --
![Coast line](/images/2022/02/coast-line.svg) | ![odd contour lines](/images/2022/02/odd-cotour-lines.svg) | ![even contour lines](/images/2022/02/even-contour-lines.svg)
[shoreline (*.svg)](/images/2022/02/coast-line.svg) | [odd isolines (*.svg)](/images/2022/02/odd-cotour-lines.svg) | [even isolines (*.svg)](/images/2022/02/even-contour-lines.svg)

You can now open these files in Luban and assign settings to each of them for cutting or engraving.

Reduce the size for all files to 140 mm in Luban.

![Luban Resize](/images/2022/02/luban-resize.png)

For the first piece of plywood we will take all three files: the file with the shoreline&nbsp;- we will engrave it, the file with isolines 500, 1500, 2500, 3500 and 4500 feet, which we will also engrave (they will serve as a guide for further assembly of our model) and a file with isolines of 1000, 2000, 3000 and 4000 feet&nbsp;- on which we will cut.

![Luban Toolpath](/images/2022/02/luban-toolpath.png)

For the second piece of plywood we will take only files with isolines (without shoreline) and those lines that we engraved on the first plywood will now be cut, and those on which we cut on the first plywood&nbsp;- we will engrave.

![Luban Toolpath 2](/images/2022/02/luban-toolpath-2.png)

It turns out that the distance between the isolines for cutting is 1000 feet (in height) on both plywood.

## Assembling the model of the island

The piece of plywood on which we engraved our shoreline will be the basis for assembly. On it we will glue the largest element from the second piece of plywood, then the next element from the first piece of plywood we will glue on top of the element from the second… and so step by step we will assemble the model of the terrain.

You can start assembling from the lowest layer and move up, or from the top layer and move down respectively.

Because my laser didn't cut through a piece of plywood in all places and I had to separate the parts with a model knife, I decided to assemble the island from the top to the shoreline, removing the smallest elements first and applying them to the larger ones.

<center>
  <iframe width="560" height="315" src="https://www.youtube.com/embed/P1e5ME25k6Y" title="YouTube video player"  frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

In the process of assembling, I thought it would be good to add the name of the island and its coordinates to the base, the largest detail on which I laser engraved the shoreline. After that I glued the island to the base.

<center>
<blockquote class="twitter-tweet"><p lang="uk" dir="ltr">Look what I have done!<br><br>Material: 2 pieces of plywood 1,5mm 150 × 150mm<br>Isolines: Elevation tiles by Mapzen<br>Processing: Inkscape<br>Cutting, engraving: <a href="https://twitter.com/hashtag/snapmaker?src=hash&amp;ref_src=twsrc%5Etfw">#snapmaker</a> 2.0 A150 <br><br>Assembling: by myself. <a href="https://t.co/2TAbXT71jH">pic.twitter.com/2TAbXT71jH</a></p>&mdash; Andrey Golovin (@andygol_) <a href="https://twitter.com/andygol_/status/1489968188636999683?ref_src=twsrc%5Etfw">February 5, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</center>
