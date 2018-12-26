---
layout: post
title:  "Creating a Thumbnail Builder"
author: Joshua Tanner
description: Using HTML5 Canvas to Create an ArcGIS Online Thumbnail Builder
image: assets/images/paints.jpg
---

Nevermind this post, take me directly to the [ArcGIS Thumbnail Builder](https://dev.navigator.oregon.gov/agol/thumbnailbuilder/index.html?background=./img/gallery/boat.jpg&logo=./img/gallery/osmb.png&titleColor=111,166,166,0.7&title=Great%20White%20Shark%20Migratory%20Patterns&category=Feature%20Service&sidebarColor=166,17,3,0.7)!

I'm a pretty outspoken advocate of ArcGIS Online.  

*Disclaimer: I'm the administrator for an account with roughly 2,000 users for the State of Oregon.  However, I don't think this influences my support for this product.*  

I've been around long enough where I've developed dozens of GIS applications for every level of
government.  Most workflows are redundant and can take days / weeks / months to develop.  Managers and
influencers have an expectation for near instant results. Most staff (planners, biologists, analysts, etc)
are now expected to wear multiple hats, with little to no GIS or web development experience.  ArcGIS Online enables these staff to create their own web maps and applications with a simple point and click interface.  This can reduce the burden on the GIS staff, and in some cases completely transform how GIS knowledge is distributed within an organization.

The downside?  Creating things quickly means skimping on the details.  This can result in
garbage for metadata and missing thumbnails.

It's important to create a quality thumbnail for items in ArcGIS Online.  Not only can it provide some basic details (type of item, title), but it also can
infer the quality of the item itself.  A crappy or missing thumbnail sends the message that the item
is of sub-par quality and shouldn't be trusted.

Esri has a great blog post from Ben Szukalski titled ['Put your best thumbnail forward'](https://www.esri.com/arcgis-blog/products/arcgis-online/uncategorized/put-your-best-thumbnail-forward/) which outlines some of the important elements to consider when creating a good thumbnail.  Ben recommends using a template (Photoshop or similar) for creating thumbnails.  I've found this to be cumbersome to distribute and keep updated within
a large group of users.

The best solution would be a web tool that would allow users to quickly create these thumbnails online.  Scott Moore (@Esri) developed a great tool that I used for years, [which is hosted on Github](https://github.com/Esri/thumbnail-builder-js).  Although super useful, it relied on an ArcGIS Server geoprocessing service and sadly stopped working for me when we migrated servers.  Since layering text and images is something that could easily be done within an HTML5 Canvas, I figured I would port the tool over myself.

The [HTML5 Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) is like an artists canvas, where shapes and text can be drawn.  If you want to learn how to work with the Canvas API, I highly
recommend [Chris Courses YouTube series](https://www.youtube.com/watch?v=EO6OkltgudE).  I used it myself to refresh my memory.

This was also my first time using [Materialize](https://materializecss.com/), as CSS library based on Google's Material Design.  I was highly pleased by the final product.  Simple and functional appearance with non-complex and easy to read code.  

![Thumbnail Builder Screenshot]({{site.url}}/assets/images/blog/screenshots/thumbnail_builder_ss.png)

I even added a gallery with fun examples:

![Thumbnail Builder Screenshot]({{site.url}}/assets/images/blog/screenshots/tb_gallery.jpg)

The tool only took 5 or 6 hours to put together.  After completing the tool, I solicited feedback on Twitter and quickly received some great support from other people that liked the tool.  To my disbelief, [Jim Herries](https://twitter.com/jherries) from Esri even made use of it for one of his web apps, titled [The 51 Fastest Growing Counties in terms of GDP](https://nation.maps.arcgis.com/home/item.html?id=c9043022a43e44eaaca078553c3abbc2)!  Thanks [Jim](https://twitter.com/jherries)!

![Tweet from Esri]({{site.url}}/assets/images/blog/screenshots/tb_tweet.jpg)

The tool is public and free to use.  It is available at the following URL: [https://dev.navigator.oregon.gov/agol/thumbnailbuilder/index.html](https://dev.navigator.oregon.gov/agol/thumbnailbuilder/index.html).

Let me know what you think!
