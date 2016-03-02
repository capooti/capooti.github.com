---
categories: Uncategorized, GIS, Tutorials, Python, OpenLayers, Django, geopy, jQuery
date: 2010/09/17 21:00:27
guid: http://www.paolocorti.net/?p=297
permalink: http://www.paolocorti.net/2010/09/17/geocoding-in-web-applications-openlayers-and-geopy-to-the-rescue/
tags: Uncategorized, GIS, Tutorials, Python, OpenLayers, Django, geopy, jQuery
title: 'Geocoding in web applications: OpenLayers and geoPy to the rescue!'
---
In many situations, in your web applications, you will need a feature for geocoding an address, a city, a country...
A possible approach is to use the Javascript API of the main geocoding services (Google Maps, Yahoo! Maps, GeoNames...).
But as I have showed in a <a href="http://www.paolocorti.net/2009/10/14/geocoding-with-geopy/">previous post</a>, if you are using Python, there is an excellent API that will take care of this, without Javascript headaches: geoPy.

In this post I will show how to use geoPy, OpenLayers and a bit of jQuery to assemble a simple but nice tool for geocoding within OpenLayers.
In my server code I will use Django but you may opt for any Python web framework (Pylons, Zope, TurboGears, web2py...) with very slight modifications to the code.
You could even adapt the Javascript code of this sample (a mix of OpenLayers and jQuery) for using it without geoPy but with a different API (server or Javascript).

Basically this is what the tool will do: typing an address or place in a jQuery dialog, the dialog will be filled with results from three major gecoding services (Google, Yahoo! and GeoNames). Each result will be a link, and clicking on it there will be a zoom at that place in the OpenLayers map.

<img src="http://www.paolocorti.net/wp-content/uploads/2010/09/olandgeopy.png" alt="OpenLayers and geoPy to the rescue!" title="olandgeopy" width="400"  />

As suggested we have a mix of client (Javascript) and server (Django) code.
The idea is to expose a Django view accepting a string parameter (the place name to geocode) giving an xml output like this one:

$$code(lang=xml)
<?xml version="1.0" ?>
<geoservices>
  <service name="google">
    <location lat="42.2057962" lon="13.5203333" name="67048 Rocca di Mezzo L'Aquila, Italy"/>
  </service>
  <service name="yahoo">
    <location lat="42.204762" lon="13.51869" name="Rocca di Mezzo AQ, Italy, IT"/>
    <location lat="41.96497" lon="13.01067" name="00020 Rocca di Mezzo RM, Italy, IT"/>
  </service>
  <service name="geonames">
    <location lat="42.2058" lon="13.52033" name="Rocca Di Mezzo, IT 67048"/>
    <location lat="42.17704" lon="13.51706" name="Rovere Di Rocca Di Mezzo, IT 67048"/>
  </service>
</geoservices>
$$/code

This is the Django view implementing this functionality:

$$code(lang=python)
from django.shortcuts import render_to_response
from django.http import HttpResponse
from django.template import RequestContext
from olgeopy import settings
from xml.dom.minidom import Document
from geopy import geocoders

def geolocate(request):
    """
    Geolocate an address returning an xml response.
    """
    # get address from querystring
    address = ''
    if 'address' in request.GET:
        address = request.GET.get('address', '')
    # Create the minidom document
    doc = Document()
    # Create the <geoservices> base element
    geoservices = doc.createElement("geoservices")
    doc.appendChild(geoservices)
    try:
        # google
        g = geocoders.Google(settings.GOOGLE_API_KEY)
        google_service = get_service_element(doc, 'google', g, address)
        geoservices.appendChild(google_service)
        # yahoo
        y = geocoders.Yahoo(settings.YAHOO_APP_ID)
        yahoo_service = get_service_element(doc, 'yahoo', y, address)
        geoservices.appendChild(yahoo_service)
        # geonames
        n = geocoders.GeoNames()
        geonames_service = get_service_element(doc, 'geonames', n, address)
        geoservices.appendChild(geonames_service)
    except:
        print 'Error...'
    resp = HttpResponse(doc.toprettyxml(indent="  "), mimetype='text/xml')
    return resp

