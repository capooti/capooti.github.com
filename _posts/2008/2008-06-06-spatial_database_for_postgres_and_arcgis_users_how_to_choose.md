---
categories: GIS, PostGIS, ArcIMS, ArcObjects, ArcGis Desktop, ZigGis, FeatureServer,
  QGIS, uDig, gvSIG, GeoServer, ArcSde
date: 2008/06/06 17:45:32
guid: http://www.paolocorti.net/public/wordpress/index.php/2008/06/06/spatial-database-for-postgres-and-arcgis-users-how-to-choose/
permalink: http://www.paolocorti.net/2008/06/06/spatial-database-for-postgres-and-arcgis-users-how-to-choose/
tags: GIS, PostGIS, ArcIMS, ArcObjects, ArcGis Desktop, ZigGis, FeatureServer, QGIS,
  uDig, gvSIG, GeoServer, ArcSde
title: 'Spatial Database for Postgres and ArcGis users: how to choose'
---
As many of you can have already read, ArcSde for Postgres is coming out at ArcGis 9.3 (it is currently in the release candidate state). It will let you store geometries in two formats, Esri geometry and PostGis geometry, in the same fashion ArcSde for Oracle is letting Esri or Oracle geometries be stored. 
I have seen some interest in the GIS community about this new, and i was reading <a href="http://geobabble.wordpress.com/2008/05/28/using-arcsde-93-with-postgresql-part-1/">interesting</a> <a href="http://geobabble.wordpress.com/2008/06/02/using-arcsde-93-with-postgresql-part-2/">posts</a> by Bill Dollins, <a href="http://blog.cleverelephant.ca/2008/05/why-use-arcsde.html">Paul Ramsey</a>, <a href="http://www.spatiallyadjusted.com/2008/05/09/a-look-at-postgresql-and-arcsde/">James Fee</a> and <a href="http://blog.davebouwman.net/2008/05/09/PreppingForWhere20.aspx">Dave Bouwman</a>, so I thought i would post here my opinion. Plus, as some of you may already know, <a href="http://www.obtusesoft.com/pr.html">zigGis 2.0 is out</a>, so I am very interested in understanding where it would be better to use one (ArcSde) or the other (zigGis 2.0) solution.

First let me introduce you my point of view: i am since the ARC/INFO era a great fan of Esri software, and I am one of the many users in the GIS community saying that Esri is much much better vs Open Source in the GIS desktop products sector. 
In fact, while also in the GIS OS scenario there are good products like QGIS, uDig, gvSIG (and some other), in my opinion no one of them still can even fairly reach what ArcGis Desktop and its extensions can actually do (with the big limit that you can use it only in a Microsoft box, but that is another story).

At my job place, working for the public sector and willing to follow the CEE recommendations, I am also an advocate of Open Source software, and I truely believe that -not like for desktop products - the GIS OS scenario for server side products its perfectly close to commercial solutions, in some case even better.

