---
categories: Uncategorized, GIS, PostGIS, devs, Windows, Ubuntu, FeatureServer, Python,
  WMS, OpenStreetMap, Web2.0, Flickr, Twitter, OpenLayers, WFS
date: 2008/05/03 02:13:47
guid: http://www.paolocorti.net/public/wordpress/index.php/2008/05/03/a-day-with-featureserver-2/
permalink: http://www.paolocorti.net/2008/05/03/a-day-with-featureserver-2/
tags: Uncategorized, GIS, PostGIS, devs, Windows, Ubuntu, FeatureServer, Python, WMS,
  OpenStreetMap, Web2.0, Flickr, Twitter, OpenLayers, WFS
title: 'A day with FeatureServer #2'
---
In the <a href="http://www.paolocorti.net/public/wordpress/index.php/2008/02/21/a-day-with-featureserver-1/">previous post</a> we have seen an introduction to FeatureServer, and we were just playing with the base edit sample, with the scribble layer.
Now it is time to use FeatureServer with our datasets: I am assuming that you will want to create FeatureServer services for shapefiles, PostGis layers, OpenStreetMap, Twitter and Flickr.

<a href='http://www.paolocorti.net/public/wordpress/wp-content/uploads/2008/05/adaywithfeatureserver.png'><img src="http://www.paolocorti.net/public/wordpress/wp-content/uploads/2008/05/adaywithfeatureserver.png" alt="" title="the nice OpenLayers interface with the Yahoo Map WMS and some data source from FeatureServer" width="150" height="112" class="alignnone size-thumbnail wp-image-80" /></a>

<strong>Data preparation for this demo</strong>

<strong>1) shapefiles</strong>
If you want to follow my steps, you can download the sample shapefiles of New York I was using to assemble this demo <a href=" http://arcdata.esri.com/data/tiger2000/tiger_download.cfm">here</a>: select New York as a state and then as a county, and download the Block Groups and the Roads, a MULTIPOLYGON and MULTILINESTRING shapefiles).
I have copied (an extract) of this shapefiles to the /home/corti/training/featureserver/data directory.

<strong>2) PostGIS</strong>
I have then imported the same datasets in PostGIS with the following procedure:

<pre lang="text">
corti@corti-ubuntu:~/training/featureserver/data/newyork$ shp2pgsql -I -s 4326 blocks.shp blocks4326 > blocks4326.sql
Shapefile type: Polygon
Postgis type: MULTIPOLYGON[2]

corti@corti-ubuntu:~/training/featureserver/data/newyork$ shp2pgsql -I -s 4326 streets.shp streets4326 > streets4326.sql
Shapefile type: Arc
Postgis type: MULTILINESTRING[2]
</pre>

then load them into postgis:

<pre lang="text">
corti@corti-ubuntu:~/training/featureserver/data/newyork$ psql -d gisdb -h localhost -U gis -f streets4326.sql
corti@corti-ubuntu:~/training/featureserver/data/newyork$ psql -d gisdb -h localhost -U gis -f blocks4326.sql 
</pre>

<strong>3) Open Street Map</strong>
If you are interested in GIS and free data you must have heard of OpenStreetMap (OSM). Basically is a collaborative portal for creating and mantaning geodata, a sort of wikipedia for GIS.

With FeatureServer you can create an OSM service, but you must note that only query by bbox (envelope) or id are supported, like these:

<pre lang="text">
http://featureserver/featureserver.cgi/osm/all.gml?bbox=-124.1,47.2,-123.9,47.5
http://featureserver/featureserver.cgi/osm/8551622.gml
</pre>

