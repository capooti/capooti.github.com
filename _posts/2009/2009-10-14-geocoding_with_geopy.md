---
layout: post
title: "Geocoding with GeoPy"
description: "Geocoding with GeoPy"
category:
tags: [Uncategorized, GIS, devs, Python, geopy, google, yahoo, geonames]
---
{% include JB/setup %}

There are now may geocoding web services out there, like <a href="http://code.google.com/apis/maps/documentation/index.html#Geocoding_HTTP_Request">Google geocoder</a>, <a href="http://developer.yahoo.com/maps/rest/V1/geocode.html">Yahoo geocoder</a>,  <a href="http://geocoder.us/">geocoder.us</a> (only for US address), <a href="http://msdn.microsoft.com/en-us/library/cc983790.aspx">Microsoft MapPoint</a>, <a href="http://www.geonames.org/">GeoNames</a> and <a href="http://www.mediawiki.org/wiki/MediaWiki">MediaWiki</a>.

Normally you may access this web services API directly with HTTP <del datetime="2009-10-16T07:27:16+00:00">REST</del> request, and get a response in common formats like xml, json, kml.
For example you may geocode with the Google geocoder an address like this one: "1071 5th Avenue, New York, NY" with a request like this one:

<a href="http://maps.google.com/maps/geo?q=1071+5th+Avenue,+New+York,+NY&output=json&oe=utf8&sensor=true_or_false=&key=your_apy_key">http://maps.google.com/maps/geo?q=1071+5th+Avenue,+New+York,+NY&output=json&oe=utf8&sensor=true_or_false=&key=your_apy_key</a>

In this case we are querying the Google geocoder web service to get a response in json via the output parameter in the query string. This is the json response we get back from the web service:

{% highlight json %}
{
  "name": "1071 5th Avenue, New York, NY",
  "Status": {
    "code": 200,
    "request": "geocode"
  },
  "Placemark": [ {
    "id": "p1",
    "address": "1071 5th Ave, New York, NY 10128, USA",
    "AddressDetails": {
   "Accuracy" : 8,
   "Country" : {
      "AdministrativeArea" : {
         "AdministrativeAreaName" : "NY",
         "SubAdministrativeArea" : {
            "Locality" : {
               "DependentLocality" : {
                  "DependentLocalityName" : "Manhattan",
                  "PostalCode" : {
                     "PostalCodeNumber" : "10128"
                  },
                  "Thoroughfare" : {
                     "ThoroughfareName" : "1071 5th Ave"
                  }
               },
               "LocalityName" : "New York"
            },
            "SubAdministrativeAreaName" : "New York"
         }
      },
      "CountryName" : "USA",
      "CountryNameCode" : "US"
   }
},
    "ExtendedData": {
      "LatLonBox": {
        "north": 40.7860701,
        "south": 40.7797749,
        "east": -73.9563496,
        "west": -73.9626448
      }
    },
    "Point": {
      "coordinates": [ -73.9588670, 40.7829790, 0 ]
    }
  } ]
}
{% endhighlight %}

Note that the response includes the main result (the coordinates of the geocoded point) and other important attributes, like - in the case of Google - the administrative area name, the sub administrative area name, the country name and so on.

The other web services works in a manner very similiar to Google. For example this is how the same address is geocoded by GeoNames (we will use the name place here - GeoNames is a database of geographic names: "Solomon Guggenheim Museum, NY")
If we make this request to the web services:
<a href="http://ws.geonames.org/search?q=Solomon+Guggenheim+Museum+NY">http://ws.geonames.org/search?q=Solomon+Guggenheim+Museum+NY</a>

we get this response:

{% highlight xml %}
<geonames style="MEDIUM">
<totalResultsCount>1</totalResultsCount>
<geoname>
<name>Solomon R Guggenheim Museum</name>
<lat>40.782879</lat>
<lng>-73.959027</lng>
<geonameId>5119572</geonameId>
<countryCode>US</countryCode>
<countryName>United States</countryName>
<fcl>S</fcl>
<fcode>BLDG</fcode>
</geoname>
</geonames>
{% endhighlight %}

