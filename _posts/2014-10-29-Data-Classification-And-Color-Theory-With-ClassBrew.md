---
layout: post
title:  "Data Classification and Color Theory with ClassyBrew"
author: Joshua Tanner
description: Statistical classification and color theory with our own open source library.
image: assets/images/color-leaf.jpg
---
***Make your map beautiful and statistically relevant***

[ClassyBrew][classybrew] on GitHub

When creating a choropleth map, there are generally two key questions.

1. How do I classify the data? (*break up the data*)
2. What color scheme should be used?

**Data Classification**

If you ever classified data using GIS desktop software or a capable API like [Esri's Javascript API][breaksesri], you've been lucky to have this already solved for you.  A quick selection of an appropriate data classification, such as: equal area, natural breaks, quantile, standard deviation, etc. will provide you with breaks in your data.  Once your data is classified you can define how you want to symbolize your data by color, or select any available color palettes.  You still need to answer the above questions, but all the tools are ready at your disposal.

My favorite data classification method is called the Jenks Natural Breaks method.  It calculates inherent places to break up your data based on the number of classes you want.  These statistically related numeric ranges are perfect for a when you want a meaningful breakup of you data.  If you are looking for an amazing writeup of the Jenks Natural Breaks, and an algorithm in Python, read the following blog post by Daniel Lewis:

[Jenks' Natural Breaks Algorithm in Python][jenks]

I started to break down the algorithim into javascript, until I blindly stumbled [geostats][geostats], a comprehensive open source library provided by Simon Georget.  Geostats is a standalone library that provides methods to classify your data using various different methodoligies, including Jenks Natural Breaks.  If you are looking for a multipurpose tool, I would defintely consider incorporating it into your future projects.

**Color Theory**

What colors best represent your newly classified data?  More often than most, you will be classifying your data because it follows some sequential pattern.  [Cynthia Brewer][cynthia] has spend a good chunk of her career figuring this out for you.  Thank you [Cynthia][cynthia]!  Her research and cartographical background created [COLORBREWER], a visual decision making tool for coloring your map to represent your data.  

**The Problem**

I recently created a map in [Leaflet][leaflet] and needed to classify my data (geojson) and assign a color value to each sequential range.  Since there is no inherent ability to render data based on associated values, you will need to do this from scratch.  I found a handy [chropleth example][leaflet-example] on [Leaflet][leaflet] that did just that.

This was accomplished by manually defining your breaks and returning , again a manually assigned color:

{% highlight javascript %}
function getColor(d) {
    return d > 1000 ? '#800026' :
           d > 500  ? '#BD0026' :
           d > 200  ? '#E31A1C' :
           d > 100  ? '#FC4E2A' :
           d > 50   ? '#FD8D3C' :
           d > 20   ? '#FEB24C' :
           d > 10   ? '#FED976' :
                      '#FFEDA0';
}
{% endhighlight %}

I'm sure you can see a problem with this method.  If your data changes or you want a new color palette, you have to manually change these values.

What I needed was an automatic way to do this.  The pseudocode looked like:

{% highlight text %}
Define my data series
Define the desired number of classes
Define the color palette based on ColorBrewer
Classify your data using Jenks Method
function getColor(number) {
	return appropriate color
}
{% endhighlight %}

**The Solution**

Using the [Jenks'][jenks] algorithm and sequential [colorbrewer][colorbrewer] palette, I created [classybrew][classybrew].  It does exacly what the above pseudocode defines.

Here is the setup:

{% highlight javascript %}
var brew = new classyBrew();
brew.setSeries([30.01, 14.9, 22.71, 24.96, 7.17, ...]);
brew.setNumClasses(5);
brew.setColorCode("BuGn"); // Oranges, Blues, BuGn, GnBu, ...etc
brew.classify();
{% endhighlight %}

Now we can use the [Leaflet choropleth example][leaflet-example] using the newly automated generator.

{% highlight javascript %}
function getColor(number) {
    return brew.getColorInRange(number);
}
{% endhighlight %}

Everytime we call **getColor**, we will now have our Jenks' classified and ColorBrewer approved color representing the provided number.  

***Yay!!***

**GitHub Repository**

Feel free to use the JavaScript library, provided here:

[ClassyBrew][classybrew] on GitHub

The library contains a simple example of using the classybrew object, as well as a minified version only ***18kb*** in size.

**Considerations**

Thank you to the following individuals for helping make this possible:

[Cynthia Brewer][cynthia] - ColorBrewer

[Simon Georget][geostats] - GeoStats

[Daniel J. Lewis][jenks] - Jenks Natural Breaks Description

[leaflet]: http://leafletjs.com/
[cynthia]: http://www.personal.psu.edu/cab38/
[leaflet-example]: http://leafletjs.com/examples/choropleth.html
[colorbrewer]: https://github.com/axismaps/colorbrewer
[geostats]: https://github.com/simogeo/geostats
[classybrew]: https://github.com/tannerjt/classybrew
[breaksesri]: https://developers.arcgis.com/javascript/jsapi/classbreaksrenderer-amd.html
[jenks]: http://danieljlewis.org/2010/06/07/jenks-natural-breaks-algorithm-in-python/
