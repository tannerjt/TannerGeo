---
layout: post
title:  "Spatial Version Control with GeoGig"
author: Joshua Tanner
description: A basic overview of GeoGig.
image: assets/images/leaf.jpg
---

There are a lot of concepts that are core to the programming and development communities that have yet to manifest themselves (to the masses) in the GIS world.  One of them is version control.

#### Basics

If you have ever used git or svn, you know of it's usefulness in maintaining a movable history of the 'state' of your code or other files.  If you have never heard of version control, it is simply a system that keeps track of changes and allows you to 'roll-back' if needed.  You can visualize these changes (additions and subtractions) and provide readable comments with every *commit*.  It also provides methods to resolve *conflicts* if multiple changes contradict each other.

I one had a colleague who explained version control very elegantly.

> 'copy and paste is the one true evil.' - colleague/friend

To be honest, I use this all the time when I try to explain the usefulness of version control.  When you copy and paste, you now have two completely separate datasets that know nothing about each other.  When one is updated, the other falls behind and trying to keep them smartly in sync can become futile.  With version control, we can checkout a copy, make changes, and commit them back to the remote repository.  If updates need to be rolled back, we can easily revert them.  

#### Why GIS Needs It
Technically, Esri has been using versioning in enterprise geodatabase solutions for some time now.  It allows concurrent editing of a geodatabase, conflict management, and the ability to revert to a previous state. However, it requires complex setup and the need to be in the Esri environment.

I'll be the first to admit my working project folders usually have about 10+ versions of the same dataset.  GIS analysts are always worried about needing to revert their edits or make a backup in the event of bad edits.  When I worked for the City of Charleston, most of my work was with the planning department.  My 'final changes' were never actually final and I would gulp when I heard the phrase "let's go back to the version we used for the last city council meeting...".  If your like me, you have a folder that looks like this:

    bike_route_01152014.shp
	bike_route_01202014.shp
	bike_route_proposal1.shp
	bike_route_final.shp
	bike_route_final2.shp
	bike_route_final_for_real.shp

This is a one simple example of why we need version control... badly.

Another reason is making sure your team is always working with the latest data.  Even if everybody is working on a local clone of a spatial dataset, they could run a quick *pull* command at anytime and retrieve the most up to date data.

####Why Not Just Use Git?
Current version control systems like git work great for text.  The flawlessly track what text was added or removed, and this provides the information we need to understand changes.  Spatial data is binary, and although current systems will track the changes, it doesn't provide us with much insight into what changed.  If we wanted to understand what changes were made in our latest commit, we would only see that a bunch of 1's and 0's changed... not much help.  What we need to see is how many features were added, modified, or removed and on which dataset.

