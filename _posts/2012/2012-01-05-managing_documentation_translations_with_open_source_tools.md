---
layout: post
title: "Managing documentation's translations with Open Source tools"
description: "Managing documentation's translations with Open Source tools"
category:
tags: [GIS, reST, Sphinx, po4a]
---
{% include JB/setup %}

Today together with [Paolo Cavallini] [1], I have been trying to figure out a workflow for managing documentation's translations for QGIS.

There are many tools for managing documentation workflows out there: a common approach to which I have been exposed by following both the Python and the OSGeo communities is to keep the documentation source files in the restructured text markup format, and to generate the end user documentation with tools like docutils and Sphinx to a bunch of different formats (i.e. html, odt, pdf, latex... just to name a few).

Python documentation itself and two wide spread GIS projects like MapServer and GeoServer are following these workflows. Other projects follow different approaches, for example GDAL and PostGIS use Docbook with Doxygen.

While this formats, workflows and tools (reSTructured Text, Docutils, Sphinx...) are invaluable for managing the documentation of a projects, they do not provide a specific tool for managing the multiple translations of the documentation itself, in the same fashion like the gettext tool help developers to manage GUI software localizations.
What is needed in the management of the localization of documentation, is to find a way to manage different versions of each documentation files for each language. But we have a good news here: there is a toolset, from the Perl world, named po4a, that can be combined together with docutils and/or sphinx for this purpose.

In a few words, as suggested at the website, "the whole point of po4a is to make the documentation translation maintainable. The idea is to reuse the gettext methodology to this new field. Like in gettext, texts are extracted from their original locations in order to be presented in a uniform format to the translators. The classical gettext tools help them updating their works when a new release of the original comes out. But to the difference of the classical gettext model, the translations are then re-injected in the structure of the original document so that they can be processed and distributed just like the English version."

So an optimal approach to manage documentation translations could be the combined use of restructured text and po4a. Let's give it a try.

### Setup the environment

First, let's create an empty .rst file with some content in it. For this purporse you could use [this] [2] .rst file from the GeoServer documentation sources.
Let's name this file original.rst.

You may render thir .rst file at any time as html by using the docutils rst2html command:

    $ rst2html original.rst original.html

As initial step, we need to create the template of the translation file with the po4a-gettextize command:

    $ po4a-gettextize -f text -m original.rst -p translation.pot

We need to make a copy of this file for each language for which we need to make a translation. Let's create a copy for the Italian translation:

    $ cp translation.pot translation.IT.po

### Translate the files

Now you can safely translate the translation.IT.po file. You will need to add a translation for each msgid in the msgstr field.
For example, for the first paragraph we will have this translation:

    #. type: Plain text
    #: original.rst:7
    msgid ""
    "GeoServer is an open source software server written in Java that allows "
    "users to share and edit geospatial data. Designed for interoperability, it "
    "publishes data from any major spatial data source using open standards."
    msgstr ""
    "GeoServer e' un software server Open Source scritto in Java che consente"
    "agli utenti di condividere ed editare informazione geografica."
    "Sviluppato per l'interoperabilita', consente la pubblicazione di dati"
    "da tutte le piu' importanti sorgenti dati usando standard open."

### Get the translated file

Finally we can get the translation of the original rst file, by using the po4a-translate command:

    $ po4a-translate -f text -m original.rst -p translation.IT.po -l IT.rst -k 40

You can finally look at the final result, for example by converting the IT.rst file to html using docutils:

    $ rst2html IT.rst IT.html

Open the IT.html file in a web browser, and see yourself the results.

### Mananging updates

The real value of the po4a toolset is in the fact that it greatly helps to manage the updates to source files to be translated during the life of the project.
Any time the original file is updated/rearranged the user will be able to identify what it needs to be changed on the translation file by using the pro4a-updatepo command:

    $ po4a-updatepo -f text -m original.rst -p translation.IT.po

After running the po4a-updatepo command, the translation.IT.po file will be synced with the original.rst changed file. New paragraphs will be added, and modified paragraphs will be marked by a "fuzzy" tag in the file itself.
For example, if we try to modify the first paragraph of the file, after running the po4a-updatepo command this is what we will get in the translation.IT.po file:

    #. type: Plain text
    #: original.rst:9
    #, fuzzy
    msgid ""
    "GeoServer is an open source software server written in Java that allows "
    "users to share and edit geospatial data. Designed for interoperability, it "
    "publishes data from any major spatial data source using open standards "
    "(little modification)."
    msgstr ""
    "GeoServer e' un software server Open Source scritto in Java che consenteagli "
    "utenti di condividere ed editare informazione geografica.Sviluppato per "
    "l'interoperabilita', consente la pubblicazione di datida tutte le piu' "
    "importanti sorgenti dati usando standards open."


[1]: http://www.faunalia.it/
[2]: http://docs.geoserver.org/stable/en/user/_sources/introduction/history.txt
