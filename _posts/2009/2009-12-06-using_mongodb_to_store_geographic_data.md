---
layout: post
title: "Using MongoDb to store geographic data"
description: "Using MongoDb to store geographic data"
category:
tags: [Uncategorized, GIS, devs, Python, GDAL, MongoDb, NoSQL]
---
{% include JB/setup %}

### The NoSQL movement
<p>In the last months there has been a plenty of activity in the <a href="http://en.wikipedia.org/wiki/NoSQL">non relational (NoSQL)</a> database world.</p>
<p>NoSQL database tries to solve 3 main RDBMS problems:</p>
<ul>
<li><strong>Scalability</strong>, for example the ability <a href="http://adam.blog.heroku.com/past/2009/7/6/sql_databases_dont_scale/">to automatically partitioning data across multiple machines</a></li>
<li><strong>Performance</strong>, in some case <a  href="http://havemacwillblog.com/2008/11/10/6-reasons-why-relational-database-will-be-superseded/">RDBMS can be very slow</a></li>
<li><strong>Fixed schema</strong>: RDBMS have nice goodness (referential integrity, relationships, triggers...) but force you to store any object to a fixed schema (migrations are a pain!)</li>
</ul>
<p>Basically there are several different kinds of NoSQL database:</p>
<ul >
<li><strong>Key/Value</strong> (Scalaris, Tokio Cabinet, Voldemort): store data in key/value pairs: very efficient for performance and higly scalable, but difficult to query and to implement real world problems</li>
<li><strong>Tabular</strong> (Cassandra, HBase, Hypertable, Google BigTable): store data in tabular structures, but columns may vary in time and each row may have only a subset of the columns</li>
<li><strong>Document Oriented</strong> (CouchDb, MongoDb, Riak, Amazon SimpleDb): like Key/Value but they let you nest more values for a key. This is a nice paradigm for programmers as it becomes easy, specially with script languages (Python, Ruby, PHP...), to implement a one to one mapping relation between the code objects and the objects (documents) in the database</li>
<li><strong>Graph</strong> (Neo4J, InfoGrid, AllegroGraph): stores objects and relationships in nodes and edges of a graph. For situations that fit this model, like hierarchical data, this solution can be much much faster than the other ones</li>
</ul>
<p><strong>MongoDb</strong> is a document oriented NoSQL database. With such a database it is very easy to map the programming objects (documents) we want to store to the database. JSON is a very viable standard to do this mapping, and so MongoDb does: it stores JSON documents in the database.</p>
<p>To makes performance better JSON is stored by MongoDb in a efficient binary format called BSON. BSON is a binary serialization of JSON-like documents and stands for Binary JSON.</p>
<p>For getting more information on the topic I highly recommend reading this interesting article: <a  href="http://www.rackspacecloud.com/blog/2009/11/09/nosql-ecosystem/">NoSQL Ecosystem</a></p>

### Can I use a NoSQL database to store GIS data?
<p>In the last months I have progressively been getting interested in this topic and decided to test a way to store and read GIS data in a NoSQL database.</p>
<p>Managing GIS data with NoSQL in circumstances where performances and scalability are a major issue could be the way for the win.</p>
<p>To do so I have decided to use MongoDb, because of its nice Python API and its rich query expressions syntax.
But note that it should be very easy to reproduce my experiment with other NoSQL databases.</p>
<p>To manage GIS data it was not difficult to decide to use GDAL/OGR, the popular (and beauty!) FOSS toolkit for GIS developers.</p>
<p>I am not sure if documenting my experiments may be useful for anyone, but I have decided to assemble this tutorial, at least for documenting my experience if later I will need to consider for production a NoSQL technology.</p>
<p>Please note that there have been already several attempts to manage GIS data with NoSQL database (and surely I am missing some other ones so please feel free to add a comment or email me about this):</p>
<ul >
<li>ZODB mainly with <a  href="http://zmapserver.sourceforge.net/ZCO/index.html">Cartographic Objects for Zope</a>, by <a  href="http://sgillies.net">Sean Gillies</a></li>
<li>FeatureServer is using BerkleyDB datasource for storage</li>
</ul>

