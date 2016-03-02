---
categories: GIS, PostGIS, devs, Windows, Ubuntu, Apache, FeatureServer, QGIS, uDig,
  gvSIG, Python, WFS, WMS
date: 2008/02/21 22:10:14
guid: http://www.paolocorti.net/public/wordpress/index.php/2008/02/21/a-day-with-featureserver-1/
permalink: http://www.paolocorti.net/2008/02/21/a-day-with-featureserver-1/
tags: GIS, PostGIS, devs, Windows, Ubuntu, Apache, FeatureServer, QGIS, uDig, gvSIG,
  Python, WFS, WMS
title: 'A day with FeatureServer #1'
---
Some friends already spoke me well about FeatureServer by MetaCarta in the last weeks, so I already was waiting for having a bit of time to get started with it. Then James <a href="http://www.spatiallyadjusted.com/2008/02/05/bringing-open-source-gis-into-an-esri-shop/">posted this on his blog</a>, and my curiosity was definitely fired.

So I decided to spend a day for installing and testing it, without thinking of the lack of documentation (FeatureServer is still a young project, so no wonder here if the only way to get infos is <a href="http://www.python.org/">digging in the source code</a> and <a href="http://featureserver.org/pipermail/featureserver/2008-February/000184.html">posting</a> <a href="http://featureserver.org/pipermail/featureserver/2008-February/000196.html">to the</a> <a href="http://featureserver.org/pipermail/featureserver/">mailing list</a>). The day I considered to spend on it then spawned to more and more hours that I could imagine, and given my actually very busy schedule at my job, I had to find free hours during the night and the weekend. I then decided to write this post to help people in getting started with FeatureServer in a quicker way that was for me.

FeatureServer is a simple and powerfull RESTful-Pythonic WFS server. 
Only from this last sentence there are 3 very important things that made me like (and you should - also) FeatureServer before even getting started with it:

<ul>
<li>Its <a href="http://en.wikipedia.org/wiki/Representational_State_Transfer">RESTful</a> architecture</li>
<li>It is written in Python, and having chosen Plone as our CMS here at my office I am starting to like this language very much</li>
<li>I truly believe that WFS is the way to go for remotely editing GIS data</li>
</ul>

Now, before showing you how to set up a simple FeatureServer environment with Ubuntu (but also if you are using Windows - shame on you! :-) - you should not have difficulties to follow the steps), I will list some FeatureServer's limitations I came in (if some of the developers are reading this post and find out something wrong please let me know):

<ul>
<li>At this time FeatureServer doesn't let you to reproject your data - at least if you use PostGIS direct connect (not sure thugh if this is also for OGR, it sholdn't because OGR supports projections, but I couldn't figure it out how to pass the srid). At least this is the conclusion I came digging in the source code.
So If you want to display your data togheter with others WFS and WMS layers, be sure to be consistent with the srid of your layers.
A nice way to go without recreating the data in a different srid, if you are using PostGIS, is to create a view for each layer and projecting your data from the view using the st_transform function over your geometry columns. Be aware, anyway, that if you opt for this way, your WFS layer won't be able to edit your data.
If you need editing functionalities the only way to go is to physically reproject your data.</li>
<li>
You cannot add PostGIS data sources with a schema different than "public". This looks to be an easy implementable feature, I wish I could be more proficient with Python for giving to the project a patch, maybe in the next weeks I will give a try ;-)
</li>
<li>According to the <a href="http://featureserver.org/pipermail/featureserver/2008-February/000197.html">answer in the mailing list</a> from <a href="http://crschmidt.net/blog/">Christopher Schmidt</a>, FeatureServer does NOT implement the WFS getCapabilities() interface, this meaning that at this time we will not be able to add a layer served from FeatureServer to desktop clients like QGIS, uDig and gvSIG.
This is because the outstanding FeatureServer architecture supports data sources without a fixed schema (DBM, BerkleyDB, SQLite, OSM).
For data sources with a fixed schema (OGR, PostGIS, WFS, Flickr, Twitter) Christopher suggested me to provide a patch implementing DataSource-level "schema" method. Once again, I wish I could be a Python guru, but I am still far from that, unluckily, otherwise I would be happy to help ;-) This doesn't meaning that I will not try to give a try in the next weeks!
</li>
<li>Only Point, Line and Polygon OGC simple geometries are supported for editing. Multigeometries feature editing will give different results depending from the datasources you are using.
For example at this time editing a multigeometry feature in a OGR data source (for example for adding shapefile layers to your WFS) will result in truncating the geometry to the first part. Editing a feature in a PostGIS data source will thrown an error saying that multigeometry editing is not supported for PostGIS layers in FeatureServer.
So if for example you have - like me - PostGIS layers with a MultiPolygon geometrytype you should consider to recreate them with a Polygon geometrytype (if - of course - there is a way without compromising your data).
</li>
</ul>

Having said so, times to make hands dirty, and let's go straight installing FeatureServer and testing it with the amazing edits-enabled demo.

We will install FeatureServer from the SVN repository, this will make you sure to have the latest version where some bugs were repaired from the actual stable version.
You just need to type in your command prompt (you need to have installed the SVN client):

<pre lang="text">
svn co  http://svn.featureserver.org/trunk
</pre>

