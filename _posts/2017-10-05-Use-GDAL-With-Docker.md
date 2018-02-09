---
layout: post
title:  "Use GDAL With Docker"
author: Joshua Tanner
description: A brief understanding of Docker for running GDAL operations in a container.
image: assets/images/container.jpg
---

In this post, we're going to convert a geojson file to a shapefile using a prebuilt Docker image that already has GDAL installed.  

[GDAL](http://www.gdal.org/) is the library that most geospatial professionals use for working with geospatial data outside of traditional GUI desktop applications.  One of the most used features (at least for me) is the ability to convert spatial file types with [ogr2ogr](http://www.gdal.org/ogr2ogr.html).

Now, you could install GDAL on your own machine and configure everything to work via command line, but latelty I've been seeing the benifits of using isolated environments for specific workflows.

One way to have a isolated environment for working with GDAL is to use [Docker](https://www.docker.com/).

[Docker](https://www.docker.com/) is like having a virtual machine that has all of its resource isolated from its' surroundings.  However, unlike a virtual machine, Docker containers virtualize the host's operating system and not the hardware.  A [container image](https://www.docker.com/what-container) is a lightweight, stand-alone, executable package of a piece of software that includes everything needed to run it: code, runtime, system tools, system libraries, settings.

So, why mess with the setup of our own pristine machine when we can tinker and break an ephemeral enviroment without a care in the world?  Docker is like having an unlimited supply of new computers with different operating systems and setups.  You can pull from predefined images, or make your own from scratch that can inherit from existing images.

Enough with trying to explain Docker.  It's awesome.  It works.  It will make you a better programmer.  It will save you the headache of breaking your local computer when installing and trying new things.

## Step 1: Install Docker

Assuming you don't already have it installed, you can get it work [Mac](https://www.docker.com/docker-mac), or [Windows 10](https://www.docker.com/docker-windows) on their official website.  The install should be seamless if you have an updated mac or windows machine.  Older windows machines (<Windows 10) will need to go the [Docker Toolbox](https://www.docker.com/products/docker-toolbox) route, which requires more initial setup.

## Step 2:  Pull the GDAL Image

I'm using this [image](https://hub.docker.com/r/geodata/gdal/) from [geodata](https://github.com/geo-data), which is an Ubuntu (trusty) OS that already has GDAL installed and configured.

From the command line, type:

```bash
$ docker pull geodata/gdal
```

This will download the image and allow us to build a container from this image.  Once complete (may take a couple minutes), you can verify it is available by listing your docker images:

```bash
$ docker image ls
```

### Step 3:  Prepare Data Volume

[Volumes](https://docs.docker.com/engine/admin/volumes/volumes/) in Docker are a way to share and persist data from your containers.  Containers are inherently ephemeral, and data goes *poof* once you remove a container if you are not storing data to a volume.  Volumes also allow you to share data between containers and between your local machine and a running container.

Create a new directory:

```bash
$ cd ~/documents/
$ mkdir mydata && cd $_
```
Put some geojson into it.  You can copy this single point geojson if you want:

```json
{"type":"FeatureCollection","features":[{"type":"Feature","properties":{},"geometry":{"type":"Point","coordinates":[-123.19158554077147,44.851122784247245]}}]}
```

If using the *geojson* from above, echo the entire contents into a new file named **point.geojson**

```bash
# ~/documents/mydata/
$ echo '{"type":"FeatureCollection","features":[{"type":"Feature","properties":{},"geometry":{"type":"Point","coordinates":[-123.19158554077147,44.851122784247245]}}]}' > point.geojson
```

We will need the path to this directory, so lets print the current working directory and copy it to the clipboard:

```bash
$ pwd
# /Users/tannergeo/Documents/mydata
```

## Run the Container with GDAL in Interactive Mode

We are going to do a lot in one command, so let me explain after:

```bash
$ docker run -it --rm -v /Users/tannergeo/Documents/mydata:/root/mydata/ geodata/gdal /bin/bash
```

Here is what is going on.  We're building a container from the image and interacting with it directly.  We're also mounting our *mydata* folder into the container so that the files are shared between them.  For reference:

+ `-it` - interactive ttyl
+ `--rm` - remove the container after we exit it
+ `-v` - the volume to mount */from/host/* : */on/container*

## Run ogr2ogr on Container

Now that we're on the Unbuntu machine that has GDAL already setup, we can run our ogr2ogr command.  First, lets verify volume did indeed mount:

```bash
# on container
# should looke like
# root@someid93889s: ~/
$ cd /root/
$ ls
mydata
$ cd mydata && ls
point.geojson
```

Awesome, now anything in this folder will be available both by the container and your local machine.  Let's convert this file to a shiny new shapefile!

```bash
#/root/mydata/
$ ogr2ogr -f "ESRI Shapefile" point_shp.shp "point.geojson"
$ ls
point.geojson point_shp.dbf point_shp.prj point_shp.prj point_shp.shx
```

## Exit Container and See Shapefile on Local Machine

Let's now exit the contianer and see how the output was shared on our local machine.  Since we flagged `--rm`, the container will be removed, but the data will persist.

*type **exit** to go back to your terminal*

```bash
#~/documents/mydata/
$ root@someid93889s:exit
$ ls
point.geojson point_shp.dbf point_shp.prj point_shp.prj point_shp.shx
```

Without messing with our own machine, we were able to convert a geospatial geojson file to a shapefile on an already configured Ubuntu machine with GDAL.  These isolated environments are really a great utility, especially if you're prone to breaking your computer when messing with your own configurations (like I do often).  I might never install new libraries locally again!

## Conclusion

[Docker](https://www.docker.com/) has a variety of different uses, not just running commands in an isolated environment.  It can be used to replicate development environments across an entire team.  Docker also helps to automatically scale apps in cloud environments.

There are many other great tutorials on Docker if you want to continue learning, since this was a short and sweet tutorial lacking many of the great features that Docker has to offer.  Here are a few:

1. [Dockers Official Getting Started Tutorial](https://docs.docker.com/get-started/)
2. [Docker Lab Tutorials](https://github.com/docker/labs)
