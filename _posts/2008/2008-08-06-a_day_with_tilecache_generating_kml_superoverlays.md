---
layout: post
title: "A day with TileCache: generating KML Super-Overlays"
description: "A day with TileCache: generating KML Super-Overlays"
category:
tags: [Uncategorized, GIS, MapServer, devs, Tutorials, Apache, FeatureServer, Python, GeoServer, WMS, OpenLayers, TMS, TileCache, kml, MetaCarta, GDAL]
---
{% include JB/setup %}

<em>My friend Diego Guidi is the smartest GIS/.NET developer I personally know here in Italy. He is the developer of NetTopologySuite, the port in the .NET world of the popular <a href="http://www.vividsolutions.com/jts/jtshome.htm">Java's JTS Topology Suite</a> from <a href="http://www.vividsolutions.com/">VIVID Solutions</a>. I wanted, sooner or later, write some stuff here about WMS and TMS, and now I am very happy that Diego asked me to publish this brilliant article about this topic.</em>

First of all, let me thanks <a href="http://twitter.com/capooti">Paolo</a> for hosting this post! I hope that this article can be interesting and useful like <a href="http://www.paolocorti.net/2008/02/21/a-day-with-featureserver-1/">other stuff</a> that you <a href="http://www.paolocorti.net/2008/06/06/spatial-database-for-postgres-and-arcgis-users-how-to-choose/">can find</a> <a href="http://www.paolocorti.net/2008/02/29/do-you-get-errors-in-the-mapscritp-c-tutorial/">here</a>...

<strong>Introduction</strong>

There are <a href="http://www.google.com/search?q=is+%22google+maps%22+gis&sourceid=navclient-ff&ie=UTF-8&rlz=1B3GGGL_enUS272US273">a lot of discussions</a> out there about how to define <a href="http://earth.google.com/">Google Earth</a>, <a href="http://maps.google.com/">Google Maps</a>, and <a href="http://maps.live.com/">related apps</a>... are they GIS? Viewers? Video games? Even a neologism was created: Neogeography. I think that all the folks out there have the same idea in mind: maybe Google don't make the same business as ESRI, but Google Earth is cool, and it's funny to play with it!

For a simple user, the verb "play" could be meaning: "wow! I fly over my house!"; for me, as a GIS developer, the verb "play" means something slightly different: "How can I show my GIS data, like shapefiles, geodatabases and/or raster images, in Google Earth?".
The simpler answer to this question is to publish your data as Web Map Service (WMS) and then consume it from any compatible clients, like OpenLayers or Google Earth; this solutions works, and works well, if you don't matter performances... let's give a look to a simple real-world example.
Your customer calls at the phone and say something like: "I have a bunch of data, some shapefiles and some GeoTIFF images: can you please help me to show these data in Google Earth?". You may say: "Absolutely! Google Earth is a WMS client, so I help you building a WMS service". The problem is that the "bunch of data" is 10+ gigabytes of raster images, and the customer, as usual, love to see immediately his data on screen (and not to wait seconds - or minutes - before WMS sends the response for each request), just because "if google can do it, why you can't do the same?"...
The way to achieve this goal is not to buy an extremely powerful hardware for your service, but simply to convert the data, published by the service itself, in a way that is faster to manage... like a ton of little images (a.k.a. tiles) stored to disk.

<strong>Enter KML Super-Overlays</strong>

<a href="http://code.google.com/apis/kml/documentation/kml_21tutorial.html#superoverlays">Super-Overlays</a> are a way, offered by the KML standard , to manage extremely large datasets; from the <a href="http://code.google.com/apis/kml/documentation/kml_21tutorial">KML tutorial</a>:

Q: How can I share a 47MB image with the world?
A: One piece at a time.

So, basically create a Super-Overlay involves a three-step procedure:

<ul>
  	<li>1. Prepare your data.</li>
   	<li>2. Generate the tiles.</li>
   	<li>3. Prepare a valid KML document to publish this tiles.</li>
</ul>

Although the first step is relatively easy to do with well-known products, both open-source (MapServer, GeoServer or SharpMap) or commercial, the next steps are something new and unknown, at least for me.

<strong>TileCache to the rescue</strong>

TileCache is an open-source project made by the smart guys at MetaCarta Labs - check also their other amazing apps, like OpenLayers and FeatureServer - and it's essentially a proxy between generic GIS clients and a Web Map Service: TileCache's purpose, as the name suggests, is to manage a request from a client, ask a Web Map Service for the response and cache it, so for next requests for the same data the proxy itself could use his cached copy and avoid requests to the Web Map Service.