<strong>4) Flickr</strong>
To play this whole demo you should have access to an account (yours or some other's with public photos in the New York area we are using) for Flickr. 
Flickr is an outstanding photos management web 2.0 application you may already know, if not it is worth to try it (you can have a free account if you are not wanting to upload thousands of pictures per month).
Obviously the photos must be geotagged (= location fields in the EXIF files must be not empty), so after uploading some pictures you will have 4 ways to go:
<ul>
<li>
You are a very lucky owner of a GPS-enabled camera, that will automatically geotag your photos, like <a href="http://www.spatiallyadjusted.com/2007/01/18/ricoh-releases-new-gps-ready-digital-camera/">this one</a>
</li> 
<li>
You may buy a GPS to integrate with your camera to obtain geotagged photos (generally you may need some kind of software that interpolate your GPS route or points and the photos using the time at which both point and photos are taken). Or some producer is able to connect their camera at external GPS, and automatically get the location for EFIX, like <a href="http://www.canon.co.jp/imaging/wft/wft-e2/manual/gps/index.html">this one from Canon</a>
</li> 
<li>
Last (but not least in many cases, specially for your old pictures) you may still manually geotag your photos, by using the map application in the Flickr web site (or many other desktop and web applications out there)
</li> 
</ul>

The way to access to all your geotagged photos from FeatureServer is like usual:
<pre lang="text">
http://featureserver/featureserver.cgi/flickr/all.gml
</pre>

<strong>5) Twitter</strong>
Twitter is a micro-blogging platform that is gaining big popularity nowadays. Many IT (and GIS) people is actually twittering, and I am also victim of this new mania (btw: if you are interested and want to follow me <a href="http://twitter.com/capooti">I am here</a>). To be honest i initially thought to be a stupid thing, but after I realized is a very good way to exchange opinions and suggestions with <a href="http://twitter.com/codinghorror">many</a> <a href="http://twitter.com/geobabbler">many</a> <a href="http://twitter.com/xanadont">many</a><a href="http://twitter.com/acangiano"> many</a> <a href="http://twitter.com/crschmidt">many</a> <a href="http://twitter.com/TheSteve0">many</a> <a href="http://twitter.com/cageyjames">experts</a> <a href="http://twitter.com/dbouwman">in</a> <a href="http://twitter.com/ajturner">the</a> <a href="http://twitter.com/D_Guidi">community</a>, so I decided not to loose this opportunity.
Anyway, if you don't have your own account and don't want to create one, you just need to know the name of an account actually located in the New York zone we are using).