Products like MapServer, GeoServer, PostGis and their skeleton libraries like Proj and GDAL (just to name a few), are a wonderful example of outstanding results obtained within the Open Source philosophy from a fantastic GIS community of developers and very passionate users.
I came to the OS side some years ago, when at my office we couldn't afford anymore the Esri  product suite (it was the 9.0 era, then). 
To go straight, we decided to switch to Open Source for the server family (choicing PostGis and MapServer replacing ArcSde for SQL Server and ArcIms), while we decided to keep the ArcView's licenses. Nowadays i think this was a decision that saved us money and still let us to keep a powerful GIS environment.
What I was missing then, though, was the Esri GeoDatabase model, and the possibility to view and edit data via ArcMap without using complicated solutions implemented by using some sort of interchange files.
Regarding the GeoDatabase model, I quickly found that using Postgres SQL and PostGis Spatial functions I could almost obtain what i really needed from the Geodatabase stuff.
Regarding the possibility to use PostGis in ArcMap, that days where the times when I started to think about writing a PostGis connector for ArcGis Desktop in my spare time (typically at the weekends and with long no-sleep nights). Not being totally satisfied of pgArc, and before I could even start with my project, I discovered the magnificent ArcObjects code that Abe Gillespie wrote to implement its first zigGis version (0.3 if i don't remember badly).
The code Abe wrote was really a wonder at my eyes: a completed PostGis Workspace implementations developed from scratch by using ArcObjects in an extreme, original and not at all documented way (at that time, but maybe even now). 
To get an idea, here is how looks the code to add a PostGis layer in ArcMap using the zigGis library:

$$code(lang=vbnet)
  'Create PostGisWorkspaceFactory
  Dim wksf As IWorkspaceFactory
  Set wksf = New PostGisWorkspaceFactory
  
  'Open the PostGIS Workspace from a from IPropertySet (in the SDEWorkspaceFactory fashion)
  Dim ps As IPropertySet
  Set ps = New PropertySet
  With ps
    .SetProperty "server", "localhost"
    .SetProperty "database", "TUTORIAL"
    .SetProperty "user", "psqluser"
    .SetProperty "password", "psqluser"
    .SetProperty "port", "5432"
    'specify the configfile property if you want to log zigGis actions for debugging
    '.SetProperty("configfile", @"C:\ziggis\ZigGis\logging.config")
  End With
  Dim ws As IWorkspace
  Set ws = wksf.Open(ps, 0)
  
  'Alternatively you can open the PostGIS Workspace from a zigFile
  'ws = wksf.OpenFromFile(@"C:\ziggis\ZigGis\example.zig", 0);
  
  'Open the PostGIS feature class
  Dim fwks As IFeatureWorkspace
  Set fwks = ws
  Dim fc As IFeatureClass
  Set fc = fwks.OpenFeatureClass("zone")
  
  'Create the new layer (default renderer is ISimpleRenderer)
  Dim layer As IFeatureLayer
  Set layer = New PostGisFeatureLayer
  Set layer.FeatureClass = fc
  layer.Name = fc.AliasName
  
  'Add the layer in ArcMap focus map
  Dim pMxDoc As IMxDocument
  Set pMxDoc = ThisDocument
  pMxDoc.FocusMap.AddLayer layer
$$/code

(more samples <a href="http://www.paolocorti.net/public/wordpress/index.php/2007/02/21/iworkspacefactory-wksf-new-postgisworkspacefactory/">here</a>)

So I just fell in love with that project, and as far Abe had no time to keep it in life because of other works he was heavily involved in, I decided to inherit its work and to keep it in life.
Almost at the same time, also Bill Dollins, one of the most experienced GIS people I have ever known, joined the project, and together we released zigGis 1.0, the first zigGis version with an installer. Quickly other versions came out, until zigGis 1.2, the first version massively installed by the community (almost 5,000 downloads).
One big thing was missing then: zigGis was read only, and for editing data you needed to go out from ArcMap and go straight with some OS alternatives like QGIS, gvSIG or uDig.
Abe rejoined the project, and a lot of development time was spent for implementing editing and make zigGis performance much better.
Now zigGis 2.0 is out, giving the user a full GIS experience with PostGis in ArcMap.

Having talked about the release of ArcSde for Postgres and of zigGIS 2.0, let's give a close look at some kind of combinations you can adopt for storing GIS data in Postgres and access them by ArcGis Desktop.
The typical every day Esri user may very possibly not ever heard about Postgres and PostGis. In a few words you can consider the Postgres/PostGis combination an Open Source equivalent of the xRDBMS/ArcSde product (where x=Oracle, SQL, DB2, Informix).
Now, at ArcGis 9.3 release, the x in xRDBMS will also host Postgres.

I would say that when ArcSde for Postgres will be out (soon), there will be at least four ways to go for using Postgres in ArcGis Dekstop:

<ul>
<li>1.  Using ArcSde for Postgres storing data as Esri geometries (Esri-centric with Postgres view)</li>
<li>2.  Using ArcSde for Postgres storing data as PostGis geometries using the Esri Geodatabase model (Esri-centric hybrid view)</li>
<li>3.  Using ArcSde for Postgres storing data as PostGis geometries using the OGC Simple Feature model (PostGis-centric hybrid view)</li>
<li>4.  Using PostGis on Postgres with zigGIS (PostGis-centric view)</li>
</ul>

(3) this is the combination that Bill is currently showing in his blog.

I will go now more in details on what you get and what you loose by chocing each of this combination. Note that I am going in an Esri-centric to PostGis-centric order.
Note once more that what i am writing here is based on my understanding, as far as i could not try ArcSde for Postgres: so if some of you has some correction to show, drop a note in the comments please.

<strong>1.  Using ArcSde for Postgres storing data as Esri geometries (Esri-centric with Postgres view)</strong>
Go with this if you:
<ul>
<li>need Geodatabase goodness (domains, subtypes, networks, etc...)</li>
<li>want store raster in the Geodatabase</li>
<li>want use versions, (long transactions)</li>
<li>want to read and edit the Geodatabase only using Esri software like ArcGis Desktop (with a minimum of an ArcEditor license if you need to edit data)</li>
<li>want to serve GIS data on the web only by ArcGis server (until someone will write the 9.3 bindings for MapServer, GeoServer and so on)</li>
<li>don't need the powerful PostGis SQL Spatial Functions and you are going to do everything by Esri clients and ArcObjects (though maybe Esri is going to provide its own implementation, like they recently did for Oracle and SQL Server â€“ not sure about this)</li>
</ul>



<strong>2.  Using ArcSde for Postgres storing data as PostGis geometries using the Esri Geodatabase model (Esri-centric hybrid view)</strong>
Go with this if you:
<ul>
<li>need Geodatabase goodness (domains, subtypes, networks, etc...)</li>
<li>want store raster in the Geodatabase</li>
<li>want use versions (long transactions)</li>
<li>want to be able to read data with major Open Source clients (QGIS, uDig, gvSIG, MapServer, GeoServer, FeatureServer and many others). You will need at least an ArcView license for reading data with Geodatabase goodness (ie: to navigate relationship class)</li>
<li>want to edit with an ArcEditor license (with the other clients you would surely compromise the Geodatabase structure)</li>
<li>want to serve GIS data on the web by ArcGis server or by any important OS map server (MapServer, GeoServer and so on)</li>
<li>want to use PostGis SQL Spatial Functions (but without doing INSERT, UPDATE and DELETE operations otherwise you would still destroy the Geodatabase structure)</li>
</ul>

<strong>3.  Using ArcSde for Postgres storing data as PostGis geometries using the OGC Simple Feature model (PostGis-centric hybrid view)</strong>
Go with this if you:
<ul>
<li>dont' need Geodatabase goodness (domains, subtypes, networks, etc...)</li>
<li>want store raster in the Geodatabase</li>
<li>want use versions (long transactions)</li>
<li>want to be able to read and write data with major Open Source clients, if not using versions (QGIS, uDig, gvSIG, MapServer, GeoServer, FeatureServer and many others).</li>
<li>want to edit in ArcGis desktop with an ArcEditor license</li>
<li>want to serve GIS data on the web by ArcGis server or by any important OS map server (MapServer, GeoServer and so on)</li>
<li>want to use PostGis SQL Spatial Functions, if not using versions (including INSERT, UPDATE and DELETE operations)</li>
</ul>

<strong>4.  Using PostGis on Postgres with zigGIS (PostGis-centric view)</strong>
Go with this if you:
<ul>
<li>don't need Geodatabase goodness (domains, subtypes, networks, etc...)</li>
<li>don't want store raster in the Geodatabase</li>
<li>don't want use versions (long transactions)</li>
<li>want to be able to read and write data with major Open Source clients (QGIS, uDig, gvSIG, MapServer, GeoServer, FeatureServer and many others)</li>
<li>want to read and edit data with an ArcView license via zigGIS, saving the costs for ArcEditor and ArcSde licenses</li>
<li>want to serve GIS data by any important OS map server (MapServer, GeoServer and so on)</li>
<li>want to use PostGis SQL Spatial Functions</li>
</ul>

Now many points are obvious, but I need to clarify some of them, specially for PostGis users not knowing what you will get adopting the Esri GeoDatabase implementation.

<strong>Considerations if you are already a PostGis user</strong>

First: what you get if you use ArcSde?
You will be able to store your GIS data in an RDBMS like Oracle, SQL Server, Informix, DB2 and Postgres (from 9.3). Possibly Postgres, if you are reading this post ;-)