This is the same request made to the Yahoo Geocoding Web Service: <a href="http://local.yahooapis.com/MapsService/V1/geocode?appid=your_app_id&street=1071+Fifth+Avenue&city=NY&state=NY">http://local.yahooapis.com/MapsService/V1/geocode?appid=your_app_id&street=1071+Fifth+Avenue&city=NY&state=NY</a>

And this is the result (only xml and Serialized PHP are available for Yahoo):

{% highlight xml %}
<ResultSet xsi:schemaLocation="urn:yahoo:maps http://api.local.yahoo.com/MapsService/V1/GeocodeResponse.xsd">
<Result precision="address">
<Latitude>40.782976</Latitude>
<Longitude>-73.959358</Longitude>
<Address>1071 5th Ave</Address>
<City>New York</City>
<State>NY</State>
<Zip>10128-0112</Zip>
<Country>US</Country>
</Result>
</ResultSet>
{% endhighlight %}

Google and GeoNames have also the possibility to reverse geocoding an address (to find an associated textual location such as a street address from geographic coordinates).
This is how to do the reverse geocoding request with Google (note that now we pass in the querystring the coordinates of the point and not the address):
<a href="http://maps.google.com/maps/geo?q=40.7829790,-73.9588670&output=json&oe=utf8&sensor=true_or_false&key=your_api_key">http://maps.google.com/maps/geo?q=40.7829790,-73.9588670&output=json&oe=utf8&sensor=true_or_false&key=your_api_key</a>

And this is the result (we still have a json output - note that there in this case are many results!):