####Geogig Background
[GeoGig](http://geogig.org/) (previouly GeoGit) is a distributed version control system (DVCS) for geospatial data.  The 'distributed' term is important and follows closer to the architecture of Git than SVN (google git vs svn for more info).  The software is open source, and currently maintained by the [Eclipse Foundation](https://www.locationtech.org/projects/technology.geogig) and formerly maintained by BoundlessGeo.  A disclaimer on the GeoGig site states that is in 'development' and considered unstable.  In my short experience with the software, it worked great, but I would not yet use this in a full production environment.

I would love to see [GeoGig](http://geogig.org/) developed to be stable enough to use as a reliable tool for spatial data version control.  

####GeoGig Sample Workflow
I would recommend following the [Getting Started](http://geogig.org/#install) guide for downloading and getting up and running with GeoGig.  The following workflow will demonstrate the following scenario:

*You are working in a team environment editing bike routes.  The shapefile needs to be accessible on your local computer for when you are editing in the field and taking work home.  You also need to push your changes to a shared network drive and pull down changes that other editors are doing.  I will assume you are working in a windows environment.*

1. On your (S:) shared drive, initialize a GeoGig repository.  This is where your master repo will live, and will act like a library for users to check out and back in their edits.


        S:master_routes> geogig init

2. This will create a .geogig directory that will track all of your changes, among other things.  You will need to import your bike_routes shapefile which can be anywhere.  For this example I will assume it's in the same folder as your newly created GeoGig repository.  Think of this step as putting it into a warehouse.

        S:master_routes> geogig shp import ./bike_routes.shp

3. Now that it is in the warehouse, you need add it to a *loading dock* - like location before committing it entirely.  Only changes will be added, and in our initial case will be all of our existing features in bike_routes.  

		S:master_routes> geogig add

4. This is the equivalent of 'add all changes' to our loading dock to be shipped from the warehouse.  Next we will commit our added changes, which is like putting it on the truck.

		S:master_routes> geogig commit -m "Add initial commit of bike routes"

5. The -m trigger allows us to add a message with our commit.  This is helpful to make each commit human readable and easier to use.  Now we want all of our users/editors of this repository to pull down a working repo to their local machines.  For this, our (S:) shared drive will act as a remote repository.  From our (C:) local drive, we can add our connection:

		C:local_bike_routes> geogig clone S:/bike_routes/

6. The clone command clones our master repository and automatically creates a remote tracking branch.  In essence, it is a clone that knows how to access it's master.  It can pull down any changes done to, or push changes up to the master.  Our master remote branch is automatically named *origin*.  If you look in your directory you might be confused.  There bike_routes shapefile is nowhere to be found, only a .geogig directory?!?! With GeoGig, everything is stored in the database within the .geogig repository.  If you want to export the shapefile to work on, you will have to do this implicitly:

		C:local_bike_routes> geogig shp export HEAD:bike_routes ./bike_routes.shp

7. This exports out a shapefile for you to work on.  You can also export the data out in various different formats (geojson, postgres, spatialite, etc), which can be a great tool in itself.  Now that you have a local shapefile to edit, you can make the changes you need in whatever software you are using (qgis, arcgis, etc).  Once you are ready to push the changes up to the master repository, you will have to use a couple command line steps.  You will need to: import the shapefile into your local repo, stage your edits (loading dock), and commit them (put them on the truck).  Once everything is committed in your local geogig repository, you will push the changes to your remote (origin) branch.  

        C:local_bike_routes> geogig shp import ./bike_routes.shp -d bike_routes
        C:local_bike_routes> geogig add
		C:local_bike_routes> geogig commit -m "Add new rutledge ave bike route"
		C:local_bike_routes> geogig push origin master

8. We have now updated not only our own bike_routes feature, but also the feature on our S: drive.  All our other editors working on their own local repositories can pull in these changes, and view your comments in the log.

		C:local_bike_routes> geogig pull origin master
		C:local_bike_routes> geogig log

####Discussion

I tried this workflow on a dataset that I need to continually update and share (even through I warned you not to use it in production).  It worked without error and organized my process ten-fold.  Below are things I liked, and some I didn't.

Positive:

+ I liked that it inherits from git.  Most of the commands are the same, although the workflow is slightly different.  This makes the learning curve very minimal for those already familiar with git.
+ The distributed architecture works well for geospatial data
+ Being able to export to different formats is great.
+ The documentation is clear and concise.
+ Fast.

Negative:

+ No support for file geodatabase.  This means you will have to export to a shapefile to use this utility.
+ I don't like that I have to import my data before I add and commit it.  Importing and then adding seems redundant and complicates the process.  This part is different from git.
+ This may be lazy, but typing git is easier than typing geogig.  I typed it a lot in my workflow and it would be easier if it defaulted to gig or something simple.  Again, I could just be lazy and this could probably be changed easily.

It would be also nice to see a GitHub-like solution for publishing and collaborating on spatial data.  Overall I liked it and will be using this a lot more in my current work.  I encourage both the use and further development of GeoGig to create a stable solution for geospatial data version control.

####Extras

GeoGig has many extras that I didn't cover in this article that I think others should check out.  Here are a few:

+ [Importing OSM Data into GeoGig](http://geogig.org/docs/interaction/osm.html)
+ [The GeoGig Console](http://geogig.org/docs/interaction/console.html)
+ [Using GeoGig with GeoServer](http://geogig.org/docs/interaction/geoserver_ui.html)
+ [Build in Web Server and API](http://geogig.org/docs/interaction/web-api.html)

I especially liked the GeoGig console.  This allowed me to more easily write a batch script that exported the latest shapefiles our of my master repository and output a log of all recent commits.

##

Other Links:
[James Fee Article on GIS Version Control](http://www.spatiallyadjusted.com/gis-version-control/)