<a href='/assets/images/ahqh2wf9bzck_99dd3p9nfz_b.png'><img src="/assets/images/ahqh2wf9bzck_99dd3p9nfz_b-300x227.png" alt="TileCache Sequence Diagram" title="TileCache Sequence Diagram" width="300" height="227" class="aligncenter size-medium wp-image-85" /></a>

I've talk about generic "GIS services" as clients, because TileCache proxy could act not only as Web Map Service (requests <a href="http://labs.metacarta.com/wms-c/tilecache.py?LAYERS=basic&FORMAT=image/png&SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&STYLES=&EXCEPTIONS=application/vnd.ogc.se_inimage&SRS=EPSG:4326&BBOX=-90,0,0,90&WIDTH=256&HEIGHT=256">like this one</a>) but also as a Tile Map Service (requests <a href="http://labs.metacarta.com/wms-c/tilecache.py/1.0.0/basic/5/32/23.png">like this one</a>): in particular, TMS handling capabilities are used in KML Super-Overlays, so later we talk a little bit more about this service.
One thing that must be clear: cached tiles are generated by a Web Map Service; so, the first step of our procedure ("Prepare your data", do you remember?) means that we need to build and configure a WMS that is accessible from TileCache (and TileCache only: you could hide this WMS behind a proxy because the only service that is needed to be exposed over the Internet is only TileCache).

<strong>TileCache is not the only one...</strong>

Although TileCache, at least for my experience, works really well, other software is available to make the same purpose; one of the most known is GDAL2Tiles, that is part of the GDAL project. GDAL2Tiles it's beyond the scope of this article, but I want to specify some bigger differences between both projects, so maybe you could understand if GDAL2Tiles is better for you:

<ul>
	<li>While TileCache needs a configured WMS service as data source, GDAL2Tiles works with <a href="http://www.gdal.org/formats_list.html">any of the formats supported</a> by GDAL. For my experience, It's easier to prepare a WMS, test how it works (and how our data is rendered) using a WMS Client and then generate the tiles using TileCache.</li>
	<li>While GDAL2Tiles needs to pregenerate all the tiles, TileCache could generate a tile on the fly when it's asked for the first time (of course, for the next requests, the cached tile is used): this saves a lot of time because often the pregeneration process is extremely slow.</li>
</ul>

<strong>Finally @work</strong>

<strong>Prerequisites</strong>

Remember: TileCache does nothing all alone, so first of all we need a Web Map Service; we want to try TileCache immediately, then we use a WMS exposed via Internet, <a href="http://onearth.jpl.nasa.gov/">NASA's OnEarth</a> (<a href="http://onearth.jpl.nasa.gov/wms.cgi">at this url</a>), that publish a "current global view of the earth", as described in <a href="http://onearth.jpl.nasa.gov/wms.cgi?request=GetCapabilities">capabilities document</a>, called daily_planet. Building a complete WMS server is an interesting topic itself, and maybe in a next article we will write about that; for now, if you're interested to use your own data, check <a href="http://ms-ogc-workshop.maptools.org/">this tutorial</a> about how to configure and use MapServer as Web Map Service.
The only mandatory requisite is that the WMS must support <a href="http://spatialreference.org/ref/epsg/4326/">EPSG:4326  -WGS 84 LatLong</a> as Spatial Reference System for published data, because Google Earth could manage only this kind of SRS.
A suggested requisite is that the service could publish data using at least one image format that supports transparency - like PNG - because often Google Earth's users love to show your data upon other stuff, without hiding it.

<strong>Apache HTTP Setup</strong>

