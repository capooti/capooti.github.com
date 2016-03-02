---
categories: Uncategorized, PostGIS, Python, OpenStreetMap, OpenLayers, GDAL, Django,
  GeoDjango
date: 2009/04/01 18:17:29
guid: http://www.paolocorti.net/?p=93
permalink: http://www.paolocorti.net/2009/04/01/a-day-with-geodjango/
tags: Uncategorized, PostGIS, Python, OpenStreetMap, OpenLayers, GDAL, Django, GeoDjango
title: A day with GeoDjango
---
This time i will introduce a really brilliant framework that every serious Web/GIS developers should be aware of: GeoDjango.

Django is a Python Web framework for agile developers. It implements best web frameworks practices like coding by convention, MVC, ORM, REST, URL dispatcher and so on. Django is for Python what Rails is for Ruby, Grails is for Java, and MonoRail (and now ASP.NET MVC) is for .NET.

GeoDjango is a Django application that <a href="http://code.djangoproject.com/svn/django/trunk/django/contrib/gis/">is now included in the Django trunk</a> with a lot of excellent stuff for developing GIS web application.

<em><strong>GeoDjango Building Blocks</strong></em>

GeoDjango installation is based on Python, Django and two kinds of components: a Spatial Database and Geospatial libraries

<strong>1) Spatial Database</strong>
A spatial database like PostGis (recommended), MySQL Spatial and Oracle Spatial (and since some day also SpatiaLite, that is an excellent option for development purposes)

<strong>2) GeoSpatial libraries</strong>
The basic open sources geospatial libraries needed from GeoDjango are:
<ul>
<li>GEOS (Geometry Engine Open Source), a C++ port of the JTS (Java Topology Suite) that implements the <a href="http://www.opengeospatial.org/standards/sfs">OCG Simple Feature for SQL specification</a></li>
<li>GDAL (Geospatial Data Abstraction Library), specifically the OGR Simple Feature Library, that gives read and, in many cases, write access to a variety of vector file formats, implementing the <a href="http://www.opengeospatial.org/standards/sfa">OCG Simple Feature specification</a></li>
<li>PROJ.4 (Cartographic Projections library), an ubiquitous open source GIS library for managing spatial reference systems and projections</li>
<li>GeoIP C API, an IP-based geolocation library</li>
</ul>

GeoDjango, in order to correctly work, requires only GEOS. Plus, you will need PROJ.4 if you want to use PostGIS as the spatial database, and GDAL if you want to use some GeoDjango utilities like the excellent <a href="http://geodjango.org/docs/layermapping.html">LayerMapping class</a>.

Note that you can use the GeoDjango Python interfaces to GEOS, GDAL and GeoIP indipendently from Django in your Python scripts, in the Python shell, or in different Python Web Frameworks.

<em><strong>GeoDjango goodness</strong></em>

Let's see what are the major benefits provided in Django by GeoDjango

<strong>1) GeoDjango models</strong>

With GeoDjango you can use Spatial attributes in your models, like in this sample:

<pre lang="python">
from django.contrib.gis.db import models

class Lakes(models.Model):
    name = models.CharField(max_length=100)
    rate = models.IntegerField()
    geom = models.MultiPolygonField()
    objects = models.GeoManager()
</pre>

note that now Model is imported from django.contrib.gis.db and not from django.db (from which is inherited).

The geometry fields implemented are the ones defined by OGS Simple Feature:
<ul>
<li>PointField</li>
<li>LineStringField</li>
<li>PolygonField</li>
<li>MultiPointField</li>
<li>MultiLineStringField</li>
<li>MultiPolygonField</li>
<li>GeometryCollectionField</li>
</ul>

The last line in the above sample is very important because let Django to override the base Manager for returning Geographic QuerySet.
You need to add this line if you want to use the spatial SQL benefit.

When you will sync your model with the database, the corresponding spatial DDL SQL will be produced and run against the database, in our case, if we would be using PostGIS

<pre lang="sql">
BEGIN;
CREATE TABLE "land_lakes" (
    "id" serial NOT NULL PRIMARY KEY,
    "name" varchar(100) NOT NULL,
    "rate" integer NOT NULL
)
;
SELECT AddGeometryColumn('land_lakes', 'geom', 4326, 'MULTIPOLYGON', 2);
ALTER TABLE "land_lakes" ALTER "geom" SET NOT NULL;
CREATE INDEX "land_lakes_geom_id" ON "land_lakes" USING GIST ( "geom" GIST_GEOMETRY_OPS );
COMMIT;
</pre>

<strong>2) LayerMapping utility</strong>

LayerMapping is a very nice utility for import data like shapefiles or other OGR data sources in the spatial database.

