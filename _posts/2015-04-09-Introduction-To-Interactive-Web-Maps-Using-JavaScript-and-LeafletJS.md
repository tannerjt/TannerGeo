---
layout: post
title:  "Introduction to Interactive Web Maps Using JavaScript and LeafletJS"
author: Kristin Tanner
description: Get up and running with a great web mapping solution.
image: assets/images/map.jpg
---

![test image]({{ site.url | absolute_path}}/assets/images/map.jpg)

This is an excellent beginning web application for anyone new to JavaScript and/or [LeafletJS](http://leafletjs.com/).  The tutorial was developed by MIT and is easy to understand.  As someone who is grateful to all professors/peers who have shared their knowledge and experience with me over the years, I like to contribute when able.  I have included some modifications for aspiring web developers below that address publishing your application via [GitHub Pages](https://pages.github.com/) as well as [Bower](http://bower.io/), a package manager that makes including frameworks, libraries, and utilities in your projects more convenient for developers.  Altogether, this makes for an excellent start to learning about general web development as well as creating interactive web maps.


The full MIT tutorial can be found [here](http://duspviz.mit.edu/leaflet-js/).

View my application [here](http://tannerkj.github.io/MIT-campus-coffee/).

###### Adjustments I made when running through this tutorial:

###### Step 1.b

I did not serve locally using Python.  I created a [GitHub repo](https://github.com/tannerkj/MIT-campus-coffee) and viewed changes locally using *http://localhost/~username/projectname/* before pushing to GitHub.  It is a good standard practice to follow.  Also, you then have the ability to publish your webpages using GitHub Pages.  

###### Step 1.d

I used [Bower](http://bower.io/), as mentioned above, to add Leaflet to my project.  It would be prudent to become familiar with Bower and its dependencies since many common packages can be easily installed with its use.  Dependencies for Bower include [Node and npm](https://nodejs.org/) and [Git](http://git-scm.com/).  

Using command line, inside my working directory, I entered the following:

	bower install leaflet

You will notice a new folder in your working directory called *bower_components* that contains the necessary Leaflet files for your project.  Therefore, the Leaflet CSS link in the *head* section of your index.html would be referenced as:

	<link rel="stylesheet" href="./bower_components/leaflet/dist/leaflet.css" />

and the Leaflet JavaScript link at the bottom of the *body* section would be referenced as:

	<script src="./bower_components/leaflet/dist/leaflet.js"></script>

View my application [here](http://tannerkj.github.io/MIT-campus-coffee/).

View my source code [here](https://github.com/tannerkj/MIT-campus-coffee).


![MIT-campus-coffee]({{site.url}}/assets/images/blog/screenshots/MIT_campus_coffee.png)		
