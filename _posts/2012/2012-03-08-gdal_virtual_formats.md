---
layout: post
title: "The power of GDAL virtual formats"
description: "The power of GDAL virtual formats"
category:
tags: [GIS, GDAL, python, qgis, mapserver]
---
{% include JB/setup %}

GDAL is most likely the most powerfull GIS toolset out there, with very geekish features that may seem complicated at first, but become extremely powerful and simple at the same time once you master them. <br />
One of these features I most like is the virtual format concept, valid both for raster data sources (GDAL) and for vectorial data sources (OGR).
Basically GDAL gives to the user a very simple mechanism to create virtual formats from some different sources.<br />
Let's analyze this feature, with a couple of samples derived from real world scenarios I have been envolved in the last weeks.

### Raster GDAL Virtual Formats ###

GDAL gives the possibility to create [GDAL virtual formats] [1]: it is possible to create a GDAL dataset _"composed from other GDAL datasets with repositioning, and algorithms potentially applied as well as various kinds of metadata altered or added. VRT descriptions of datasets can be saved in an XML format normally given the extension .vrt."_<br />
In the virtual GDAL dataset it is possible, by simply using some xml, to manage, alter, and add important elements of a raster, like the spatial reference, the six values for the affine geotransformation, the metadata, the mask band.<br />
For each band of the resulting raster, it is possible to manage important parameters like the NoDataValue, the ColorTable the vertical units, just to name a few ones.<br />

In a recent project I had a netCDF dataset, composed of 104 subdatasets (weather forecast variables), each of them composed of 10 bands (each band with the forecast for one of the next 10 following days).<br />
I wanted to have a single virtual raster managing the values of 3 of the 107 variables for each forecast day, for using it within MapServer (and at some extent in QGIS).

This is what I have been using in xml for creating the virtual raster composed of 3 bands with the values of the 3 variables in each band for the 7th day of the forecast:

{% highlight xml %}
<VRTDataset rasterXSize="1280" rasterYSize="640">
  <SRS>GEOGCS["WGS 84",
    DATUM["WGS_1984",
        SPHEROID["WGS 84",6378137,298.257223563,
            AUTHORITY["EPSG","7030"]],
        AUTHORITY["EPSG","6326"]],
    PRIMEM["Greenwich",0,
        AUTHORITY["EPSG","8901"]],
    UNIT["degree",0.0174532925199433,
        AUTHORITY["EPSG","9122"]],
    AUTHORITY["EPSG","4326"]]</SRS>
  <GeoTransform> -1.4062500000000000e-01,  2.8125000000000000e-01,  0.0000000000000000e+00,  8.9925385321783935e+01,  0.0000000000000000e+00, -2.8101682913057480e-01</GeoTransform>
  <Metadata>
    <MDI key="raster_sample_key">My metadata for my virtual raster</MDI>
  </Metadata>
  <VRTRasterBand dataType="Float32" band="1">
    <Metadata>
      <MDI key="band_sample_key">My metadata for my virtual raster band variable 1</MDI>
    </Metadata>
    <NoDataValue>0</NoDataValue>
    <SimpleSource>
      <SourceFilename relativeToVRT="0">NETCDF:/path/to/ncdf/mydatset.nc4:my_var_1</SourceFilename>
      <SourceBand>7</SourceBand>
      <SourceProperties RasterXSize="1280" RasterYSize="640" DataType="Float32" BlockXSize="1280" BlockYSize="1"/>
      <SrcRect xOff="0" yOff="0" xSize="1280" ySize="640"/>
      <DstRect xOff="0" yOff="0" xSize="1280" ySize="640"/>
    </SimpleSource>
  </VRTRasterBand>
  <VRTRasterBand dataType="Float32" band="2">
    <Metadata>
      <MDI key="band_sample_key">My metadata for my virtual raster band variable 2</MDI>
    </Metadata>
    <NoDataValue>0</NoDataValue>
    <SimpleSource>
      <SourceFilename relativeToVRT="0">NETCDF:/path/to/ncdf/mydatset.nc4:my_var_2</SourceFilename>
      <SourceBand>7</SourceBand>
      <SourceProperties RasterXSize="1280" RasterYSize="640" DataType="Float32" BlockXSize="1280" BlockYSize="1"/>
      <SrcRect xOff="0" yOff="0" xSize="1280" ySize="640"/>
      <DstRect xOff="0" yOff="0" xSize="1280" ySize="640"/>
    </SimpleSource>
  </VRTRasterBand>
  <VRTRasterBand dataType="Float32" band="3">
    <Metadata>
      <MDI key="band_sample_key">My metadata for my virtual raster band variable 3</MDI>
    </Metadata>
    <NoDataValue>0</NoDataValue>
    <SimpleSource>
      <SourceFilename relativeToVRT="0">NETCDF:/path/to/ncdf/mydatset.nc4:my_var_3</SourceFilename>
      <SourceBand>7</SourceBand>
      <SourceProperties RasterXSize="1280" RasterYSize="640" DataType="Float32" BlockXSize="1280" BlockYSize="1"/>
      <SrcRect xOff="0" yOff="0" xSize="1280" ySize="640"/>
      <DstRect xOff="0" yOff="0" xSize="1280" ySize="640"/>
    </SimpleSource>
  </VRTRasterBand>