def get_service_element(doc, service_name, geocoder, address):
    """
    Generate an xml element for a geolocation service.
    """
    # Create the main service element
    service = doc.createElement("service")
    service.setAttribute("name", service_name)
    # Get places list from geocoder
    places = geocoder.geocode(address, exactly_one=False)
    # Create the location elements for each service
    while True:
        try:
            place = places.next()
            print(place)
            location = doc.createElement("location")
            location.setAttribute('lon', '%s' % place[1][1])
            location.setAttribute('lat', '%s' % place[1][0])
            location.setAttribute('name', place[0])
            service.appendChild(location)
        except StopIteration:
            break
    return service
$$/code

Basically the geolocate view checks if in the request there is the address string to geocode, and then by using the minidom API it creates the output xml.
Note that each location node is built by querying the geocoding service with a geoPy's geocoder object.

On the client side, there will be an html page with an OpenLayers map in it, and the jQuery dialog needed from the application.
This is an extract of it:

{% highlight xml %}
<!-- search -->
<div id="search-dialog" title="Find a location">
</div>
<a href="#" onclick="setResources(map,"{ url geolocate-addresses }");jQuery('#search-dialog').dialog('open'); return false">Find a location</a>

<!-- Map -->
<div id="map-id"></div>
{% endhighlight %}

At this point, clicking on the search link, the dialog will open, linked to the map, and the Django view will be requested to get the necessary xml.

This is the Javascript code doing the trick:

{% highlight javascript %}
var map;
var urlxml;
var xmlreq;

function getLocations(searchtext)
{
    //search only at least if there are 2 chars
    if(searchtext.length>1){
        if (xmlreq != undefined) {
            xmlreq.abort();
        };
        $("#results").html('<br /><div id="progress"><p>Geocoding ' + searchtext + '...</p></div>');
        outxml = urlxml + searchtext;
        //read xml
        xmlreq = $.ajax({
            type: "GET",
            data: '',
            url: outxml,
            dataType: "xml",
            success: parseXml
        });
    }
    else{
        $("#results").html('No results.');
    }
}

function setResources(ol_map,geocoded_xml){
    map = ol_map;
    urlxml = geocoded_xml + '?address=';
}

function setMapCenter(lon, lat)
{
    var proj = new OpenLayers.Projection("EPSG:4326");
    var point = new OpenLayers.LonLat(lon, lat);
    map.setCenter(point.transform(proj, map.getProjectionObject()));
}

function parseXml(xml)
{
    $("#results").html('<br />');
    var searchtext = $("#txtAddress").val();
    //for each service (Google, Yahoo, Geonames...)
    $(xml).find("service").each(function()
    {
        var service_name = $(this).attr("name");
        $("#results").append('<strong>Results for "' + searchtext + '" from ' + service_name + "</strong>:<br />");
        //for each location result in that service
        $(this).find("location").each(function()
        {
            var loc_name = $(this).attr("name");
            var lon = $(this).attr("lon");
            var lat = $(this).attr("lat");
            html_location = '<a href="" OnClick="javascript:setMapCenter(' + lon + ',' + lat + ');return false;">' + loc_name + '</a>';
            $("#results").append(html_location + "<br />");
        });
        $("#results").append('<br />');
    });
};

$.fx.speeds._default = 700;
$(function() {
    var search_form = " \
        <div class='search-id'> \
            <div id='dialog' title='Geolocated results'> \
                <form> \
                    <input id='txtAddress' type='text' name='txtAddress' /> \
                </form> \
            </div> \
            <div id='results'></div> \
        </div> \
        <script> \
            $('#txtAddress').keyup(function(e) { \
                getLocations($('#txtAddress').val()); \
            }); \
        </script>"
    // dialog
    $("#search-dialog").dialog({
        autoOpen: false,
        autoClose: true,
		show: 'blind',
		hide: 'explode',
		height: 300,
		width: 400,
		zIndex: 4000
    });
    $('#search-dialog').html(search_form);
  });
{% endhighlight %}

Basically, when the search-dialog opens and the user types a search string in the txtAddress input, the getLocations method is invoked.
This method will send an AJAX xml request to the Django view and, in case of success, will parse the xml Django response to compose the geolink list in the dialog.

The complete source code is <a href='http://www.paolocorti.net/wp-content/uploads/2010/09/olgeopy.zip'>here</a>, have fun!
