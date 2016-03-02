---
categories: Uncategorized, GIS, Python, GDAL, GeoDjango, conferences, Shapely, GeoAlchemy
date: 2010/11/29 11:24:10
guid: http://www.paolocorti.net/?p=315
permalink: http://www.paolocorti.net/2010/11/29/after-the-gfoss-day-2010-the-italian-foss4g-event/
tags: Uncategorized, GIS, Python, GDAL, GeoDjango, conferences, Shapely, GeoAlchemy
title: After the GFOSS Day 2010, the Italian FOSS4G event
---
Last week I have been attending the <a href="http://www.gfoss.it/drupal/gfossday2010">GFOSS Day 2010</a> in <a href="http://www.openstreetmap.org/?lat=42.9562&lon=12.7033&zoom=13&layers=M">Foligno</a>, basically the FOSS4G Italian Event that is held every year.
The conference was split in <a href="http://www.gfoss.it/drupal/gfossday2010/programma">two days</a>: in the first days a good number of workshop and tutorials were given, related to a number of geospatial technologies.
In the second day there were the instutional talks, mostly related on the open data theme.
This year I myself gave two talks at the conference: in the first day, togheter with <a href="http://www.itopen.it/">Alessandro Pasotti</a>, we have been giving an extended tutorial about developing geospatial software with Python, here are the slides:

<div style="width:425px" id="__ss_5755093"><strong style="display:block;margin:12px 0 4px"><a href="http://www.slideshare.net/capooti/developing-geospatial-software-with-python-part-1" title="Developing Geospatial software with Python, Part 1">Developing Geospatial software with Python, Part 1</a></strong><object id="__sse5755093" width="425" height="355"><param name="movie" value="http://static.slidesharecdn.com/swf/ssplayer2.swf?doc=pythongispart1-101112083509-phpapp01&stripped_title=developing-geospatial-software-with-python-part-1&userName=capooti" /><param name="allowFullScreen" value="true"/><param name="allowScriptAccess" value="always"/><embed name="__sse5755093" src="http://static.slidesharecdn.com/swf/ssplayer2.swf?doc=pythongispart1-101112083509-phpapp01&stripped_title=developing-geospatial-software-with-python-part-1&userName=capooti" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="355"></embed></object><div style="padding:5px 0 12px">View more <a href="http://www.slideshare.net/">presentations</a> from <a href="http://www.slideshare.net/capooti">Paolo Corti</a>.</div></div>

Basically in the first part, after introducing common groundblocks used by almost any geospatial application for geodata managment (GDAL, GEOS, PROJ.4), I have been talking about the most common Python libraries available for using the functionalities of that groundblocks: <a href="http://trac.osgeo.org/gdal/wiki/GdalOgrInPython">GDAL Python Bindings</a>, GeoDjango, Shapely and GeoAlchemy.
In the second part Alessandro, after introducing commonly used geospatial web services, both standard (OGC) and not standard, has been talking the various Python approach to interact with them with some Python libraries (OWSLib, GeoPy). He has then been introducing a couple of Python libraries for web mapping (Mapnik and <a href="http://mapserver.org/mapscript/python.html">MapScript</a>) and finally he described the possible kinds of customizing QGis and GRASS using Python.

All the material of this tutorial (including the second part and the sample code) is <a href="https://github.com/elpaso/python-gis-workshop">here</a>.

In the second day, I have been presenting the EFFIS (the European Forest Fire Information System), the GIS system I am currently working on here at Joint Research Centre, Institute for Environment and Sustainability: slides are <a href="http://www.gfoss.it/drupal/files/Corti_EFFIS2_0.pdf">here</a> (in Italian).
The presentation was basically an Italian version of the <a href="http://2010.foss4g.org/presentations_show.php?id=3693">talk</a> that my collegue Cristiano Giovando gave at the last FOSS4G in Barcelona.

Some final noteS on the GFOSS Day: the event was a success, with over 150 attenders and an excellent set of talks, workshop and tutorials. The organization was great and I am already looking forward for the next event, possibly the <a href="http://events.unitn.it/foss4g2011">XII GRASS and GFOSS meeting in Trento</a>.