Now that you have the latest version, you need to configure FeatureServer. According to the few docs you will find under the doc directory, you may install FeatureServer to run under Python CGI, mod_python, or as a standalone server (Windows users: don't forget to have installed Python 2.4, that for Ubuntu users is installed by default).

At this time i have tried the first way (Python CGI) and this is as I have configured Apache httpd (if you want to replicate this configuration don't forget to add a featureserver host in the hosts file):

$$code(lang=xml)
<VirtualHost *:80>
	ServerName featureserver
	DocumentRoot /home/corti/public_html/featureserversvn/trunk/featureserver

	<Directory />
		Options FollowSymLinks
		AllowOverride All
		AddHandler cgi-script .cgi
		Options +ExecCGI
	</Directory>

	ErrorLog /var/log/apache2/featureserver_error.log.
	LogLevel warn
	CustomLog /var/log/apache2/featureserver_access.log combined
	ServerSignature On
</VirtualHost>
$$/code

You will need to install some Python libraries, depending on what DataSources and what Services you want to use.
For me it was enough to install the followings (Windows users, you will need to download the installers from the website and install it or use the <a href="http://pypi.python.org/pypi">Cheese Shop</a>):

<pre lang="txt">
install GDAL, simplejson and cheetah:
sudo apt-get install python-gdal
sudo apt-get install python-simplejson
sudo apt-get install python-cheetah
sudo apt-get install python-psycopg
</pre>

<a href="http://www.gdal.org/">GDAL</a> is mandatory if you want to WFSfy <a href="http://www.gdal.org/ogr/ogr_formats.html">OGR datasources</a> (like shapefiles, for example and PostGIS without a direct-connect).

<a href="http://svn.red-bean.com/bob/simplejson/tags/simplejson-1.3/docs/index.html">Simplejson</a> is a must if you want to use <a href="http://json.org/">JSON</a> (the default) and <a href="http://wiki.geojson.org">GeoJSON</a> FeatureServer services. Basically it is a JSON encoder/decoder. 
JSON is a lightweight data-interchange format. You may wonder why using it instead than going via ubiquitous XML. Well, there <a href="http://www.b-list.org/weblog/2006/dec/21/i-cant-believe-its-not-xml/">are</a> <a href="http://blogs.msdn.com/mikechampion/archive/2006/12/21/the-json-vs-xml-debate-begins-in-earnest.aspx">many</a> <a href="http://www.infoq.com/news/2006/12/json-vs-xml-debate">reasons</a>.
GeoJSON is a JSON data-interchange with geographic content encoded.

<a href="http://www.cheetahtemplate.org/">Cheetah</a> is an OS template engine and code generation tool, written in Python. You will need it if you want to use FeatureServer HTML services.

<a href="www.initd.org">psycopg</a> is a PostgreSQL database adapter for Python, and it will probably already be installed in your box if you have ever tried to connect to PostgreSQL via Python (for me it was already there, having needed it for connecting to Postgres from Zope).

After finishing installing all this components, you will finally be able to access to FeatureServer. FeatureServer will WFSfy your layers via a Service (default JSON). Which layers and what service can be configured in the featureserver.cfg file (in FeatureServer root).
If you want to make a quick try with the demo embedded in what you have dowloaded from the FeatureServer SVN repository, just open your featureserver.cfg:

<pre lang="txt">
# Metadata section allows you to define the default
# service type to be created, and location for error logging
[metadata]
default_service=JSON
error_log=/home/corti/public_html/featureserversvn/trunk/featureserver/error.log

# each additional section is a 'layer', which can be accessed.
# see DataSources.txt for more info on configuring.
[scribble]
type=DBM
file=/tmp/featureserver.scribble
gaping_security_hole=yes
</pre>

In this file there is JSON service, and just one layer is served, named scribble. The data source for this layer is DBM, that uses <a href="http://www.webthing.com/software/AnyDBM/">AnyDBM</a> combined with <a href="http://docs.python.org/lib/module-pickle.html">pickle</a> to store features in a file on disk. What you need to do here is to change the file path to a directory to your disk where to write the featureserver.scribble (or whatever you name it) file. Note that this directory should be  writable from the httpd users (typically www-data).

Now time to test your installation, using this DBM data source. First make sure everything is working by accessing the scribble layer and getting is gml, just type:

http://featureserver/featureserver.cgi/scribble/all.gml

you should get an empty wfs:FeatureCollection tree, as far as you still didn't use the demo application.
Now time to access the demo application (an OpenLayers application). Just go to: 

http://featureserver

try to add some geometries (note that DBM is a multigeometry layer) and then try again to get the gml (or json or geojson or atom or whatever) of your layer. You will get your dataset, with as many features as many you have added from the demo.
Note that you can choose your format appending the format to your request (example /scribble/all.gml).
The url format to query a layer will always be:
http://featureserver/featureserver.cgi/layername/id.format

where id is the id of the feature you want to get (or "all" if you want to get the whole set) and format is the format you want to get (gml, kml, atom, JSON, GeoJSON).
You can append query parameters to your url to filter the results (for example with a spatial box) , as explained in the Querying.txt file under the doc directory.

FeatureServer as you have seen from the demo will let you to edit your data: let's try some editing using <a href="http://curl.haxx.se/">curl</a> (as suggested from the FeatureServer home page). Curl is a magnificent OS tool for or transferring files with URL syntax, supporting FTP, FTPS, HTTP, HTTPS and many others, very usefull also to test web applications. Ubuntu users have it installed by default, Windows users will need to download it and install it.

What we will do is to delete a feature (id=2), then insert it again:

before deleting let's save the feature in json:
http://featureserver/featureserver.cgi/scribble/2.json

now we will delete it:

<pre lang="txt">
curl -X DELETE http://featureserver/featureserver.cgi/scribble/2.json
</pre>

now we will re-insert it:

<pre lang="txt">
curl -d @2.json  http://featureserver/featureserver.cgi/scribble/create.json
</pre>

I truly believe FeatureServer is really brilliant stuff, and you will have even a better idea next time when we will take to the party your datasets (like shapefile and PostGIS layer) and the magnificent OpenLayers. Until then: happy WFSs with FeatureServer!






 





















