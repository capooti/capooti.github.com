---
categories: GIS, GDAL, Oracle Spatial, GeoDjango
date: 2011/03/22 17:00:00
guid: http://www.paolocorti.net/2011/03/22/compiling-gdal-with-oracle-support/
permalink: http://www.paolocorti.net/2011/03/22/compiling-gdal-with-oracle-support/
tags: GIS, GDAL, Oracle Spatial, GeoDjango
title: Compiling GDAL with Oracle Spatial support
---

These are my quick notes for installing Oracle Instant Client on a Linux box (currently I have tested this on a Ubuntu 10.10 64 bit box and with Oracle 11.2 and GDAL 1.8.0), and then configuring GDAL for using Oracle (OCI and GeoRaster drivers support).

Finally I will show how to configure GeoDjango to use an Oracle spatial database by using the Cx_Oracle Python library.

### Installation of Oracle Instant Client

First download Oracle Client files from [here][OCI32]  or [here if you are on a 64 bit architecture][OCI64].

[OCI32]: http://www.oracle.com/technetwork/topics/linuxsoft-082809.html
[OCI64]: http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html

You need to download the following 3 zip archives:

    ~/software/oracle$ ls
    instantclient-basic-linux-x86-64-11.2.0.2.0.zip  instantclient-sdk-linux-x86-64-11.2.0.2.0.zip  instantclient-sqlplus-linux-x86-64-11.2.0.2.0.zip

Then extract the archives to a location you prefer:

    ~/software/oracle$ unzip instantclient-basic-linux-x86-64-11.2.0.2.0.zip 
    ~/software/oracle$ unzip instantclient-sqlplus-linux-x86-64-11.2.0.2.0.zip 
    ~/software/oracle$ unzip instantclient-sdk-linux-x86-64-11.2.0.2.0.zip 

You then need to set the following environment variables, for example editing the .bashrc file in your home directory:

    nano ~/.bashrc

    ...
    # Oracle (OCI)
    export ORACLE_HOME=/home/pcorti/software/oracle/instantclient_11_2
    export PATH=$PATH:$ORACLE_HOME
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME

Finally create the following soft links:

    ~/software/oracle$ cd $ORACLE_HOME
    ~/software/oracle/instantclient_11_2$ ln -s libclntsh.so.11.1 libclntsh.so

You should be able now to connect to an Oracle database with the SQL*Plus Instant Client:

    sqlplus hr/your_password@//mymachine.mydomain:port/MYDB
    SQL> SELECT * FROM MYTABLE;
    ...

### Installation of GDAL with OCI and GeoRaster support

For compiling GDAL with OCI and GeoRaster support, you first need to add this soft link:

    cd $ORACLE_HOME
    ~/software/oracle/instantclient_11_2$ ln -s . lib

Now [download][GDAL18], extract, configure and compile GDAL, for example:

    ./configure --with-oci-include=/home/pcorti/software/oracle/instantclient_11_2/sdk/include --with-oci-lib=/home/pcorti/software/oracle/instantclient_11_2 --with-python --with-pg
    make
    sudo make install

[GDAL18]: http://download.osgeo.org/gdal/gdal-1.8.0.tar.gz

when configuring make sure OCI and GEORASTER are enabled:

    OCI support:               yes
    GEORASTER support:         yes

After installing make sure you have OCI and GeoRaster as supported formats:

    :~/training/gdal/oracle$ ogr2ogr --formats
    Supported Formats:
      -> "ESRI Shapefile" (read/write)
        ...
      -> "OCI" (read/write)
        ...


    ~/training/gdal/oracle$ gdalinfo --formats
    Supported Formats:
      VRT (rw+v): Virtual Raster
      GTiff (rw+v): GeoTIFF
      ...
      GeoRaster (rw+): Oracle Spatial GeoRaster
      ..

### Testing GDAL and Oracle

