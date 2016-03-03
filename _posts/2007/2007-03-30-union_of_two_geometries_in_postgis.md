---
layout: post
title: "Union of two geometries in PostGIS"
description: "Union of two geometries in PostGIS"
category:
tags: [PostGIS, devs]
---
{% include JB/setup %}

For people not familiar with the Spatial SQL, I post this quick sample showing its beauty and simplicity at the same time.
We will go using PostGIS, but this could be performed in a similiar way with any GIS Database compliant with <a href="http://www.opengeospatial.org/standards/sfs">OGC Simple Feature Access - SQL Option</a>.

<strong>The geomunion function</strong>

The geomunion Open GIS function make it possible to combine two geometries and getting from these a single geometry.

{% highlight sql %}
FUNCTION geomunion(geometry, geometry)
  RETURNS geometry
{% endhighlight %}

It is very easy to generate a geoprocessed layer from an original layer, making an union of its polygons based on a common attribute.

I will show you how to use the geomunion PostGIS function with a quick sample.

<!--more-->

We will start from a MULTIPOLYGON layer, named polygon1, composed of 7 geometries with different values for the CODE field (1,2,3). In the following picture you can take a look at the polygon1 layer:

<a href='/assets/images/original.gif' title='Original Polygon layer'><img src='/assets/images/original.gif' alt='Original Polygon layer' /></a>

We will use the geomunion OpenGIS function to generate another MULTIPOLYGON PostGIS layer, named polygon1_union, composed of the polygons of polygon1 layer merged for the common value of CODE field.

We will get the polygon1_union layer, as in the picture:

<a href='/assets/images/geoprocessed.gif' title='Geoprocessed (unified) Polygon layer'><img src='/assets/images/geoprocessed.gif' alt='Geoprocessed (unified) Polygon layer' /></a>

<strong>Creation of the original PostGIS layer (polygon1)</strong>

To create polygon1 layer, just execute in your PostGIS environment the following SLQ code:

{% highlight sql %}
--create input polygon PostGIS layer for testing this function
BEGIN;
CREATE TABLE "polygon1" (gid serial PRIMARY KEY, "code" int4);
SELECT AddGeometryColumn('','polygon1','the_geom','-1','MULTIPOLYGON',2);
INSERT INTO "polygon1" ("code",the_geom) VALUES ('1','01060000000100000001030000000100000005000000E9416B83D5753141B852A25670DC5041FC407E9F0A773141566A115472DC504117F5DB212F773141138F8E6DD0DA5041F8CE0346DE753141C6A70A43D8DA5041E9416B83D5753141B852A25670DC5041');
INSERT INTO "polygon1" ("code",the_geom) VALUES ('2','01060000000100000001030000000100000005000000A4E0BB8CE97931419666F0CD28DC5041093C5D616A7C3141C401A2B72BDC504113E7E48F847C314164FCCD58D2DB5041947CC37C147A314138367AB5CFDB5041A4E0BB8CE97931419666F0CD28DC5041');
INSERT INTO "polygon1" ("code",the_geom) VALUES ('2','01060000000100000001030000000100000008000000FC407E9F0A773141566A115472DC5041670D5EE27D7831413F99EEB774DC5041AC9FD0C3A67831413DBB555627DC5041A4E0BB8CE97931419666F0CD28DC5041947CC37C147A314138367AB5CFDB5041B67037C3147831416515BA8BCDDB5041D419A0E7177731411C8FB24CDADB5041FC407E9F0A773141566A115472DC5041');
INSERT INTO "polygon1" ("code",the_geom) VALUES ('2','01060000000100000001030000000100000007000000D419A0E7177731411C8FB24CDADB5041B67037C3147831416515BA8BCDDB5041947CC37C147A314138367AB5CFDB50417841F32B4E7A3141DEB0900358DB5041FC7CE1C9437A3141E95712AD45DB5041523C054626773141031236D435DB5041D419A0E7177731411C8FB24CDADB5041');
INSERT INTO "polygon1" ("code",the_geom) VALUES ('3','01060000000100000001030000000100000006000000947CC37C147A314138367AB5CFDB504113E7E48F847C314164FCCD58D2DB50419F213232AA7C3141C5857CE251DB5041FC7CE1C9437A3141E95712AD45DB50417841F32B4E7A3141DEB0900358DB5041947CC37C147A314138367AB5CFDB5041');
INSERT INTO "polygon1" ("code",the_geom) VALUES ('3','01060000000100000001030000000100000006000000B5CB1B70037A314104A54306D4DA5041FC7CE1C9437A3141E95712AD45DB50419F213232AA7C3141C5857CE251DB50418902324DB67C3141CEB42B9028DB5041C4A9EA7F417C3141964085C0E1DA5041B5CB1B70037A314104A54306D4DA5041');
INSERT INTO "polygon1" ("code",the_geom) VALUES ('3','01060000000100000001030000000100000006000000523C054626773141031236D435DB5041FC7CE1C9437A3141E95712AD45DB5041B5CB1B70037A314104A54306D4DA50411D4C631552783141AEBC61A9C9DA504117F5DB212F773141138F8E6DD0DA5041523C054626773141031236D435DB5041');
END;
{% endhighlight %}

<strong>Creation of the geoprocessed PostGIS layer (polygon1_union)</strong>

Now we need to create the PostgreSQL table that will contain the geoprocessed layer, polygon1_union.

First we create the PostgreSQL table with the following sql code (note that we use the same fields from the original layer, polygon1)

{% highlight sql %}
--create table 'polygon1_union' for geometries to be unified
CREATE TABLE "polygon1_union" (gid serial PRIMARY KEY, "code" int4);
{% endhighlight %}

Now we add the PostGIS spatial column to this layer using the OpenGIS AddGeometryColumn function:

{% highlight sql %}
--add geometry column (schema_name,table_name,column_name,srid,type,dimension)
SELECT AddGeometryColumn('','polygon1_union','the_geom','-1','MULTIPOLYGON',2);
{% endhighlight %}

Finally we insert in the polygon1_union PostGIS layer the polygons from the polygon1 PostGIS layer merged on common values from the CODE field. Note how we have to use the multi PostGIS function in order to avoid the downcasting of multipolygon geometries to polygon geometries.

{% highlight sql %}
--generate merged polygons from original layers based on common values from the code field
--note: use the multi function because the geomunion could downcast multipolygon to polygon
insert into polygon1_union (the_geom,code)
select astext(multi(geomunion(the_geom))) as the_geom,code from polygon1
group by code
{% endhighlight %}

<strong>Results</strong>

It is easy to realize that from an original PostGIS layer composed of 7 multipolygons (polygon1) we derived a new PostGIS layer (polygon1_union) composed of 3 multipolygons.

The original PostGIS layer, polygon1, is composed of 7 multipolygons, as you can realize querying your database:

{% highlight sql %}
--get original polygon from polygon1
select code, AsText(the_geom) as the_geom from polygon1
{% endhighlight %}

In these 7 multipolygons there are 3 ones with a code value of 2, 3 ones with a code value of 3, and only one with a code value of 1.

After geoprocessing polygon1 in polygon1_union PostGIS layer with the geomunion function, there are only 3 multipolygons, one for each value of the code field.

{% highlight sql %}
--get geoprocessed polygon from polygon1_union
select code, AsText(the_geom) as the_geom from polygon1_union
{% endhighlight %}

The interesting thing about PostGIS (like for Oracle Spatial and MySLQ Spatial) is that you can query, edit and geoprocess geometries just with plain SQL, without the need of commercial software and API.