The geodatabase is able to contain three dataset types:
<ul>
<li>Feature classes</li>
<li>Raster datasets</li>
<li>Tables</li>
</ul>

Regarding feature classes, you may opt to use some of the Esri Geodatabase Model benefits, like:
<ul>
<li>Feature datasets</li>
<li>Subtypes</li>
<li>Attribute domains</li>
<li>Relationship classes</li>
<li>Topology</li>
<li>Network datasets</li>
<li>Geometric networks</li>
<li>Terrain (TIN)</li>
<li>Address locator</li>
<li>Linear referencing</li>
<li>Versioning</li>
</ul>

for more info you may look the <a href="http://webhelp.esri.com/arcgisdesktop/9.2/index.cfm?TopicName=Feature_class_basics">ArcGis documentation</a>.

Keep in mind that doing so you are highly tied to the Esri suite. In fact the geodatabase model is managed at server side level by ArcSde with a lot of metadata tables and stored procedures in the database, that keep track of all of this stuff, and at client side (for example from ArcMap) by the ArcObjects API.
The model is very powefull and gives the developer and even the analyst a lot of GIS out of the box benefits, but - being based on this Esri structure - you are then forced to use Esri software for accessing the data. For instance, if you have created a relationship class between two feature classes, you will need ArcMap or some other ArcObjects application to navigate the relationship or to edit it.