For example for our Lakes model it would be possible to import a shapefile (or whatever OGR datasources) called lakes creating a script like this:

<pre lang="python">
import os
from django.contrib.gis.utils import LayerMapping
from land.models import Lakes

# Auto-generated `LayerMapping` dictionary for Lakes model
lakes_mapping = {
    'name' : 'name',
    'rate' : 'rate',
    'geom' : 'MULTIPOLYGON',
}

lake_shp = os.path.abspath(os.path.join(os.path.dirname(__file__), '../data/lakes.shp'))

def run(verbose=True):
    lm = LayerMapping(Lakes, lake_shp, lakes_mapping,
                      transform=False, encoding='iso-8859-1')

    lm.save(strict=True, verbose=verbose)
</pre>

Now you may run the script from the shell, and the shapefile will be loaded in your spatial database:

<pre lang="python">
from land.scripts import loadshape
loadshape.run()
</pre>

<strong>3) ogrinspect</strong>

ogrinspect is a new option for the manage.py command line Django utility that will read a OGC data source and will automatically generate a Django model and a LayerMapping dictionary for being used with the LayerMapping utility.

<pre lang="text">
$ python manage.py ogrinspect land/data/lakes.shp Lakes --srid=4326 --mapping --multi
</pre>

this will generate the output we have used above for the model and the LayerMapping script:

<pre lang="python">
# This is an auto-generated Django model module created by ogrinspect.
from django.contrib.gis.db import models

class Lakes(models.Model):
    name = models.CharField(max_length=80)
    rate = models.IntegerField()
    geom = models.MultiPolygonField(srid=4326)
    objects = models.GeoManager()

# Auto-generated `LayerMapping` dictionary for Lakes model
lakes_mapping = {
    'name' : 'name',
    'rate' : 'rate',
    'geom' : 'MULTIPOLYGON',
}
</pre>

<strong>4) Geographic Admin</strong>

By importing the admin stuff in your GeoDjango project, the admin will automatically manage your geometry field with an OpenLayers interface and an OpenStreetMap WMS as the base map (but it can be easily implemented a base map with GoogleMap, Virtual Earth, and so on).

Just add this to your admin.py:

<pre lang="python">
from django.contrib.gis import admin
from models import Lakes

admin.site.register(Lakes, admin.GeoModelAdmin)
</pre>

Now in the admin web interface, when editing the GeoDjango datasets, you will have an OpenLayers interface for editing the feature's geometry.

<img src="http://www.paolocorti.net/wp-content/uploads/2009/04/geodjango_admin-300x225.png" alt="The geometry field in the Django admin with GeoDjango" title="geodjango_admin" width="300" height="225" class="size-medium wp-image-107" />

<strong>5) GeoDjango Database and GEOS API</strong>

The <a href="http://geodjango.org/docs/db-api.html">Database API</a> and the <a href="http://geodjango.org/docs/geos.html">GEOS API</a> will basically let you manage stuff with database features. You will be able to create, update and delete features, and managing geographic querysets, spatial query and geometry operations.

Some samples:

<strong>5.1 creating a feature</strong>

<pre lang="python">
>>>from django.contrib.gis.geos import GEOSGeometry
>>>from land.models import Lakes
>>>newlake = Lakes(name='newlake', rate=3, >>>geom=GEOSGeometry('MULTIPOLYGON((( 1 1, 1 2, 2 2, 1 1)))'))
>>>Inewlake.save()
</pre>

sql behind the scenes:
<pre lang="sql">
INSERT INTO "land_lakes" ("name", "rate", "geom") VALUES (E'newlake', 3, ST_GeomFromWKB('\\001\\...', 4326))
</pre>

<strong>5.2 using a spatial operator</strong>

<pre lang="python">
>>>lake3 = Lakes.objects.get(id=3)
>>>newlake.geom.contains(lake3.geom)
True
</pre>

sql behind the scenes:

<pre lang="sql">
SELECT "land_lakes"."id", "land_lakes"."name", "land_lakes"."rate", "land_lakes"."geom" FROM "land_lakes" WHERE ST_Touches("land_lakes"."geom", ST_GeomFromWKB('\\001\\...', 4326))
</pre>

<strong>5.3 querysets methods</strong>

<pre lang="python">
>>>newlake.geom.area()
0.5
>>>centroid = newlake.geom.centroid
>>>print 'x:%s, y:%s' % (centroid.x, centroid.y)
x:1.33333333333, y:1.66666666667
>>>newlake.geom.envelope.area
1.0
>>>newlake.geom.coords
((((1.0, 1.0), (1.0, 2.0), (2.0, 2.0), (1.0, 1.0)),),)
>>>newlake.geom.get_srid()
4326
>>>newlake.geom.num_coords
4
>>>newlake.geom.num_geom
1
</pre>