### Concepts and software behind this tutorial
<ul >
<li><a  href="http://www.mongodb.org/display/DOCS/Home">MongoDb</a> is a scalable, high-performance, open source, schema-free, document-oriented database</li>
<li>PyMongo: a Python API for working with MongoDB(<a  href="http://api.mongodb.org/python/1.1.2%2B/index.html">http://api.mongodb.org/python/1.1.2%2B/index.html</a>)</li>
<li>GDAL (OGR): the famous Geospatial Data Abstraction Library (we will use the OGR part, that is for vectorial data) (<a  href="http://www.gdal.org/">http://www.gdal.org/</a>)</li>
</ul>

### Start MongoDb
<ul>
<li>download and install MongoDb following the instructions on MongoDb web site
</li>
<li>start MongoDb in background:
<pre lang="text">
paolo@paolo-laptop:~$ ./training/mongodb-linux-i686-2009-10-14/bin/mongod
</pre>
</li>
</ul>

### And now the code!
<p>For using the code in this tutorial I have used this shapefile: <a  href="http://www.census.gov/geo/cob/bdy/co/co00shp/co99_d00_shp.zip">Census usa counties 2000</a>.
Download it if you want to follow this sample step by step.</p>
<p>Now it is time to run some code. Just copy and paste in a file the following scripts and execute it:</p>

{% highlight python %}
import os
from pymongo.connection import Connection
from progressbar import ProgressBar
from osgeo import ogr

def shape2mongodb(shape_path, mongodb_server, mongodb_port, mongodb_db, mongodb_collection, append, query_filter):
        """Convert a shapefile to a mongodb collection"""
        print '*** Converting a shapefile to a mongodb collection ***'
        driver = ogr.GetDriverByName('ESRI Shapefile')
        print 'Opening the shapefile %s...' % shape_path
        ds = driver.Open(shape_path, 0)
        if ds is None:
                print 'Can not open', ds
                sys.exit(1)
        lyr = ds.GetLayer()
        totfeats = lyr.GetFeatureCount()
        lyr.SetAttributeFilter(query_filter)
        print 'Starting to load %s of %s features in shapefile %s to MongoDB...' % (lyr.GetFeatureCount(), totfeats, lyr.GetName())
        print 'Opening MongoDB connection to server %s:%i...' % (mongodb_server, mongodb_port)
        connection = Connection(mongodb_server, mongodb_port)
        print 'Getting database %s' % mongodb_db
        db = connection[mongodb_db]
        print 'Getting the collection %s' % mongodb_collection
        collection = db[mongodb_collection]
        if append == False:
                print 'Removing features from the collection...'
                collection.remove({})
        print 'Starting loading features...'
        # define the progressbar
        pbar = ProgressBar(maxval=lyr.GetFeatureCount()).start()
        k=0
        # iterate the features and access its attributes (including geometry) to store them in MongoDb
        feat = lyr.GetNextFeature()
        while feat:
                mongofeat = {}
                geom = feat.GetGeometryRef()
                mongogeom = geom.ExportToWkt()
                mongofeat['geom'] = mongogeom
                # iterate the feature's  fields to get its values and store them in MongoDb
                feat_defn = lyr.GetLayerDefn()
                for i in range(feat_defn.GetFieldCount()):
                        value = feat.GetField(i)
                        if isinstance(value, str):
                                value = unicode(value, 'latin-1')
                        field = feat.GetFieldDefnRef(i)
                        fieldname = field.GetName()
                        mongofeat[fieldname] = value
                # insert the feature in the collection
                collection.insert(mongofeat)
                feat.Destroy()
                feat = lyr.GetNextFeature()
                k = k + 1
                pbar.update(k)
        pbar.finish()
        print '%s features loaded in MongoDb from shapefile.' % lyr.GetFeatureCount()

def mongodb2shape(mongodb_server, mongodb_port, mongodb_db, mongodb_collection, output_shape, query_filter):
        """Convert a mongodb collection (all elements must have same attributes) to a shapefile"""
        print '*** Converting a mongodb collection to a shapefile ***'
        connection = Connection(mongodb_server, mongodb_port)
        print 'Getting database MongoDB %s...' % mongodb_db
        db = connection[mongodb_db]
        print 'Getting the collection %s...' % mongodb_collection
        collection = db[mongodb_collection]
        print 'Exporting %s elements in collection to shapefile...' % collection.count()
        drv = ogr.GetDriverByName("ESRI Shapefile")
        ds = drv.CreateDataSource(output_shape)
        lyr = ds.CreateLayer('test', None, ogr.wkbPolygon)
        print 'Shapefile %s created...' % ds.name
        cursor = collection.find(query_filter)
        # define the progressbar
        pbar = ProgressBar(collection.count()).start()
        k=0
        # iterate the features in the collection and copy them to the shapefile
        # for simplicity we export only the geometry to the shapefile
        # if we would like to store also the other fields we should have created a metadata element with fields datatype info
        for element in cursor:
                element_geom = element['geom']
                feat = ogr.Feature(lyr.GetLayerDefn())
                feat.SetGeometry(ogr.CreateGeometryFromWkt(element_geom))
                lyr.CreateFeature(feat)
                feat.Destroy()
                k = k + 1
                pbar.update(k)
        pbar.finish()
        print '%s features loaded in shapefile from MongoDb.' % lyr.GetFeatureCount()

# delete otuput shape if it exists
input_shape = '/home/paolo/training/mongodb/shape/usa/co99_d00.shp'
output_shape = '/home/paolo/training/mongodb/shape/test/output.shp'
driver = ogr.GetDriverByName('ESRI Shapefile')
if os.path.exists(output_shape):
        driver.DeleteDataSource(output_shape)
# connection information
mongodb_server = 'localhost'
mongodb_port = 27017
mongodb_db = 'gisdb'
mongodb_collection = 'usa_counties'
# 1. first we import features from shapefile to mongodb
print 'Importing data to mongodb...'
shape2mongodb(input_shape, mongodb_server, mongodb_port, mongodb_db, mongodb_collection, False, 'STATE in (40,41,42)')
# 2. then we export features from mongodb to shapefile
print 'Exporting data from mongodb...'
mongodb2shape(mongodb_server, mongodb_port, mongodb_db, mongodb_collection, output_shape, {"STATE": "40"})
# 3. now some test with mongodb
connection = Connection(mongodb_server, mongodb_port)
print 'Getting database MongoDB %s' % mongodb_db
db = connection[mongodb_db]
print 'Getting the collection %s' % mongodb_collection
collection = db[mongodb_collection]
# counting the collection
print 'Elements in collection: %s' % collection.count()
# finding one feature
feature = collection.find_one()
print 'Here is one random feature that has been stored:'
print feature
# some query now
print 'There are %s counties in STATE = 40' % collection.find({"STATE": "40"}).count()
print 'There are %s counties in STATE = 40 and AREA > 0.5' % collection.find({"STATE": "40", "AREA": {"$gt": 0.5}}).count()
{% endhighlight %}

<p>code is deeply commented, so understanding what is going on should be simple.</p>
<p>Basically:</p>
<ul >
<li>for accessing the MongoDB entities I am using the PyMongo API</li>
<li>for accessing the shapefile features I am using the OGR API</li>
<li>to import a shapefile to a MongoDB collection, I iterate using OGR all the features of the shapefile and I copy them in the MongoDb collection. Note that I use a query filter (in a SQL form) if I do not want to copy all the features</li>
<li>to export a shapefile from a MongoDb collection, I iterate using PyMongo all the documents of the database and I copy them in a new shapefile. If I do not want to export to the shapefile the whole collection I may provide a query filter (in a NoSQL form)</li>
<li>finally I provide some query on the features stored in MongoDb</li>
</ul>
<p>This is the output at my console:</p>

{% highlight bash %}
(mongodb)paolo@paolo-laptop:~/training/mongodb/geomongo$ python geomongodb.py
Importing data to mongodb...
*** Converting a shapefile to a mongodb collection ***
Opening the shapefile /home/paolo/training/mongodb/shape/usa/co99_d00.shp...
Starting to load 181 of 3489 features in shapefile co99_d00 to MongoDB...
Opening MongoDB connection to server localhost:27017...
Getting database gisdb
Getting the collection usa_counties
Removing features from the collection...
Starting loading features...
100% |##########################################################################################################################|
181 features loaded in MongoDb from shapefile.
Exporting data from mongodb...
*** Converting a mongodb collection to a shapefile ***
Getting database MongoDB gisdb...
Getting the collection usa_counties...
Exporting 181 elements in collection to shapefile...
Shapefile /home/paolo/training/mongodb/shape/test/output.shp created...
100% |##########################################################################################################################|
77 features loaded in shapefile from MongoDb.
Getting database MongoDB gisdb
Getting the collection usa_counties
Elements in collection: 181
Here is one random feature that has been stored:
{u'PERIMETER': 2.5612567807393698, u'NAME': u'Chester', u'AREA': 0.207426571347607, u'LSAD': u'06', u'CO99_D00_': 1324.0, u'CO99_D00_I': 1323.0, u'COUNTY': u'029', u'LSAD_TRANS': u'County', u'STATE': u'42', u'geom': u'POLYGON ((-75.674975000000003 40.242876000000003,-75.674888999999993 40.242919,
.........
.........
,-75.674975000000003 40.242876000000003))', u'_id': ObjectId('4b1aef531c900d23a900005f')}
There are 77 counties in STATE = 40
There are 2 counties in STATE = 40 and AREA > 0.5
{% endhighlight %}

### Useful resources
<p>I have found useful this blog post about <a  href="http://gissolved.blogspot.com/2009/05/populating-mongodb-with-pois.html">using MongoDb with Esri ArcGis</a></p>
<p>About MongoDb I have found these useful blog posts:</p>
<ul >
<li><a  href="http://blog.mongodb.org/post/142940558/what-is-the-right-data-model">What is the right data model?</a></li>
<li><a  href="http://blog.mongodb.org/post/119945109/why-schemaless">Why schemaless?</a></li>
</ul>
<p>Also be sure to read the documentation of the API and these tutorials for gaining more info about PyMongo and GDAL/OGR:</p>
<ul >
<li><a  href="http://api.mongodb.org/python/1.1.2%2B/index.html">PyMongo API</a></li>
<li><a  href="http://api.mongodb.org/python/1.1.2%2B/tutorial.html">PyMongo Tutorial</a></li>
<li><a  href="http://trac.osgeo.org/gdal/wiki/GdalOgrInPython">GDAL/OGR in Python</a></li>
</ul>
