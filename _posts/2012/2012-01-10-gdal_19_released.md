---
categories: GIS, GDAL, GFT
date: 2012/01/10 20:00:00
guid: http://www.paolocorti.net/2012/01/10/gdal_19_released/
permalink: http://www.paolocorti.net/2012/01/10/gdal_19_released/
tags: GIS, GDAL, GFT
title: Playing with the Esri File Geodatabase and the Google Fusion Tables GDAL drivers
draft: false
---

Today in the GDAL mailing list [Frank Warmerdam] [0] has announced that GDAL 1.9.0 has finally been [released] [1].
Being a major new release, it offers many new features, but what I was waiting for is the support for [Esri File GDB] [2] and [Google Fusion Table] [3].
So I couldn't resist to install it and giving a try.

### Using GDAL with Esri File Geodatabase

For using the File Geodatabase driver I had to dowload the ESRI File Geodatabase API (you need to be registered for downloading it).

It was then just a matter of setting the value of the library path in the LD_LIBRARY_PATH variable, and using the --with-fgdb GDAL configuration option when configuring before compilation for obtaining support for FGDB in GDAL.

After compilation I could verify that I had this format supported by using the --formats option of the ogrinfo command:

    $ ogrinfo --formats
    -> "ESRI Shapefile" (read/write)
    -> "MapInfo File" (read/write)
    ...
    -> "FileGDB" (read/write)
    ...
    
I could then easily import a shapefile to a new file GDB, by using the ogr2ogr command:

    $ ogr2ogr -f "FileGDB" mygdb.gdb ~/data/shapefile/country.shp

### Support for Google Fusion Tables

Google Fusion Tables support is the first cloud storage supported by GDAL natively.
I guess many others drivers will come for other cloud computing storages, but at the moment this is a big plus for Google, if compared to others.

For using the GFT driver, you need to have CURL on your system, and configuring GDAL with the --with-curl option before compiling.

After compilation you can easily verify that your GDAL installation is supporting the GFT format:

    $ ogrinfo --formats
    -> "ESRI Shapefile" (read/write)
    -> "MapInfo File" (read/write)
    ...
    --> "GFT" (read/write)
    ...

After installation, I could then import a shapefile straight to the Google cloud, by using the ogr2ogr command with the GFT driver:

    $ ogr2ogr -f GFT "GFT:email=myuser@gmail.com password=mypassword" ~/data/shapefile/country.shp
    
And as soon as my shapefile was uploaded to the cloud, I could easily [access it with the Google Maps API] [4]

So, now that we have the GDAL support for this formats, next step will be to support them in the plethora of GIS tools using the GDAL library (QGIS, MapServer, GeoServer....).

Brilliant!

[0]: http://home.gdal.org/warmerda/
[1]: http://lists.osgeo.org/pipermail/gdal-dev/2012-January/031450.html
[2]: http://www.gdal.org/ogr/drv_filegdb.html
[3]: http://www.gdal.org/ogr/drv_gft.html
[4]: https://www.google.com/fusiontables/DataSource?snapid=S355557avFt

