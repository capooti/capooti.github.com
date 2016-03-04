---
layout: post
title: "PostGIS in action"
description: "PostGIS in action"
category:
tags: [GIS, books, PostGIS, Postgres]
---
{% include JB/setup %}

[PostGIS in action] [1] has landed: finally a book about PostGIS, we were all missing it!  
A software project that has a public visibility since almost 10 years, with a large community and a long series of use cases, finally has its deserved book.

I started using PostGIS in 2006 in a situation where the company I were working for at that time had to cut the cost of licenses and maintenance. Opting for FOSS gave us also the possibility to eliminate long administrative times needed to change any of the requirements in the licenses.  
But above all, we knew to select a technology that could provide more sustainable conditions as only open source software can provide.  
With all this in my mind, I identified in PostGIS (and Postgres) a perfect replacement for the system that was in production at that time (based on Arcsde and MS SQL Server): we would have never switch back.

At that time the software was already excellent, but the documentation was not yet at the current level.  
Though, there was (and it is still there) an indispensable resource that came handy for requiring help: the mailing list.  
And you could count on the fact that at any time of the day, Regina and Leo, the authors of the "PostGIS in Action" book, were there to try to give you an answer or helping you to find one.

Now I have to admit that having this book at that time it would have been a dream!  
But even now, after five years of experience and hacks on the best open source spatial database out there, I can safely say that reading the book has given me (and will give me) many benefits.  
The reading of the material assembled for this book by the authors that I made during these months of the Manning Early Access Program (Manning is a great publishing company that, as Packt and Apress gives a great choices of book titles about FOSS) has been impressive.

And now I am here to say that, if at some extent I was thinking to have an extensive knowledge of this excellent open source spatial database, after reading the book of Regina and Leo I realized how actually I was wrong!  
I found in the book lots of different ideas, and my knowledge of PostGIS has been definitely improved.  
In simple words, I think that the book is an effective reading not only for the experienced developer and GIS analyst, but even people getting started with PostGIS will find many benefits reading it. Any GIS developer or architect willing to do any serious development with PostGIS should have a copy of this book always available around.

### PART I

The book is an impressive block of over 500 pages, but the reading is always very fluent. It is divided into three parts: the first part describes the base concepts, starting from the definition and the features of a spatial database and following with a description and a brief history of the Postgres and PostGIS projects. Finally it provides a first quick and illuminating example (**Chapter 1**).

**Chapter 2** introduces the supported geometry type: here there is an explanation about the aims of the geometry_columns table, about the concept of dimension and spatial reference system (SRID).  
Then there is a discussion of the main functions that interact with the geometry_columns table, particularly AddGeometryColumn, DropGeometryColumn, UpdateGeometrySRID and Populate_Geometry_Columns.  
The chapter concludes with an overview of the different geometries according to the OGC specification, with a parallel of their description and the use of them within PostGIS: useful examples are given here (as indeed throughout the whole course of the book).

The **third chapter** attempts to illustrate the different approaches for the storage of spatial information (heterogeneous and homogeneous geometry columns), with the pros and cons of each approach.  
There is then a description of the concept of inheritance of tables in Postgres and its usefulness.  
The chapter concludes with a real use case in which the various approaches described earlier are applied, with an additional description of database rules and triggers.

**Chapter 4**, together with chapter 5, possibly constitute the core of the book: there is an extended description of the main spatial functions included with PostGIS.  
After a quick overview of geometry constructors functions (ST_GeomFromText, ST_GeomFromEWKT, ST_GeomFromWKB and similiars), the chapter describes the output functions such as ST_AsGML.  
It follows a quick description of the main geometric supported formats (WKT, EWKT, WKB, GML, KML, GeoJSON, .. .) and the geometric types conversion functions.  
It is then explained which functions to use to project a geometry of a spatial reference system to another and then how to verify the validity of a geometry. Following there is an overview of the measurement functions (ST_Length, ST_Length3D, ST_Area, and ST_Perimeter), functions of decomposition (ST_Box2D, ST_Envelope), the ST_X and ST_Y functions to get the coordinates of points, ST_Boundary and functions to obtain the centroid, the point on surface ant the nth point (ST_Centroid, ST_PointOnSurface, ST_PointN).  
Finally the chapter introduces an overview of the functions for collection geometries and multi Breaking down (ST_GeometryN ST_Dump), of the functions of composition (ST_Point and ST_MakePoint for making points, ST_MakePolygon, ST_BuildArea and ST_Polygonize for making polygons, ST_Multi for promoting single to multi geometries) and of the functions for simplification (ST_SnapToGrid, ST_Simplify, and ST_SimplifyPreserveTopology).