<strong>Considerations if you are already an ArcSde user</strong>
PostGIS will let you store data in the Postgres RDBMS. You can store Feature classes and Tables according to the OCG Simple Feature implementation.
There is still not the possibility to store raster, but my opinion is that rasters should be better stored in the filesystem than in a RDBMS (well, it is just my opinion â€“ of course). And â€“ as far as Open Source software evolves quickly, it is even possible that a solution for storing raster in PostGis will be soon available.

You obviously cannot use the GDB features: if you need them you have two options:
1)use ArcSde for PostGIS (only from 9.3, of course);
2)write the logic of your model by using Postgres objects or by coding it in your application (instead than in the data).

If you opt for the first solution, be sure to read this Bill Dollins post about it. It is a very good solution for taking in your organization other cool Open Source software, desktop clients like uDig, QGIS, and gvSIG (you will be able to effectively read and edit data with them), and outstanding (and free!!!) server products like MapServer, GeoServer, FeatureServer and SharpMap.

If you still would like to use PostGis from within ArcGis desktop but you can't afford the ArcSde and ArcEditor licenses, zigGis could be the way to go.
But if that is your choice and you miss the intelligent data (GDB), how can you obtain similiar GDB behaviours in PostGis?
Well, mostly is just a question to use some DDL, Postgres SQL and PostGis Spatial SQL. For example:
<ul>
<li>GDB Subtypes: Postgres Primary and Foreign Keys. and some triggers if you want to differentiate behaviour for each feature class subtype. But ArcMap won't display the different subtypes in the TOC, you may obtain this only by customization</li>
<li>GDB Attribute Domains: Postgres Check Constraints. But ArcMap won't show you the combobox in the edit data form. Again: you may obtain this only by customization</li>
<li>GDB Relationship Classes: Postgres Primary and Foreign Keys, with cascade behaviour. Trigger with spatial SQL for spatial relationship. ArcMap won't show the relationship, only if customized</li>
<li>GDB Topology: Postgres triggers with spatial SQL. The topologic dataset will be updated in ArcMap as soon as the triggers fire and the map is refreshed</li>
<li>GDB Network Datasets: Postgres triggers with spatial SQL. The network connectivity rules can be implemented from ArcCatalog with customization</li>
<li>GDB Geometric Network: Implementable like for network datasets</li>
</ul>

Fot this time is all, I hope i could clarify some points to you.

<em>Disclaimer: I still couldn't directly test ArcSde for Postgres 9.3 (and not sure I can ever be able to), and i am one of the zigGis developers</em>.