{% highlight json %}
{
  "name": "40.7829790,-73.9588670",
  "Status": {
    "code": 200,
    "request": "geocode"
  },
  "Placemark": [ {
    "id": "p1",
    "address": "1070 5th Ave, New York, NY 10128, USA",
    "AddressDetails": {
   "Accuracy" : 8,
   "Country" : {
      "AdministrativeArea" : {
         "AdministrativeAreaName" : "NY",
         "SubAdministrativeArea" : {
            "Locality" : {
               "DependentLocality" : {
                  "DependentLocalityName" : "Manhattan",
                  "PostalCode" : {
                     "PostalCodeNumber" : "10128"
                  },
                  "Thoroughfare" : {
                     "ThoroughfareName" : "1070 5th Ave"
                  }
               },
               "LocalityName" : "New York"
            },
            "SubAdministrativeAreaName" : "New York"
         }
      },
      "CountryName" : "USA",
      "CountryNameCode" : "US"
   }
},
    "ExtendedData": {
      "LatLonBox": {
        "north": 40.7861576,
        "south": 40.7798624,
        "east": -73.9557594,
        "west": -73.9620546
      }
    },
    "Point": {
      "coordinates": [ -73.9589070, 40.7830100, 0 ]
    }
  }, {
    "id": "p2",
    "address": "Solomon R Guggenheim Museum, New York, NY 10128, USA",
    "AddressDetails": {
   "Accuracy" : 9,
   "Country" : {
      "AdministrativeArea" : {
         "AdministrativeAreaName" : "NY",
         "SubAdministrativeArea" : {
            "Locality" : {
               "DependentLocality" : {
                  "AddressLine" : [ "Solomon R Guggenheim Museum" ],
                  "DependentLocalityName" : "Manhattan",
                  "PostalCode" : {
                     "PostalCodeNumber" : "10128"
                  }
               },
               "LocalityName" : "New York"
            },
            "SubAdministrativeAreaName" : "New York"
         }
      },
      "CountryName" : "USA",
      "CountryNameCode" : "US"
   }
},
    "ExtendedData": {
      "LatLonBox": {
        "north": 40.7860266,
        "south": 40.7797314,
        "east": -73.9558794,
        "west": -73.9621746
      }
    },
    "Point": {
      "coordinates": [ -73.9590270, 40.7828790, 0 ]
    }
  },
.... other results
{% endhighlight %}

If you need to invoke these web services in your code and you are using a Python, there is an excellent API that let you query all these web services in the same manner without messing your code with xml/json request/response to them: this API is called GeoPy.
Let's see how it is easy to work with it. First I create a virtualenv in my Ubuntu 9.04 with Python 2.5 and geopy 0.93. GeoPy at this time doesn't support Python > 2.5, and I will use the GeoPy from a branch, because that is the only one that at this time supporting reverse geocoding. If you do not need reverse geocoding you may use the stable version.

{% highlight bash %}
paolo@paolo-laptop:~/training$ virtualenv --python=python2.5 geopy
Running virtualenv with interpreter /home/paolo/training/reverse_geocoding/bin/python2.5
Using real prefix '/usr'
New python executable in geopy/bin/python2.5
Also creating executable in geopy/bin/python
Installing setuptools............done.
paolo@paolo-laptop:~/training$ cd geopy
paolo@paolo-laptop:~/training/geopy$ source bin/activate
(geopy)paolo@paolo-laptop:~/training/geopy$ easy_install http://geopy.googlecode.com/svn/branches/reverse-geocode
Downloading http://geopy.googlecode.com/svn/branches/reverse-geocode
Doing subversion checkout from http://geopy.googlecode.com/svn/branches/reverse-geocode to /tmp/easy_install-JVmlqN/reverse-geocode
Processing reverse-geocode
Running setup.py -q bdist_egg --dist-dir /tmp/easy_install-JVmlqN/reverse-geocode/egg-dist-tmp-HaqZ4c
zip_safe flag not set; analyzing archive contents...
Adding geopy 0.93dev-r84 to easy-install.pth file
Installed /home/paolo/training/geopy/lib/python2.5/site-packages/geopy-0.93dev_r84-py2.5.egg
Processing dependencies for geopy==0.93dev-r84
Finished processing dependencies for geopy==0.93dev-r84
{% endhighlight %}

GeoPy needs SimpleJson and BeautifulSoup, so we install them in the virtualenv:

{% highlight bash %}
(geocoding)paolo@paolo-laptop:~/training/geocoding$ easy_install simplejson
(geocoding)paolo@paolo-laptop:~/training/geocoding$ easy_install BeautifulSoup
{% endhighlight %}

Now let's activate the virtualenv againg and time to test this API.
I will geocode the address with Google and after I get the point coordinates I will reverse geocode that point:

{% highlight python %}
>>>from geopy import geocoders
>>>g = geocoders.Google('your_api_key')
>>>(place, point) = g.geocode('1071 5th Avenue, New York, NY')
Fetching http://maps.google.com/maps/geo?q=1071+5th+Avenue%2C+New+York%2C+NY&output=kml&key=your_api_key...
>>>print place, point
1071 5th Ave, New York, NY 10128, USA (40.782978999999997, -73.958866999999998)
>>>(new_place, new_point) = g.reverse(point)
Fetching http://maps.google.com/maps/geo?q=40.782979%2C-73.958867&output=kml&key=your_api_key...
>>>print new_place, new_point
1070 5th Ave, New York, NY 10128, USA (40.783009999999997, -73.958906999999996)
{% endhighlight %}

Using the API with different web services is just very similiar, there is a geocoder class for each web service: this is how to interact with Yahoo:

{% highlight python %}
>>>from geopy import geocoders
>>>y = geocoders.Yahoo('your_app_id')
>>>(place, point) = y.geocode('1071 5th Avenue, New York, NY')
Fetching http://api.local.yahoo.com/MapsService/V1/geocode?output=xml&location=1071+5th+Avenue%2C+New+York%2C+NY&appid=your_app_id...
>>>print place, point
1071 5th Ave, New York, NY 10128-0112, US (40.782975999999998, -73.959357999999995)
{% endhighlight %}

You will find more samples at the <a href="http://code.google.com/p/geopy/w/list">GeoPy website</a>.

One thing that is useful with these Geocoding Web Services is the possibility to get the accuracy of the geocoded address: Google calls this <a href="http://code.google.com/apis/maps/documentation/reference.html#GGeoAddressAccuracy">"Accuracy"</a>, Yahoo calls this <a href="http://developer.yahoo.com/maps/rest/V1/geocode.html">"Precision"</a>. You have seen the value of this parameter in the answers from the web services. Unluckily this value is still not accessible from GeoPy, but looks like <a href="http://code.google.com/p/geopy/wiki/NovemberSprint">it will be soon implemented</a>.