Once you add a twitter service in FeatureServer, this is the way to call it (it will actually show only one feature - I didn't find a way to add multiple twitter users for the same service, neither I think there is this way):

<pre lang="text">
http://featureserver/featureserver.cgi/twitter/all.gml
</pre>

Be aware that the twitter user you are serving must have a location! A good application showing the location of a user is <a href=" http://twittermap.com/maps">this one</a> (type in the textbox the name of the twitter user you want to know where it is):

if you want to update your twitter locations, you have to use <a href="http://twittermap.com/maps/faq.html">this format</a>  and you can do that in many way, from any way you can send a message (so: by the web interface, by an sms with your mobile, by some twitter software like Twitux in Linux or whatever, by an instant message client like Pidgin in Linux or whatever, etc...)

You may update your location also using CURL, for example when I come back home from some holidays, i would type:

<pre lang="text">
curl -X PUT  -u username:password -d location="Via Berto, Rome, Italy" http://twitter.com/account/update_location.json
</pre>

If you want to know more about twitter and its location API, a good place to start is <a href="http://highearthorbit.com/twitter-location-api/">this nice post with useful info from Andrew Turner</a>.

<strong>The configuration file</strong> 

this is the featureserver.cfg file i am using (for more info look at the datasources.txt in the doc directory of FeatureServer): you can copy this and make some little modifications (the path to the shapefiles, the connection string for PostGis, etc...)

<pre lang="text">
# Metadata section allows you to define the default
# service type to be created, and location for error logging
[metadata]
default_service=JSON
error_log=/home/corti/tmp/featureserver/error.log

#sample scribble layer (used in #1)
[scribble]
type=DBM
file=/tmp/featureserver.scribble
gaping_security_hole=yes

#shapefile 1
[blocks]
type=OGR
dsn=/home/corti/training/featureserver/data/newyork/blocks.shp
layer=blocks

#shapefile 2
[streets]
type=OGR
dsn=/home/corti/training/featureserver/data/newyork/streets.shp
layer=streets

#postgis layer 1
[blocksPG]
type=PostGIS
dsn=host=localhost dbname=mygisdb user=myuser password=mypassword
layer=blocks4326
fid=gid
geometry=the_geom
srid=4326

#postgis layer 2
[streetsPG]
type=PostGIS
dsn=host=localhost dbname=mygisdb user=myuser password=mypassword
layer=streets4326
fid=gid
geometry=the_geom
srid=4326

#open street map
[osm]
type=OSM
osmxapi=no

#flickr
[flickr]
type=Flickr
user_id=12345678@N06
#tags=newyork

#twitter
[twitter]
type=Twitter
username=anytwitteruser
</pre>

<strong>Time to test the FeatureServer data sources </strong>
To see if everything is ok with the data sources we want to use with FeatureServer, you can try to query some feature on each of them, for example (you should get a feature gml for each of this requests):

<pre lang="text">
shapefiles
blocks: http://featureserver/featureserver.cgi/blocks/21.gml
streets: http://featureserver/featureserver.cgi/streets/21.gml

postgis
blocksPG: http://featureserver/featureserver.cgi/blocksPG/21.gml
streetsPG: http://featureserver/featureserver.cgi/streetsPG/21.gml

osm: http://featureserver/featureserver.cgi/osm/8551622.gml

flickr: http://featureserver/featureserver.cgi/flickr/2448905405.gml

twitter: http://featureserver/featureserver.cgi/twitter/all.gml
</pre>

<strong>Taking OpenLayers to the party</strong>

Now we will use the outstanding OpenLayers for testing our FeatureServer datasources, together with some WMS that OpenLayers can easily read (MetaCarta, GoogleMaps, VirtualEarth, YahooMaps, WorldWind).

We will use this html page: you need to place it in your web server:

$$code(lang=html)
<html>
<head>
  <title>OpenLayers Example: a day with FeatureServer</title>
    <!-- OpenLayers API -->
    <script src="OpenLayers-2.5/OpenLayers.js"></script>
    <!-- GoogleMaps API: you need a key for it! -->
    <script src="http://maps.google.com/maps?file=api&amp;v=2&amp;key=xxxplaceyourkeyherexxx"
	type="text/javascript"></script>
    <!-- VirtualEarth API -->
    <script src='http://dev.virtualearth.net/mapcontrol/v3/mapcontrol.js'></script>
    <!-- Yahoo Maps API -->
    <script src="http://api.maps.yahoo.com/ajaxymap?v=3.0&appid=euzuro-openlayers"></script>
    <script type="text/javascript">
         var map, drawControls, geojson, lastFeature, mywfs, vectors, featureid;
	function init(){
		//create the map
		var map = new OpenLayers.Map('map', {'maxResolution': .28125, tileSize: new OpenLayers.Size(512, 512)});
		//create the WMS layers (metacarta basic, metacarta satellite, 
		//google hybrid, google satellite, virtualearth, yahoo, worldwind bathy, worldwind landsat
		var olwms = new OpenLayers.Layer.WMS( "OpenLayers WMS basic", "http://labs.metacarta.com/wms-c/Basic.py", {'layers':'basic'}); 
		var olwms2 = new OpenLayers.Layer.WMS( "OpenLayers WMS satellite", "http://labs.metacarta.com/wms-c/Basic.py", {'layers':'satellite'}); 
		var google = new OpenLayers.Layer.Google( "Google Hybrid Map", { type: G_HYBRID_MAP } );
		var google2 = new OpenLayers.Layer.Google( "Google Satellite Map", { type: G_SATELLITE_MAP } );
		var virtualearth = new OpenLayers.Layer.VirtualEarth( "Virtual Earth Hybrid", {type: VEMapStyle.Hybrid });
		var yahoo = new OpenLayers.Layer.Yahoo("Yahoo");
		var ww = new OpenLayers.Layer.WorldWind( "WorldWind Bathy", "http://worldwind25.arc.nasa.gov/tile/tile.aspx?", 36, 4, {T:"bmng.topo.bathy.200406"});
		var  ww2 = new OpenLayers.Layer.WorldWind( "WorldWind LANDSAT", "http://worldwind25.arc.nasa.gov/tile/tile.aspx", 2.25, 4,{T:"105"});
		//create the WFS layers
		//1. shapefiles WFS layers
		var wfsNYblocksSHP = new OpenLayers.Layer.WFS("WFS New York blocks (shp)", "featureserver.cgi/blocks?format=WFS", 
			{maxfeatures: "300"}, {extractAttributes: true, displayInLayerSwitcher: true});
		var wfsNYstreetsSHP = new OpenLayers.Layer.WFS("WFS New York streets (shp)", "featureserver.cgi/streets?format=WFS", 
			{maxfeatures: "300"}, {extractAttributes: true, displayInLayerSwitcher: true});
		//2. postgis WFS layers (with a different style than default)
		var wfsNYblocksPG = new OpenLayers.Layer.WFS("WFS New York blocks (postgis)", "featureserver.cgi/blocksPG?format=WFS", 
			{maxfeatures: "300"}, {extractAttributes: true, displayInLayerSwitcher: true});
		wfsNYblocksPG.style = {fillColor:'#FF0000', fillOpacity: 0.2, strokeColor: "#FF0000", strokeOpacity: 1, strokeWidth: 1};
		var wfsNYstreetsPG = new OpenLayers.Layer.WFS("WFS New York streets (postgis)", "featureserver.cgi/streetsPG?format=WFS", 
			{maxfeatures: "300"}, {extractAttributes: true, displayInLayerSwitcher: true});
		wfsNYstreetsPG.style = {fillColor:'#FF0000', fillOpacity: 0.2, strokeColor: "#FF0000", strokeOpacity: 1, strokeWidth: 1};
		//3. osm WFS layer
		var wfsOSM = new OpenLayers.Layer.WFS("OpenStreetMap", "featureserver.cgi/osm?format=WFS", 
			{maxfeatures: "5"}, {extractAttributes: true, displayInLayerSwitcher: true});
		//4. flickr WFS layer
		var wfsFlickr = new OpenLayers.Layer.WFS("New York Flickrs", "featureserver.cgi/flickr?format=WFS", 
			{maxfeatures: "300"}, {extractAttributes: true, displayInLayerSwitcher: true});
		//5. twitter WFS layer
		var wfsTwitter = new OpenLayers.Layer.WFS("My Twitter", "featureserver.cgi/twitter?format=WFS", 
			{maxfeatures: "300"}, {extractAttributes: true, displayInLayerSwitcher: true});
		//add the layers to the map
		map.addLayers([wfsTwitter,wfsFlickr,wfsOSM,wfsNYstreetsPG,wfsNYblocksPG,wfsNYstreetsSHP, wfsNYblocksSHP,
			yahoo, virtualearth, olwms, olwms2, google, google2, ww, ww2]);
		//add map controls
		map.addControl(new OpenLayers.Control.Navigation());
		map.addControl(new OpenLayers.Control.PanZoomBar());
		map.addControl(new OpenLayers.Control.MousePosition());
		map.addControl(new OpenLayers.Control.Permalink());
		map.addControl(new OpenLayers.Control.LayerSwitcher());
		//set map center
		map.setCenter(new OpenLayers.LonLat(-73.9818, 40.712), 13);
	}
    </script>
    </head>
    <body  onload="init()">
      <div style="width:100%; height:100%" id="map"></div>
      <script defer="defer" type="text/javascript">
    </body>
</html>
$$/code

note the java api we are using for accessing the various WMS!

what is happening behind the scenes can be easily understood navigating this page with the use of Firebug to detect which GET request are issued by OpenLayers javascript:

<pre lang="text">
flickr:
http://featureserver/featureserver.cgi/flickr?format=WFS&maxfeatures=300&SERVICE=WFS&VERSION=1.0.0&REQUEST=GetFeature&SRS=EPSG%3A4326&BBOX=-74.15771484375,40.66379060988484,-73.80615234375,40.76007871037859

twitter:
http://featureserver/featureserver.cgi/twitter?format=WFS&maxfeatures=300&SERVICE=WFS&VERSION=1.0.0&REQUEST=GetFeature&SRS=EPSG%3A4326&BBOX=-74.15771484375,40.66379060988484,-73.80615234375,40.76007871037859

open street map
http://featureserver/featureserver.cgi/osm?format=WFS&maxfeatures=5&SERVICE=WFS&VERSION=1.0.0&REQUEST=GetFeature&SRS=EPSG%3A4326&BBOX=-74.15771484375,40.66379060988484,-73.80615234375,40.76007871037859

postgis
http://featureserver/featureserver.cgi/streetsPG?format=WFS&maxfeatures=300&SERVICE=WFS&VERSION=1.0.0&REQUEST=GetFeature&SRS=EPSG%3A4326&BBOX=-74.15771484375,40.66379060988484,-73.80615234375,40.76007871037859
http://featureserver/featureserver.cgi/blocksPG?format=WFS&maxfeatures=300&SERVICE=WFS&VERSION=1.0.0&REQUEST=GetFeature&SRS=EPSG%3A4326&BBOX=-74.15771484375,40.66379060988484,-73.80615234375,40.76007871037859

shapefile
http://featureserver/featureserver.cgi/streets?format=WFS&maxfeatures=300&SERVICE=WFS&VERSION=1.0.0&REQUEST=GetFeature&SRS=EPSG%3A4326&BBOX=-74.15771484375,40.66379060988484,-73.80615234375,40.76007871037859
http://featureserver/featureserver.cgi/blocks?format=WFS&maxfeatures=300&SERVICE=WFS&VERSION=1.0.0&REQUEST=GetFeature&SRS=EPSG%3A4326&BBOX=-74.15771484375,40.66379060988484,-73.80615234375,40.76007871037859
</pre>

It is easy to understand that the request OpenLayers is doing to the various FeatureServer datasources is always the same kind: a get feature request with a bounding box (the area you are zooming).

Ok, time to end with this long post, hope now FeatureServer (and various other things) are now a bit easier to understand for you - at least after this long tests it was for me!

