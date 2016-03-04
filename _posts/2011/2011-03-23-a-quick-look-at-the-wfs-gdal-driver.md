---
layout: post
title: "A quick look at the WFS GDAL Driver"
description: "A quick look at the WFS GDAL Driver"
category:
tags: [GIS, GDAL, OGR, WFS, Python]
---
{% include JB/setup %}

If you have ever tried to interact with a WFS (via a browser, or curl, or OpenLayers or whatever), you are fully aware that it has alway been a pain to interact with, and until now the only Python library that made life simpler was the excellent OWSLib by Sean Gillies (that, BTW, will deserve itself another post at this blog in the next weeks).

But since some weeks, from the release of the GDAL version 1.8.0, a new toolset is available to GIS users and developers via the new included GDAL WFS Driver (that will now make good company to the already existing WCS and WMS GDAL drivers).

Thanks to this driver, GDAL, in particoular the OGR part of it (dedicated to vectorial dataset) is fully interoperable with WFS, and a WFS has for both the GDAL command line utilities (ogrinfo and ogr2ogr) and as well for the GDAL API, the same dignity as other vectorial datasources like Shapefile, PostGIS layers, Spatialite tables and so on.

Today I finally had some time to give a try to this new driver and I must admit that, as far as I have seen, I am greatly impressed by it!

### Using the WFS driver with GDAL/OGR command line utilities

The first task I wanted to accomplish was to investigate, query and export some of the WFSs we have packaged here at my job place.

For the first two tasks (investigate, query) the ogrinfo was the perfect tool to use, for the last task (query) I have found in ogr2ogr, as always, an invaluable utility to export (and import in case you are using a WFS-T!) data from a WFS to the plethora of vectorial formats supported by GDAL.

So in the following lines there are the results of what I got so far.

#### Listing the layers of a WFS server

For getting some information about the layers and their metadata in the WFS layer, ogrinfo is absolutely the way to go:

    $ ogrinfo -ro WFS:"http://geohub.jrc.ec.europa.eu/effis/ows"
    INFO: Open of `WFS:http://geohub.jrc.ec.europa.eu/effis/ows'
          using driver `WFS' successful.
    1: EFFIS:FireNews (Point)
    2: EFFIS:Fires30Days (Point)
    3: EFFIS:Fires7Days (Point)
    4: EFFIS:FiresAll (Point)
    5: EFFIS:Hotspots1Day (Point)
    6: EFFIS:Hotspots7Days (Point)
    7: EFFIS:HotspotsAll (Point)

#### Querying a layer in a WFS server

Still ogrinfo is a very handy way to query metadata and features in a layer (note how with the -where option it is very handy to filter out features. Note that this can be made also with spatial filters):

    $ ogrinfo -ro WFS:"http://geohub.jrc.ec.europa.eu/effis/ows" FiresAll -where "Country = 'PT' AND Area_HA > 200"
    INFO: Open of `WFS:http://geohub.jrc.ec.europa.eu/effis/ows'
          using driver `WFS' successful.

    Layer name: EFFIS:FiresAll
    Geometry: Point
    Feature Count: 2
    Extent: (-8.280308, 40.457099) - (-7.537176, 42.016641)
    Layer SRS WKT:
    GEOGCS["WGS 84",
        DATUM["WGS_1984",
            SPHEROID["WGS 84",6378137,298.257223563,
                AUTHORITY["EPSG","7030"]],
            AUTHORITY["EPSG","6326"]],
        PRIMEM["Greenwich",0,
            AUTHORITY["EPSG","8901"]],
        UNIT["degree",0.0174532925199433,
            AUTHORITY["EPSG","9122"]],
        AUTHORITY["EPSG","4326"]]
    Geometry Column = the_geom
    gml_id: String (0.0)
    FireDate: String (0.0)
    Country: String (0.0)
    Province: String (0.0)
    Commune: String (0.0)
    Area_HA: Real (0.0)
    CountryFul: String (0.0)
    Class: String (0.0)
    X: Real (0.0)
    Y: Real (0.0)
    LastUpdate: String (0.0)
    OGRFeature(EFFIS:FiresAll):7
      gml_id (String) = FiresAll.7
      FireDate (String) = 06-02-2011
      Country (String) = PT
      Province (String) = Serra da Estrela
      Commune (String) = Gouveia (Sao Pedro)
      Area_HA (Real) = 237
      CountryFul (String) = Portugal
      Class (String) = 30DAYS
      X (Real) = 2841252.28
      Y (Real) = 2101233.83
      LastUpdate (String) = 07-02-2011
      POINT (-7.537176271069066 40.457099360980237)

    OGRFeature(EFFIS:FiresAll):10
      gml_id (String) = FiresAll.10
      FireDate (String) = 26-01-2011
      Country (String) = PT
      Province (String) = Minho-Lima
      Commune (String) = Gave
      Area_HA (Real) = 316
      CountryFul (String) = Portugal
      Class (String) = 30DAYS
      X (Real) = 2818227.42
      Y (Real) = 2284948.54
      LastUpdate (String) = 06-02-2011
      POINT (-8.280307882823605 42.016640863507767)