Finally some GDAL tests with Oracle. We will use a connection string composed in this way:

    OCI:user/password@server:port/schema:table

#### Sample 1: export from an Oracle table to a shapefile:

    $ ogr2ogr -f "ESRI Shapefile" test.shp OCI:user/password@server:port/schema:MY_TABLE

#### Sample 2: import in a Oracle table from a shapefile

    $ ogr2ogr -f OCI OCI:user/password@server:port/schema:MY_TABLE2 test.shp

#### Sample 3: how to get information about an Oracle table:

    $ ~/training/gdal/oracle$ ogrinfo -al -fid 1 OCI:user/password@server:port/schema:MY_TABLE
    INFO: Open of `OCI:user/password@server:port/schema:MY_TABLE'
          using driver `OCI' successful.

    Layer name: MY_TABLE
    Geometry: Unknown (any)
    Feature Count: 688
    Extent: (-17.116400, 1.040350) - (38.761000, 65.833230)
    Layer SRS WKT:
    GEOGCS["WGS 84",
        DATUM["World Geodetic System 1984 (EPSG ID 6326)",
            SPHEROID["WGS 84 (EPSG ID 7030)",6378137.0,298.257223563]],
        PRIMEM["Greenwich",0.000000],
        UNIT["Decimal Degree",0.0174532925199433],
        AUTHORITY["EPSG. See 3D CRS for original information source.","4326"]]
    Geometry Column = GEOMETRY
    ID: Integer (11.0)
    PLACE: String (765.0)
    ...
      POINT (23.7332 35.3379 0)

#### Sample 4: how to create a GeoRaster from a raster

    $ gdal_translate -of georaster input.tif georaster:user/pwd@host:1521/db,out_img,raster
    Input file size is 3212, 2148
    0...10...20...30...40...50...60...70...80...90...100 - done.
    Ouput dataset: (geor:user/pwd@host:1521/db,RDT_72$,72) on RDAPRD.OUT_IMG,RASTER

    $ gdalinfo georaster:user/pwd@host:1521/db
    Driver: GeoRaster/Oracle Spatial GeoRaster
    Files: none associated
    Size is 512, 512
    Coordinate System is `'
    Subdatasets:
      SUBDATASET_3_NAME=geor:user/pwd@host:1521/db,RDAPRD.OUT_IMG
      SUBDATASET_3_DESC=RDAPRD.Table=OUT_IMG
    Corner Coordinates:
    Upper Left  (    0.0,    0.0)
    Lower Left  (    0.0,  512.0)
    Upper Right (  512.0,    0.0)
    Lower Right (  512.0,  512.0)
    Center      (  256.0,  256.0)


### Configuring GeoDjango for using an Oracle Spatial backend

Finally, these are my quick notes for using GeoDjango with an Oracle spatial backend.

First you need to install cx_Oracle, a Python db API for Oracle.
The version that is working well for me is cx_Oracle 5.0.1 (current version, 5.1 is throwing an error from Oracle, so I had to downgrade).

For installing cx_Oracle, if, like me, you are in a virtualenv, then just type:

    pip install http://sourceforge.net/projects/cx-oracle/files/5.0.1/cx_Oracle-5.0.1.tar.gz/download

Or, download the source code from the same location and as a sudo run the setup.py file:

    sudo python setup.py install

Finally, in the settings.py file of your GeoDjango application, don't forget to update the database settings:

    DATABASES = {
        'default': {
            'ENGINE': 'django.contrib.gis.db.backends.oracle',
            'NAME': 'database',
            'USER': 'user',
            'PASSWORD': 'password',
            'HOST': 'host',
            'PORT': '1521',
        }
    }

### REFERENCES

* [Oracle Forum][ref1]
* [A blog post from thinkside][ref2]

[ref1]: http://forums.oracle.com/forums/thread.jspa?messageID=4570333
[ref2]: http://blog.thinkside.co.uk/?p=218