In the **fifth chapter**, first the authors introduce the functions of relationship between geometries. Then there is an analysis of functions like ST_Intersects, ST_Contains, ST_Within, ST_Covers, ST_ContainsProperly, ST_Touches, ST_Crosses, ST_Disjoint, ST_Difference, ST_SymDifference.  
There is then a description of nearest-neighbor functions like ST_DWithin and ST_Distance.
Finally the chapter concludes with an overview of the functions of equality (ST_Equals, ST_OrderingEquals), and the use of the intersection matrix with ST_Relate.

In **Chapter 6** spatial reference systems are introduced and extensively described. After some theoretical concepts (particularly useful for those who are beginners with GIS), there is a list of considerations about which system to choose to store data on the database, particularly on consequences if you are choosing the WGS 84 lat lon (EPSG: 4326) - a very common choice - and why if you choose such a system it is sometimes preferable to use the new Geography data type available in PostGIS 1.5.  
The chapter concludes with some considerations on how to determine the spatial reference system (SRID) of an existing spatial dataset, such as shapefile from a .prj file.

The first part of the book ends with **Chapter 7**, which describes the tool (from the CLI and UI) available for import and export data from PostgreSQL (psql, copy, pgAdmin III, pg_dump and pg_restore ) and PostGIS (shp2pgsql, shp2pgsql-gui, pgsql2shp).  
The chapter then introduces the powerful ogr2ogr command (included in GDAL), some QGIS tools for loading data in PostGIS, the osm2pgsql command for loading OpenStreetMap data and the pgsql2shp command to export a PostGIS layer to a shapefile.  
The descriptions are accompanied with brief and useful tutorials very focused on the concepts.

### PART II

Part two of the book consists of only two chapters, which describe techniques for solving spatial problems using the features and functions described in previous chapters (Chapter 8) and how to make successful performance tuning (Chapter 9).  
**Chapter 8** in particular is divided into techniques for the proximity analysis, data tagging and slicing and splicing of LineString and Polygon.  
It closes with a section on how to do a translation, scaling and rotation of a geometry.

**Chapter 9** provides an extensive overview on performance tuning to optimize the response time of the database. After a description of the query planner tool for PostgreSQL, there is an overview of the explain plans  and of the importance of choosing the correct keys and indexes.  
It follows a description of some typical SQL pattern (subselect, from subselect, self-joins) and how they can impact on performance.  
Finally the chapter describes the possible effectiveness of some system settings and functions, and make some remarks on problems generated by the geometries (for example, to avoid invalid geometry, to use simplification of shapes, to remove holes and clustering).

### PART III

The last part of the book, part three, is possibly the most interesting part, being the most applicative one: it describes the use of PostGIS with other tools and frameworks.

In particular, **Chapter 10** describes how to expand the capabilities of Postgres SQL with add ons such as the TIGER geocoder (specific to U.S. data of U.S. Census), the pgRouting library, which allows the resolution of numerical optimization problems on graphs (shortest path algorithms and traveling salesperson problem) and the extension of functionality of Postgres with another language, such as Python or R (but there is support for many others), by using PLs.  
At this point there is an extensive and interesting tutorial that shows the interface with R, the well-known statistical language, through PL/R, to develop stored procedures with statistics functions. The tutorial introduces rgdal as well (the library interface R/GDAL), by which there is an interface to spatial data outside the database.  
Python developers will find interesting the following tutorial, that explains how it is possible to extend Postgres using Python libraries with PL/Python. The tutorials shows how to quick access to external data (for example an Excel file) and how to do geocoding (with the googlemaps library, but you could get similar results with geopy for querying other services like Geonames).

