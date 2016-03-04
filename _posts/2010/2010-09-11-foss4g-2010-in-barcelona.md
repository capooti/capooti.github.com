---
layout: post
title: "FOSS4G 2010 in Barcelona"
description: "FOSS4G 2010 in Barcelona"
category:
tags: [Uncategorized, GIS, conferences]
---
{% include JB/setup %}

This year I was enough lucky to have the possibility to attend at the <a href="http://2010.foss4g.org">FOSS4G World conference in Barcelona</a> (thanks to <a href="http://ec.europa.eu/dgs/jrc/index.cfm">my company</a> for having send me there!) - that has been run just this week, as many of you may already know.
I just came back home yesterday night and until my impression are still very clear, I were thinking it is a good idea to post them here.
The conference, as you may know, is held every year and is organized by OSGeo. They try to organize it every year in a different continent, and this year was the time for Europe - Spain, so I couldn't miss it :D
It was really an impressive set of discussions related to GIS Open Source Software, attended approximately by 900 partecipants, and it was divided in two parts: Software Workshops (on the 6th and morning of the 7th) and the Conference itself (from the afternoon of the 7th to the 9th).

<strong>Workshops</strong>

There were many workshops proposed (14!) based on many cool GIS tools and frameworks. I could attend this two ones:
<ul>
	<li>Solid web mapping with Python (related to the Pylons, GeoAlchemy, Shapely, GeoJSON and MapFish frameworks for developing GIS web applications with Python) - if you are interested it is already <a href="http://www.mapfish.org/doc/tutorials/python-workshop/">here</a>!;</li>
	<li>GeoNetworks for dummies, or how to setup an SDI in 3 hours (related to using the GeoNetwork web applications for setting and managing a Spatial Data Infrastructure).</li>
</ul>

As a Python developer I was delighted to get finally my hands dirty with some framework I actually don't use for my work (here we are using Django, GeoDjango e GeoExt). Obviously the workshop time (3 hours) was just enough to get an idea about these libraries, but I promised to myself that in my spare time I will give a deeper look, specially at Shapely.

Regarding to GeoNetworks, as some of you may have read on Twitter, I have some sort of allergy to Java Web applications :D so I found it enough boring (but after all metadata management cannot be funny!).
Regarding SDI, I will rather give a deeper look at GeoNode, that is basically a Web 2.0 (groups, comments, tags, etc...) glue for GeoServer and GeoNetwork. It is based on Django (for user security) and Pinax (for the social stuff), and if you are a loyal reader of this blog you may already know that those are two techs I really like.

<strong>Talks</strong>

The number and quality of the conference talks was really impressive, and I could pick up a large amount of information about a vast amount of GIS topics and tools that I can barely  pick in my brain :P (I will need to elaborate all this in the next weeks/months).

<em>Topics</em>

Here is a list of the main topics proposed in the conference talks:

- status of many OGC standards (WMS, WFS, WPS, WCS, SOS, WMTS) implementations;
- status of SDI implementations (at least 8 related to INSPIRE);
- status of various GIS projects within the OSGeo incubation process;
- comparisons and benchmarks between OGC OWS;
- status of OSM;
- many talks related to NoSQL Database, GIS in the clouds and its implementations: here seems that there is quiet a lot of new stuff for neo-geographers to master!

<em>Tools and frameworks</em>

OpenGeo seems to have fixed a toolchain for building a GIS Open Source stack with their <a href="http://opengeo.org/products/suite/">OpenGeo suite</a> based on some OSGeo project. Given that they are highly followed from the GIS community as some very popular developers are working with them and they have a business model that really rocks IMHO, their choices may give some tool/frameworks more spread than other ones.
I hope they will add to the stack MapServer (so possibly it will be possible to use that instead than GeoServer as a GeoNode component).
I also wish that there could be a nice GIS JS solution for JQuery (who I prefer) and not only for the proposed one based on Extjs (GeoExt).
A final note about OpenGeo: I think they should have taken with them more t-shirts to sell, as I couldn't buy the PostGis one as it was already sold out :(

Out of the OpenGeo sphere, I could listen to many talks and tutorials regarding many softwares and libraries I am currently using (GDAL, pyWPS, MapServer, GeoServer, TileCache, GeoWebCache, PostGis, OpenLayers, GeoExt, MapFish, GeoNetwork, QGIS, gvSIG) or that I plan to use in the near future (GRASS, GeoNode, ZOO Project, PostGIS WKTRaster, OSSIM).

Also there were many official and unofficial benchmarks.
The <a href="http://wiki.osgeo.org/wiki/Benchmarking_2010">official one</a> was related to WMS implementations (in a number of 8!) and since what I could see MapServer and Mapnik (a new entry!) seems to have been better than the other ones.
GeoServer seems to have lost some points from last year but in my opinion remains the most complete Web Server implementation for mapping (note that I am writing this even if I prefer MapServer for my work). Also, it has a really nice administrative user interface.
GeoServer already added a WPS implementation, MapServer is going to add a WFS-T implementation possibly with TinyOWS.

This year the conference will be remembered also for other two reasons: the WPS war (I have seen at least 4 implementations: pyWPS, ZOO Project, GeoServer and 52°North), and the massive NoSQL implementations presented (at least 3 ones from what I have seen: GeoCouch DB, Cassandra by SimpleGeo and Neo4J graph).

<strong>Things to explode for future</strong>

As you may imagine, as you read my thoughts, in my head now there are many things to elaborate, but I think in a very short time I would like to give a deeper lock at the following tools, to see if they may add more value to my work:

- GeoNode
- Mapnik
- Shapely
- PostGIS WKT Raster
- NoSQL (I have to figure out to which implementation, but GeoCouch DB seems interesting and a nice reason to start using Erlang at some extent).

<strong>Final notes</strong>

As you may already know, Twitter is an excellent tool for sharing knowledge in a quick way. For following conferences is a perfect way to get live info even if you are not following a certain talk (because you are attending at another one) or you are not at the conference at all.
For people who had not the opportunity to be at the conference, I highly recommend giving a look at the <a href="http://twitter.com/#search?q=%23foss4g">#gfoss twitter hash tag</a>.

At the final day Helena Mitasova, of GRASS fame, was honored with the <a href="http://www.osgeo.org/solkatz">Sol Katz Award for Geospatial Free and Open Source Software 2010</a> for her huge effort in the GIS community since 1990. So congratulations, Helena!

Now the conference is over and I am already waiting a lot for the next one, in Denver in 2011: I hope I will be able to be there and meet all the fantastic people there was in Barcelona and more!
