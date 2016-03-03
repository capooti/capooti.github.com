---
layout: post
title: "After the GFOSS Day 2010, the Italian FOSS4G event"
description: "After the GFOSS Day 2010, the Italian FOSS4G event"
category:
tags: [Uncategorized, GIS, Python, GDAL, GeoDjango, conferences, Shapely, GeoAlchemy]
---
{% include JB/setup %}

Last week I have been attending the <a href="http://www.gfoss.it/drupal/gfossday2010">GFOSS Day 2010</a> in <a href="http://www.openstreetmap.org/?lat=42.9562&lon=12.7033&zoom=13&layers=M">Foligno</a>, basically the FOSS4G Italian Event that is held every year.
The conference was split in <a href="http://www.gfoss.it/drupal/gfossday2010/programma">two days</a>: in the first days a good number of workshop and tutorials were given, related to a number of geospatial technologies.
In the second day there were the instutional talks, mostly related on the open data theme.
This year I myself gave two talks at the conference: in the first day, togheter with <a href="http://www.itopen.it/">Alessandro Pasotti</a>, we have been giving an extended tutorial about developing geospatial software with Python, here are the slides:

<iframe src="//www.slideshare.net/slideshow/embed_code/key/zi0M3dfMxDEfXv" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/capooti/developing-geospatial-software-with-python-part-1" title="Developing Geospatial software with Python, Part 1" target="_blank">Developing Geospatial software with Python, Part 1</a> </strong> from <strong><a target="_blank" href="//www.slideshare.net/capooti">Paolo Corti</a></strong> </div>

Basically in the first part, after introducing common groundblocks used by almost any geospatial application for geodata managment (GDAL, GEOS, PROJ.4), I have been talking about the most common Python libraries available for using the functionalities of that groundblocks: <a href="http://trac.osgeo.org/gdal/wiki/GdalOgrInPython">GDAL Python Bindings</a>, GeoDjango, Shapely and GeoAlchemy.
In the second part Alessandro, after introducing commonly used geospatial web services, both standard (OGC) and not standard, has been talking the various Python approach to interact with them with some Python libraries (OWSLib, GeoPy). He has then been introducing a couple of Python libraries for web mapping (Mapnik and <a href="http://mapserver.org/mapscript/python.html">MapScript</a>) and finally he described the possible kinds of customizing QGis and GRASS using Python.

All the material of this tutorial (including the second part and the sample code) is <a href="https://github.com/elpaso/python-gis-workshop">here</a>.

In the second day, I have been presenting the EFFIS (the European Forest Fire Information System), the GIS system I am currently working on here at Joint Research Centre, Institute for Environment and Sustainability: slides are <a href="http://www.gfoss.it/drupal/files/Corti_EFFIS2_0.pdf">here</a> (in Italian).
The presentation was basically an Italian version of the <a href="http://2010.foss4g.org/presentations_show.php?id=3693">talk</a> that my collegue Cristiano Giovando gave at the last FOSS4G in Barcelona.

Some final noteS on the GFOSS Day: the event was a success, with over 150 attenders and an excellent set of talks, workshop and tutorials. The organization was great and I am already looking forward for the next event, possibly the <a href="http://events.unitn.it/foss4g2011">XII GRASS and GFOSS meeting in Trento</a>.