#### Exporting a WFS layer to a shapefile (or to whatever)

Finally, for exporting some features from a WFS layer to an OGR format like shapefile, the ubiquitous ogr2ogr did the job with great efficiency:

    $ ogr2ogr -f 'ESRI Shapefile' fires WFS:"http://geohub.jrc.ec.europa.eu/effis/ows" FiresAll

### Using the WFS driver with the  GDAL API bindings

The real power of GDAL is the API behind it, with which also the command line utilities like ogrinfo and ogr2ogr that we have previously used are written.

For the convenience of the developers, several bindings to different languages are offered (via SWIG).

As far as I am a very big fun of Python, here I will show how, with some simple lines of this language, you can access to the data behind a WFS in the same manner as with other vectorial datasets.

#### Accessing to WFS metadata by code

In this first sample script, I want to show how easy it is to access to the metadata of a WFS server, i.e. the list of the layers, and some of their attributes.

As always the starting point is to create an OGR driver (in this case a WFS one), and then the access to all the classes of the API are just the same like for other OGR drivers (for example for a shapefile).

(sample.py)
{% highlight python %}
from osgeo import ogr
driver = ogr.GetDriverByName('WFS')
wfs = driver.Open("WFS:http://geohub.jrc.ec.europa.eu/effis/ows")
for i in range(0, wfs.GetLayerCount()):
    layer = wfs.GetLayerByIndex(i)
    sr = layer.GetSpatialRef()
    print 'Layer: %s, Features: %s, SR: %s...' % (layer.GetName(), layer.GetFeatureCount(), sr.ExportToWkt()[0:50])
{% endhighlight %}

    $ python sample.py
    Layer: EFFIS:FireNews, Features: 7, SR: GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84"...
    Layer: EFFIS:Fires30Days, Features: 14, SR: GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84"...
    Layer: EFFIS:Fires7Days, Features: 0, SR: GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84"...


#### Accessing to WFS features by code

In this second sample script, I am iterating through all the features of a layer and for each feature I get the Json representation.

(sample2.py)
{% highlight python %}
from osgeo import ogr
driver = ogr.GetDriverByName('WFS')
wfs = driver.Open("WFS:http://geohub.jrc.ec.europa.eu/effis/ows")
layer = wfs.GetLayerByName('FiresAll')
for feature in layer:
    print 'Json representation for Feature: %s' % feature.GetFID()
    print feature.ExportToJson()
{% endhighlight %}

    $ python sample2.py
    Json representation for Feature: 1
    {"geometry": {"type": "Point", "coordinates": [-7.0855540000000001, 42.070912999999997]}, "type": "Feature", "properties": {"Province": "Ourense", "Commune": "Gudina, A", "LastUpdate": "11-02-2011", "FireDate": "06-02-2011", "Country": "ES", "Y": 2267694.6600000001, "gml_id": "FiresAll.1", "CountryFul": "Spain", "Area_HA": 55.0, "X": 2915712.1400000001, "Class": "30DAYS"}, "id": 1}
    Json representation for Feature: 2
    {"geometry": {"type": "Point", "coordinates": [-7.1695520000000004, 42.141610999999997]}, "type": "Feature", "properties": {"Province": "Ourense", "Commune": "Viana do Bolo", "LastUpdate": "11-02-2011", "FireDate": "06-02-2011", "Country": "ES", "Y": 2276947.27, "gml_id": "FiresAll.2", "CountryFul": "Spain", "Area_HA": 58.0, "X": 2910589.8399999999, "Class": "30DAYS"}, "id": 2}

So far as I have seen, once again GDAL reveals itself an invaluable toolsets and API that any serious GIS developers should master.

Brilliant!
