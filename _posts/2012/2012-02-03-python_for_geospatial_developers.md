---
layout: post
title: "Python for geospatial developers"
description: "Python for geospatial developers"
category:
tags: [GIS, GDAL, python, geodjango, shapely, geopy, owslib, qgis, fiona, pyproj, mapserver, mapnik]
---
{% include JB/setup %}

There is a recurring question at GIS mailing lists, forum and at some extent in my
mailbox: what is the best way to master Python for developing geospatial applications?
I myself had this question far away in 2006 when I started switching from proprietary
software to open source, and had identified in Python the way to go.

In this post I will try to quickly summarize what is the best way to go in my opinion.

### Python basics ###

If you are completely new to Python, first things to check out, are some very basic
and popular resources, like these ones:

* [the official Python tutorial] [1]
* [the "Dive into Python book"] [2]

Obviously it does not hurt that they are, like most of resurces I will provide in this
post, free resources like in beer.

### Play with some Python GIS library ###

As soon as you have some basic skills for writing simple Python programs, start
using the Python core library with some of the most popular GIS Python libraries,
choosing the ones that are more relevant to the context of your background:

* [GDAL/OGR Python bindings] [4]
* [shapely] [5]
* [pyproj] [11]
* [Fiona] [12]
* [QGIS] [8]
* [geopy] [6]
* [geodjango] [3]
* MapFish
* [owslib] [7]
* [MapServer] [13]
* [Mapnik] [18]

Before even writing any programs, test these Python APIs with a beautiful and
unique feature of Python:  [the command line interpreter] [20]

For making it even more powerful, don't forget to check out IPython and ipdb.

### Do not complicate things with an IDE, keep it simple ###

Don't use heavy IDE, now and later, but keep things simple using basic but powerful
editors like vi and/or gedit.

They are very light and still both provide features like syntax highlight and
indentation.

You will still be able to debug code in an excellent manner using pdb, or -
 even better - ipdb together with IPython.

### Time for something more challenging ###

When you will be ready for mastering more challenging stuff in order to become a
real Python hacker :), these resources are definitely a must-read:

* ["Code Like a Pythonista: Idiomatic Python"] [9]
* [the  Python Enhancement Proposals (PEPs)] [19]
* ["Python Cookbook" book][21]

### Some more resources on Python and geospatial software ###

Other Python resources, related to the geospatial sphere, worth to be mentioned are:

* the [gispython mailing list] [16]
* Sean Gillies Blog
* the questions tagged "python" @ [gis.stackexchange] [17]
* the ["Python Geospatial Development" book] [14] from Packt. This book gives you
a really nice overview of the main things to know for Python geospatial developers:
nothing you cannot figure out from free web resources, but it is really nice to have all this
packed together in a single book
* the ["Developing Geospatial software with Python" presentation] [15] that Alessandro Pasotti and me
gave at the 2010 Italian GFOSS Day

### Some best practices you should adopt ###

This is a quick list of best practices you should follow when developing with Python
(but this is the same for other languages as well!):

* write documentation using reST, and managing and rendering it with tools like
docutils and Sphinx
* write tests
* use a source control management system like git or mercurial. If you don't want
to create the SCM infrastructure yourself, use the excellent and free services by
gitHub and bitbucket

### Understand the jargon ###

Last but not least, please be sure to have a good understanding of the jargon
background with [these guys] [10] :)

Please, if you think I forget anything from this post, send me your feedback and
I will be very happy to integrate it with yours suggestion ;)
Have fun!

[1]: http://docs.python.org/tutorial/
[2]: http://www.diveintopython.net/
[3]: http://geodjango.org/
[4]: http://trac.osgeo.org/gdal/wiki/GdalOgrInPython
[5]: http://trac.gispython.org/lab/wiki/Shapely
[6]: http://code.google.com/p/geopy/
[7]: http://trac.gispython.org/lab/wiki/OwsLib
[8]: http://www.qgis.org/wiki/Writing_Python_Plugins
[9]: http://python.net/~goodger/projects/pycon/2007/idiomatic/handout.html
[10]: http://www.youtube.com/watch?v=anwy2MPT5RE
[11]: https://code.google.com/p/pyproj/
[12]: https://github.com/Toblerity/Fiona
[13]: http://mapserver.org/mapscript/python.html
[14]: http://www.packtpub.com/python-geospatial-development/book
[15]: http://www.slideshare.net/capooti/developing-geospatial-software-with-python-part-1
[16]: http://lists.gispython.org/mailman/listinfo/community
[17]: http://gis.stackexchange.com/questions/tagged/python
[18]: http://mapnik.org/
[19]: http://python.org/dev/peps/
[20]: http://docs.python.org/tutorial/interpreter.html
[21]: http://shop.oreilly.com/product/9780596001674.do