</VRTDataset>
{% endhighlight %}

At this point this is effectively a raster for GDAL and all the software and tools based on it.<br />
For example it will be easily possible to query it with gdalinfo or manipulate it with some other GDAL command line utilities, to display it with MapServer or desktop software like QGIS or even to access it programmatically with the GDAL API.

The GDAL command line utility gdalinfo will produce the following output:

    $ gdalinfo my_vrt_raster.vrt
    Driver: VRT/Virtual Raster
    Files: my_vrt_raster.vrt
    Size is 1280, 640
    Coordinate System is:
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
    Origin = (-0.140625000000000,89.925385321783935)
    Pixel Size = (0.281250000000000,-0.281016829130575)
    Metadata:
      raster_sample_key=My metadata for my virtual raster
    Corner Coordinates:
    Upper Left  (  -0.1406250,  89.9253853) (  0d 8'26.25"W, 89d55'31.39"N)
    Lower Left  (  -0.1406250, -89.9253853) (  0d 8'26.25"W, 89d55'31.39"S)
    Upper Right (     359.859,      89.925) (359d51'33.75"E, 89d55'31.39"N)
    Lower Right (     359.859,     -89.925) (359d51'33.75"E, 89d55'31.39"S)
    Center      ( 179.8593750,   0.0000000) (179d51'33.75"E,  0d 0' 0.01"N)
    Band 1 Block=128x128 Type=Float32, ColorInterp=Undefined
      NoData Value=0
      Metadata:
        band_sample_key=My metadata for my virtual raster band variable 1
    Band 2 Block=128x128 Type=Float32, ColorInterp=Undefined
      NoData Value=0
      Metadata:
        band_sample_key=My metadata for my virtual raster band variable 2
    Band 3 Block=128x128 Type=Float32, ColorInterp=Undefined
      NoData Value=0
      Metadata:
        band_sample_key=My metadata for my virtual raster band variable 3

It will be possible to convert this virtual raster to a different raster format with gdal_translate (the reverse is also possible, of course):

    $ gdal_translate -of JPEG my_vrt_raster.vrt my_raster.jpeg

MapServer directly support this virtual GDAL format, and this is how I defined the layer in the mapfile:

    # virtual raster
    LAYER
        NAME "virtualraster"
        DATA "/path/to/vrt/my_vrt_raster.vrt.vrt"
        STATUS ON
        TEMPLATE     "data/my_vrt_raster_template.html"
        PROJECTION
           "init=epsg:4326"
        END
        TYPE RASTER
        METADATA
            "wms_title"           "WMS Virtual Raster"
            "wms_srs"             "EPSG:4326"
        END
    END

QGIS will correctly display it using the GDAL Virtual Raster driver, and finally it will be accessible programmatically using the GDAL API: for example this is how in Python is is possible to get the metadata information of the first band:

{% highlight python %}
>>> from osgeo import gdal
>>> from osgeo.gdalconst import *
>>> dataset = gdal.Open('/path/to/vrt/my_vrt_raster.vrt.vrt', GA_ReadOnly)
>>> band1 = dataset.GetRasterBand(1)
>>> print band1.GetMetadata_List()[0]
>>> band_sample_key=My metadata for my virtual raster band variable 1
{% endhighlight %}

Note that he gdalbuildvrt GDAL command will build a vrt from a list of datasets, and it is very handy if you want to generate the vrt file without too much effort.

### Vectorial OGR Virtual Formats ###

In OGR, the vectorial "part" of GDAL, there is also the possibility to compose [virtual formats] [2].<br />
This format is an excellent possibility if you need to derive spatial layers from formats that are not directly spatially enabled like flat files and flat database tables with spatial information not stored in common formats like wkt or wkb.<br />
You can create a virtual format mapping the original data source with an xml file: after creating this file it will be possible to read (and if some basic condition is met even to write!) the original datasource without using interchange formats.

A couple of situation I have met in my latest projects: in the first one I had to read a flat table in oracle storing point geometries, with the geographic information stored as two double fields (x, y), and at the same time filtering the results on a CATEGORY field value.<br />
This is how I ended up creating the virtual driver to the table (using the GeometryField xml subelement for defining where the geometric information is stored):

{% highlight xml %}
<OGRVRTDataSource>
    <OGRVRTLayer name="my_table">
        <SrcDataSource>OCI:user/password@server/schema</SrcDataSource>
     	<SrcSQL>select * from MY_TABLE where CATEGORY = 1</SrcSQL>
	    <GeometryType>wkbPoint</GeometryType>
            <LayerSRS>WGS84</LayerSRS>
	    <GeometryField encoding="PointFromColumns" x="x" y="y"/>
    </OGRVRTLayer>
</OGRVRTDataSource>
{% endhighlight %}

In another scenario, my data source was some csv files storing point coordinates in two different columns: I had the need to generate a WMS service with MapServer.

This is an extract of the original datasource (mydata.csv file):

    LATITUDE,LONGITUDE, ...
    -12.935,142.612, ...
    -15.075,141.975, ...
    -15.084,141.958, ...
    ...

I ended up to use also in this case the virtual driver, without the need to generate a copy of the information in a different file storage (for example as shapefiles):

{% highlight xml %}
<OGRVRTDataSource>
    <OGRVRTLayer name="mydata">
        <SrcDataSource>CSV:/path/to/csv/mydata.csv</SrcDataSource>
        <GeometryType>wkbPoint</GeometryType>
        <LayerSRS>WGS84</LayerSRS>
        <GeometryField encoding="PointFromColumns" x="LONGITUDE" y="LATITUDE"/>
    </OGRVRTLayer>
</OGRVRTDataSource>
{% endhighlight %}

Referring to the latter xml file, this virtual dataset is now effectively supported from software using GDAL.

We can obtain information about it by using the ogrinfo command:

    $ ogrinfo mydata.vrt -al -ro -fid 900
    INFO: Open of `mydata.vrt'
          using driver `VRT' successful.
    Layer name: mydata
    Geometry: Point
    Feature Count: 14771
    Extent: (-155.117000, -40.521000) - (153.450000, 67.718000)
    Layer SRS WKT:
    GEOGCS["WGS 84",
        DATUM["WGS_1984",
            SPHEROID["WGS 84",6378137,298.257223563,
                AUTHORITY["EPSG","7030"]],
            TOWGS84[0,0,0,0,0,0,0],
            AUTHORITY["EPSG","6326"]],
        PRIMEM["Greenwich",0,
            AUTHORITY["EPSG","8901"]],
        UNIT["degree",0.0174532925199433,
            AUTHORITY["EPSG","9108"]],
        AUTHORITY["EPSG","4326"]]
    LATITUDE: String (0.0)
    LONGITUDE: String (0.0)
    ...
    OGRFeature(MCD14DL.2011273):900
      LATITUDE (String) = -27.387
      LONGITUDE (String) = 139.468
      ...

Or we could eventually convert it to a different format by using the ogr2ogr commmand:

    $ ogr2ogr mydata mydata.vrt

By using the OGR virtual datasource driver, it is possible to open the layer in QGIS.

This is how I am using the layer in the MapServer's mapfile to generate a WMS service:

    LAYER
        NAME mydata
        TYPE POINT
        DATA mydata
        CONNECTIONTYPE OGR
        CONNECTION "/path/to/data/mydata.vrt"
        STATUS ON
        TEMPLATE     "/path/to/templates/mytemplate.html"
        METADATA
            "wms_title"             "WMS virtual ogr my_data"
            "wms_srs"               "EPSG:4326"
            "wms_extent"            "-180 -90 180 90"
            "wms_enable_request"    "*"
        END
        CLASS
            SYMBOL 'circle'
            SIZE 2
            COLOR        255 0 0
        END
    END

As for virtual format for raster, also for OGR virtual layers there is the possibility to use the GDAL API.<br />
For example here I access to the virtual raster and check how many features are stored in it:

{% highlight python %}
>>> from osgeo import ogr
>>> driver = ogr.GetDriverByName('VRT')
>>> ds = driver.Open('/path/to/data/mydata.vrt', 1)
>>> layer = ds[0]
>>> layer.GetFeatureCount()
>>> 14771
{% endhighlight %}

[1]: http://www.gdal.org/gdal_vrttut.html
[2]: http://www.gdal.org/ogr/drv_vrt.html