Before installing TileCache, we need Apache HTTP Server up and running: I've downloaded and installed 2.2.9 version from the <a href="http://httpd.apache.org/download.cgi">download section</a>, that creates a ready-to-use web server that runs at the address localhost:8080; I've also downloaded and installed 2.5.2 version of Python interpreter and 3.3.1 version of Apache's <a href="http://www.modpython.org/">Mod Python</a>, an Apache module that embeds the Python interpreter within the server.
Once that software is installed, configuring Mod_Python is pretty straightforward (but If you encounter some problems, <a href="http://openlayers.org/mailman/listinfo/tilecache">TileCache's mailing list</a> is a good resource for <a href="http://www.nabble.com/Tilecache-in-mod_python-td16726713.html#a16726713">technical solutions</a>): open httpd.conf file under the Apache conf folder and add:

{% highlight bash %}
LoadModule python_module modules/mod_python.so
{% endhighlight %}

at the end of LoadModule declarations.
To check if Python interpreter is well configured, create a folder called test under Apache htdocs folder, and add a directory section like this:

{% highlight xml %}
<Directory "apacherootfolder/htdocs/test/">
    AddHandler mod_python .py
    PythonHandler mptest
    PythonDebug On
</Directory>
{% endhighlight %}

where apacherootfolder is the path of your Apache installation folder ("C:\Programmi\Apache Software Foundation\Apache2.2" in my Apache installation).

Now create a file called mptest.py under the already created test folder, with this content:

{% highlight python %}
from mod_python import apache

def handler(req):
    req.content_type = 'text/plain'
    req.write("Hello, World!")
    return apache.OK
{% endhighlight %}

Now start Apache HTTP Server and go to the url:

<pre lang="text">
http://localhost:8080/test/mptest.py
</pre>

and see what's happen... if you read the familiar "Hello, World!", you're the man!

<strong>TileCache time</strong>

Now download version 2.0.4 of TileCache, <a href="http://tilecache.org/tilecache-2.04.zip">packed as zip file</a> that you could extract under Apache "htdocs" folder; I've also renamed the root folder from "tilecache-2.0.4" to simply "tilecache", so I have a hierarchy like this: "htdocs/tilecache/" that contains all the tilecache files and executables.
Looking at tilecache files, you see "tilecache.cgi"... this means that it's not mandatory to use Apache HTTP Server and Mod_Python: <a href="http://tilecache.org/readme.html">installation readme</a> contains all information you need to use TileCache under CGI, FASTCGI, and so on... and if you like to use IIS, <a href="http://viswaug.wordpress.com/">Vish Uma</a> has made <a href="http://viswaug.wordpress.com/2008/02/03/setting-up-tilecache-on-iis/">a great tutorial</a> that you must read.

Re-open http.d conf file and add a Directory section like this:

{% highlight xml %}
<Directory "apacherootfolder/htdocs/tilecache/">
    AddHandler python-program .py
    PythonHandler TileCache.Service
    PythonPath "['apacherootfolder/htdocs/tilecache'] + sys.path"
    PythonOption TileCacheConfig "apacherootfolder/htdocs/tilecache/tilecache.cfg"
    PythonDebug Off
</Directory>
{% endhighlight %}

In the http.d file we have specified as TileCacheConfig parameter a reference to a file that contains all configuration options: tilecache.cfg is a text file (What The Hell? Why a simple and perfectly readable TXT and not an <a href="http://msdn.microsoft.com/en-us/library/aa719558%28VS.71%29.aspx">unusable XML file</a>? Are you crazy at MetaCarta?) that contains some global options (like cache strategy) and a list of published layers, each with his specifications (like WMS source, SRS, tile image format, and so on).
The configuration file itself contains documentations for all options supported, but for our purposes we need a really simple configuration file, like this:

<pre lang="text">
[cache]
type=Disk
base=apacherootfolder/htdocs/cache
[NASA_JPL_WMS]
type=WMS
url=http://onearth.jpl.nasa.gov/wms.cgi
layers=daily_planet
extension=png
size=256,256
bbox=-180.0,-90.0,180.0,90.0
srs=EPSG:4326
levels=20
</pre>

So we use a cache-to-disk strategy (we decide, but it's not a mandatory, to save our tiles under Apache "htdocs" folder because we may want to access directly from internet), and a single TileCache layer called NASA_JPL_WMS, published with SRS EPSG:4326: this layer uses specified WMS URL to ask for 256x256 tile images from daily_planet layer, requested as image/png mime type and stored as PNG files; we need images for entire world extension, so a valid bounding box in WGS 84 LatLong coordinates is specified, and we decide to use twenty levels of detail for our tiles.
These are all information TileCache needs to build a WMS request to our service, get a tile to store as cache, and then send it to clients requesting for our data.

<strong>The "web 2.0" way to make tests</strong>

It's time to show our tiles, but before using Google Earth we start with a simple html page, that could be helpful to verify if we made some mistakes, just because with FireFox and Firebug we can easily see all the requests to our services and see what happens.
We use OpenLayers library, a javascript framework for building WebGIS applications: this library is really, really, really an interesting piece of software because helps to build mapping applications easily and with nice features: take a look at <a href="http://openlayers.org/dev/examples/example-list.html">some examples</a> and explore <a href="http://gallery.openlayers.org/">the applications gallery</a>! I hope to write a blog post soon about OpenLayers... but after my vacation indeed, sorry :(
So, we change the content of the script tag of default index.html page (that is part of the TileCache distribution) with this javascript code:

{% highlight javascript %}
<script type="text/javascript">
        <!--
        OpenLayers.IMAGE_RELOAD_ATTEMPTS = 3;
        OpenLayers.Util.onImageLoadErrorColor = "transparent";       

        function init()
        {
            var map = new OpenLayers.Map($('map'),
            {
                controls: [],
                maxResolution: 360/512,
                projection: "EPSG:4326" ,
                numZoomLevels: 20,
                minZoomLevel: 0,
                maxZoomLevel: 19  
            });

            var layer1 = new OpenLayers.Layer.WMS("WMS",
                "tilecache.py?", {
                    layers: "NASA_JPL_WMS",
                    format: "image/png" });           
            var layer2 = new OpenLayers.Layer.TMS("TMS",
                "tilecache.py/", {               
                    serviceVersion: "1.0.0",
                    layername: "NASA_JPL_WMS",
                    type: "png" });
            var layer3 = new OpenLayers.Layer.TileCache(
                "TileCache", "../cache",
                "NASA_JPL_WMS", {
                    format: "image/png", });

            map.addLayers([layer1, layer2, layer3]);
            map.setBaseLayer(layer1);

            map.addControl(new OpenLayers.Control.Navigation());
            map.addControl(new OpenLayers.Control.KeyboardDefaults());   
            map.addControl(new OpenLayers.Control.PanZoom());
            map.addControl(new OpenLayers.Control.NavToolbar());
            map.addControl(new OpenLayers.Control.Permalink());
            map.addControl(new OpenLayers.Control.LayerSwitcher());
            if (!map.getCenter())
                map.zoomToMaxExtent();
        }
        // -->
    </script>
{% endhighlight %}

In this page we have build several layers, asking for our tiles in different ways:

<ul>
   	<li>1. The old and glorious Web Map Service.</li>
   	<li>2. The new upcoming star Tile Map Service.</li>
   	<li>3. The outsider, and free runner, TileCache "service".</li>
</ul>

Looking at the urls we may recognize a familiar WMS request:

<pre lang="text">
http://localhost:8080/tilecache/tilecache.py?LAYERS=NASA_JPL_WMS&FORMAT=image%2Fpng&SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&STYLES=&EXCEPTIONS=application%2Fvnd.ogc.se_inimage&SRS=EPSG%3A4326&BBOX=90,-90,180,0&WIDTH=256&HEIGHT=256
</pre>

a not-so-familiar but quite-simple TMS request:

<pre lang="text">
http://localhost:8080/tilecache/tilecache.py/1.0.0/NASA_JPL_WMS/1/3/1.png
</pre>

and an even more simple TileCache request:

<pre lang="text">
http://localhost:8080/cache/NASA_JPL_WMS/02/000/000/007/000/000/003.png
</pre>

Look deeper at the request urls, we see an interesting thing: the first two requests uses TileCache service, that take care of requests and manage in some ways a response, but third request is simply an url to an image! This is why we have cached our tiles under "htdocs" folder... isn't cool? Unfortunately, for our purposes we need to generate kml documents on-the-fly: actually, Google Earth requests are TMS requests, but with "type: kml": try to write this url directly in your browser and see what happens...

<pre lang="text">
http://localhost:8080/tilecache/tilecache.py/1.0.0/NASA_JPL_WMS/1/3/1.kml
</pre>

<strong>Greetings from the Earth</strong>

<strong>Preparing our KML</strong>

We're finally at the final step: create a KML document and see our data on Google Earth! This job is quite easy, just because TileCache distribution contains a kml template called overlay.kml (under the "htdocs/tilecache/docs/examples" folder), that we could modify to redirect calls to our ready-to-use TileCache service:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<kml xmlns="http://earth.google.com/kml/2.1">
  <Folder>  
  <NetworkLink>
    <Link>
      <href>http://localhost:8080/tilecache/tilecache.py/1.0.0/NASA_JPL_WMS/0/0/0.kml</href>
    </Link>
  </NetworkLink>
  <NetworkLink>
    <Link>
      <href>http://localhost:8080/tilecache/tilecache.py/1.0.0/NASA_JPL_WMS/0/1/0.kml</href>
    </Link>
  </NetworkLink>
  </Folder>
</kml>
{% endhighlight %}

Simply, this document instructs Google Earth about how to find external resources (other KML documents) via <a href="http://www.gearthblog.com/blog/archives/2007/04/the_google_earth_net.html">Network Links</a>, that may basically be anything the KML files can support, included kml's and/or images. TileCache uses a 'whole world' extent split into two tiles that are western and eastern hemisphere, so we're actually asking data for the entire world; if I open this document in Google Earth, our raster dataset overlays default content :)

<strong>A deeper look</strong>

We're curious, so let's take a look at one of the generated KML's.
Type an address like this in your browser:

<pre lang="text">
http://localhost:8080/tilecache/tilecache.py/1.0.0/NASA_JPL_WMS/0/1/0.kml
</pre>

and open generated response with a text editor, looking for interesting tags like:

<ul>
   	<li>1. GroundOverlay: as <a href="http://code.google.com/apis/kml/documentation/kmlreference.html#groundoverlay">kml reference suggests</a>, draws an image overlay draped onto the terrain.</li>
   	<li>2. Icon: <a href="http://code.google.com/apis/kml/documentation/kmlreference.html#icon">the image to draw</a> as overlay, is a link to our TileCache Service using TMS specifications.</li>
   	<li>3. NetworkLink: four references to other kml documents that defines next levels of detail for the shown image.</li>
</ul>

<a href='/assets/images/ahqh2wf9bzck_104gvxzrtcd_b.png'><img src="/assets/images/ahqh2wf9bzck_104gvxzrtcd_b.png" alt="kml tiles" title="kml tiles" width="151" height="152" class="aligncenter size-medium wp-image-86" /></a>

So, this means that each document defines how to retrieve subsequents documents that provides higher levels of detail: simple, easy and <a href="http://en.wikipedia.org/wiki/KISS_principle">perfectly working</a>.

<strong>Speeding your responses</strong>

Playing with the test page you may see a little but obviously problem: when TileCache needs to generate a tile from scratch, you need to wait! Perhaps wait-time is not significant at lower visualization scales, but at larger scales WMS service (that is the "real" tile generator) often needs to manage and process a very-large dataset, and a response delay of thirty seconds or more is not accettable.
Fortunately, we could resolve this issue using tilecache_seed.py script (shipped with TileCache distribution) that simply generates "fake" requests to our service, forcing TileCache to generate tiles.
Script usage is straightforward; simply open a console prompt, go to Python 2.5 installation folder (C:\Python25 in my PC) and type

<pre lang="text">
python "apacheroot/htdocs/tilecache/tilecache_seed.py" "http://localhost:8080/tilecache/tilecache.py?" NASA_JPL_WMS 0 10
</pre>

where the parameters are:

<ul>
   	<li>1. Full absolute path of "tilecache_seed.py".</li>
   	<li>2. Url of our TileCache service.</li>
   	<li>3. TileCache layer to use in tiles generation</li>
   	<li>4. Minimum zoom level, the start-point in tiles generation process.</li>
   	<li>5. Maximum zoom level, the end-point in tiles generation process.</li>
</ul>

Note that tiles generation process is extremely slow: for my experience, often is useful to pregenerate only first levels of detail (like in the example above) and let TileCache generate on-the-fly remaining levels. An acceptable compromise is to start using your service when tilecache_seed is still working: this degrade performances only a little bit if dataset's management is not too expensive for your WMS service.

<strong>Time to say goodbye</strong>

TileCache is cool, no doubt about this, but, as many other open-source projects, <a href="http://tilecache.org/readme.html">documentation</a> is not so cool like the code itself; in addition, Super-Overlays isn't clear (at least for me), so I hope that this blog post could be helpful for anyone interested. This is my little contribution to TileCache documentation: any suggest or criticism is well-accepted :)
<a href="http://twitter.com/d_guidi">Diego Guidi</a>

P.S: in this journey we've encountered some interesting "extension points" for this article... installation issues in a non-windows environment,  how to build a WebGIS application with OpenLayers, how to create a Web Map Service... actually, my thought was to stay focused on TileCache only: I hope this goal was achieved, and I promise to contribute with more articles soon.