<strong>5.4 representation</strong>

<pre lang="python">
>>>newlake.geom.wkt
'MULTIPOLYGON (((1.0000000000000000 1.0000000000000000, 1.0000000000000000 2.0000000000000000, 2.0000000000000000 2.0000000000000000, 1.0000000000000000 1.0000000000000000)))'
>>>newlake.geom.wkb
<read-only buffer for 0x8d83170, size -1, offset 0 at 0x8d6cec0>
>>>newlake.geom.kml
'<MultiGeometry><Polygon><outerBoundaryIs><LinearRing><coordinates>1.0,1.0,0 1.0,2.0,0 2.0,2.0,0 1.0,1.0,0</coordinates></LinearRing></outerBoundaryIs></Polygon></MultiGeometry>'
>>>newlake.geom.json
'{ "type": "MultiPolygon", "coordinates": [ [ [ [ 1.000000, 1.000000 ], [ 1.000000, 2.000000 ], [ 2.000000, 2.000000 ], [ 1.000000, 1.000000 ] ] ] ] }'
>>>newlake.geom.ewkt
'SRID=4326;MULTIPOLYGON (((1.0000000000000000 1.0000000000000000, 1.0000000000000000 2.0000000000000000, 2.0000000000000000 2.0000000000000000, 1.0000000000000000 1.0000000000000000)))'
</pre>

<strong>5.5 Geoprocessing</strong>

<pre lang="python">
>>>bufferedgeom = newlake.geom.buffer(1)
>>>from django.contrib.gis.geos import MultiPolygon
>>>mp = MultiPolygon(bufferedgeom)
>>>bufferedlake = Lakes(name='bufferedlake', rate=4, geom=mp)
>>>bufferedlake.save()
</pre>

<strong>6) Measurement Units API</strong>

<pre lang="python">
>>>from django.contrib.gis.measure import Distance
>>>dist = Distance(km=1)
>>>dist
1.0 km
>>>dist.m
1000.0
>>>dist.mi
0.62137119223733395
>>>area = Area(sq_m=1)                        
>>>area.sq_m
1.0
>>>area.sq_mi
3.8610215854244582e-07
</pre>

<strong>7) GDAL API</strong>

GDAL API is a fantastic API to read (and in many cases write) <a href="http://www.gdal.org/ogr/ogr_formats.html">several vector data sources</a>.

As written before, you may not need GDAL in your GeoDjango project, but it can be a very useful Python library for accessing other spatial dataset, for importing/exporting and for system integration.
For example in this sample i will copy one feature's geometry from a shapefile and will create a new feature with the same geometry in my GeoDjango spatial database:

<pre lang="python">
>>>from django.contrib.gis.gdal import DataSource
>>>ds = DataSource('home/paolo/tralining/geodjango/land/data/lakes.shp')
>>>ds.name
('home/paolo/tralining/geodjango/land/data/lakes.shp'
>>>ds.layer_count
1
>>>lakesshape = ds[0] # shapefile have only one layer
>>>lakesshape.name
'lakes'
>>>lakesshape.num_feat
2
>>>feature = lakesshape[0]
>>>feature.geom.geom_type.name
'Polygon'
>>>from django.contrib.gis.gdal import OGRGeometry
>>>mpgeom = OGRGeometry('MultiPolygon')
>>>mpgeom.add(feature.geom)
>>>importedlake = Lakes(name='importedlake', rate=0, geom=GEOSGeometry(mpgeom.wkt))
>>>importedlake.save()
</pre>

<strong>Resources for getting started with Django/GeoDjango</strong>

Django and GeoDjango, even being relatively young Open Source projects, have a fantastic documentation. Here is a list of the best web resources i found to get started with it:

<ul>
<li><a href="http://docs.djangoproject.com/en/dev/intro/tutorial01/">Django Tutorial</a></li>
<li><a href="http://docs.djangoproject.com/en/dev/">Django Documentation</a></li>
<li><a href="http://www.djangobook.com/">The free Django Book</a>, the second edition has just been released! </li>
<li><a href="http://geodjango.org/docs/tutorial.html">A fantastic GeoDjango tutorial</a></li>
<li><a href="http://geodjango.org/docs/index.html">The GeoDjango Documentation</a></li>
<li><a href="http://code.djangoproject.com/svn/django/trunk/django/contrib/gis/">GeoDjango SVN Trunk</a> (source code is extremely well commented)</li>
<li><a href="http://code.google.com/p/geodjango-basic-apps/">GeoDjango Basic Apps</a></li>
</ul>

Time to have fun with GeoDjango!