**Chapter 11** shows some typical use cases of PostGIS in the development of web applications. After a quick look at some technologies, the authors present a nice tutorial mixing OpenLayers (with GeoExt), JavaScript and use of OGC services like WMS/WFS from MapServer and/or GeoServer.  
At this point there is another tutorial that shows how you can interact with PostGIS from within PHP (but the code can be easily adapted to other languages) to dynamically generate a KML file for viewing a PostGIS layer on Google Earth.  
There is then an example about how to produce a GeoJSON file dynamically using PHP from a PostGIS table. This GeoJSON dynamically generated is then used for feeding a layer in OpenLayer (within GeoExt).  
Obviously, given that the subject of this chapter is very large, this chapter can be regarded only as a quick overview, which must certainly be supplemented by further readings if you are interested in the topic.

**Chapter 12** introduces some main Desktop GIS software that works well in conjunction with PostGIS: OpenJUMP, QGIS (my favorite), uDig and gvSIG.

**Chapter 13**, the last one of the book, introduces the new PostGIS Raster module (formerly known as WKT Raster in the first release candidate). This module offers the great possibility to finally manage raster data in PostGIS.  
In the current PostGIS version (1.5), you will need to install separately  this module.  
In PostGIS 2.0, which should be released in the upcoming weeks, the module will be part of the core software.  
The chapter explains how to store and load raster in PostGIS (using the Python raster2pgsql.py utility, which supports input in any format based on GDAL), and major commands necessary to obtain information from raster loaded in the spatial database.  
Then there is a full explanation of the purpose of the raster_columns table (similar to geometry_columns table for vector data) and the function AddRasterColumn (similiar to tAddGeometryColumn for vector data).  
The book then moves on to an overview of the main functions for raster (ST_Value, ST_BandNoDataValue, ST_SetBandNoDataValue, ST_Envelope, ST_Polygon, ST_ConvexHull, ST_SetGeoReference, ST_SetSRID, ST_SetUpperLeft, ST_SetScale, ST_Intersection). This overview are accompanied with the usual lighting tutorials: in one of them there is an interesting example about how to add the z-coordinate to a LineString from 2D pixel starting from a DEM in the spatial database.  
The chapter ends with the explanation of PostGIS raster export to other formats. I guess you will not be surprised to see how it comes back into game again the universal GDAL: through the new "PostGIS WKT Raster" driver (from version 1.7) you will be able to export to the plethora of supported GDAL raster formats.  
Other GDAL commands that comes very handy are the well-known gdal_translate command (to perform several operations on raster) and the gdalwarp command (to reproject and warp raster). Finally the chapter shows how to use PostGIS raster from MapServer using the "PostGIS WKT Raster" driver.

The book terminates with four appendices: **Appendix A** shows a series of resources and websites focused on PostGIS.  
**Appendix B** provides instructions for installation, compilation and upgrade of Postgres and PostGIS on Linux, Windows and Mac OS X. It also explains how to create your PostGIS template and your first spatial database.  
**Appendix C** provides a quick SQL tutorial, for those who come from one a non-relational world (like those of you that have always being using flat files as data sources).  
Finally, **Appendix D** provides a brief overview and tutorial on Postgres, to make the reader immediately productive before starting to read the book, if he has never been exposed to this powerful relational database.  

The book has been available since many months as a PDF ebook: now, since a few days, it is finally possible to buy the print book version from the [Manning web site] [1] or [Amazon] [2].  
I think anyone of us interested in GIS should get a copy of it: this is one of those books that we will keep with us on our desk for years and to which we will return occasionally to take an extensive read!

[1]: http://www.manning.com/obe/

[2]: http://www.amazon.com/PostGIS-Action-Regina-Obe/dp/1935182269/ref=sr_1_1?ie=UTF8&s=books&qid=1303395068&sr=8-1
