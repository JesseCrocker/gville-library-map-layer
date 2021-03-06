### Why?
Go to Google Maps, zoom into Greenville, SC and type "bike racks" or "dog parks". If you're lucky you get decent info, but inevitably you'll be looking at incomplete or irrelavant results. You may even get advertisments.

This is not to say we should make a public-version of Google Map. Actually, we already have that in OpenStreetMaps.

Rather, lots of great public infomation is locked inside all maps, proprietary and open maps alike, in much the same way styling was locked inside of HTML before CSS.

The current lock-in approach reduces:
* accuracy of updates
* the speed of changes
* the scope of sharing
* the ability to mix and match layers looking for new patterns (what do we see if we overlay tree planting data with census data?)
* portability across applications (browser vs tablet app vs GIS tools)
* depending on the source, oversight and fairness


### What's the Problem with Current Sharing Methods?
Lets say you found park data on the city's GIS system and exported it out to a file. Now, you have the bright idea to build a Google Map with a layer for the parks, plus another layer for bathroom locations.

You share it with friends and 5 of them love the idea. They make their own map with your city parks + their own interests (bike racks, breweries, dog parks, etc).

Everyone goes about sharing and copying the layer data they like the most into their own personal maps.

Cool, we have 6 interesting maps.

One week later a dog park closes due to clown sightings. Spring comes and the city has a new park and 3 new breweries.  Oh, and all the bike rakes were moved next to parking decks.

Now what?  Well, things get stale and maps die, that's what.


### How We Solve the Problem
1. A Google Docs / Drive Spreadsheet is used as a real-time data source that virtually anybody can help maintain.
2. A PHP script reads the published Google spreadsheet in real-time and converts that into a [GeoJSON format](http://geojson.org/geojson-spec.html)
3. The spreadsheet is now a public, sharable, standardized GeoJSON URL (via the geojson.php) which anybody can reference or edit in real-time.
4. Any tool which understands GeoJSON, like [LeafletJS](http://leafletjs.com/), can point at one or more map layer URLs and magically show fresh data.


### Start With a New Google Spreadsheet
* Make a copy of the [base spreadsheet template](https://docs.google.com/spreadsheets/d/10eNXFh6mzFtii7B2PW90jmHtrQLJlRCrf3kkHU0HIH8/edit?usp=sharing) (File -> Make a Copy)
* Rename your copy and start adding real "point" data, including geographic coordinates (longitude, latitude) and other properties

### Publish Your Google Spreadsheet
* Go to (File -> Publish to Web) and do the following
* "Entire Document" (unless you only want to publish one tab, in which case select the tab)
* "Comma-seperated values (.csv)
* Click Publish
* Save the URL provided. It will look like https://docs.google.com/spreadsheets/d/**{key - bunch or random numbers and letters}**/pub?output=csv

Additions or changes to the spreadsheet will appear on a refreshed map somewhere between immediately and a few minutes later.

The "CSV data source URL" to be used in the geojson.php PHP script is slightly different and looks like
``https://docs.google.com/spreadsheet/pub?key={key - bunch or random numbers and letters}&single=true&gid={tab id}&output=csv``

If you want the first tab then use *gid=0* above. If you want to point at another tab then open that tab in you browser and look in the URL for *gid=##########* and use that *gid* value.

### Allowing Other People to Help Curate the Data

Use Google's *Share* function to give *Edit* permissions. You should consider allowing "Anyone with the link" to edit the data and then all you need to do is share the edit URL with trusted people.

### Using PHP to Convert a Google Sheets CSV to a GeoJSON File
Take the "CSV data source URL" you just constructed and insert it in the $googleSpreadsheetUrl variable near the top of geojson.php

The column values are used in this example to generate the GeoJSON are hard-coded to include 4 fields: longitude, latitude, title, and notes.
If you need more fields you can add additional *properties* in your geojson.php under the $features['properties'] array section.

### Getting Longitude and Latitude

Non-programming / manual ways to get latitude and longitude numbers
* (Easiest) Go to MapQuest, zoom in and center the position you want in the middle of the map and right click on the spot you want. The pop-up will show the lat and long.
* (OK) In Google Maps zoom into a point. The URL in your browser will contain the center point's latitude and longitude (in that order) ex: 34.8509174,-82.3987371
* (Hard, but good for looking up lots of data) If you know how to do custom functions in Google Sheets then you can [convert an address into latitude and longitude](https://ctrlq.org/code/19992-google-maps-functions-for-google-script) with some customization.

### Render a Leaflet Map Showing the GeoJSON Data
The index.html file loads the GeoJSON file into a local Javascript variable. Point this at your GeoJSON file and Leaflet will 
render the GeoJSON data. The magic line for loading a single layer map is 

var geoJsonData = JSON.parse(readJSON('geojson.php'));

Leaflet JS is using open MapQuest tiles. [As of July 2016](http://devblog.mapquest.com/2016/06/15/modernization-of-mapquest-results-in-changes-to-open-tile-access/),
it's necessary to [register an account with MapQuest. Up to 15,000 views a month is free](https://developer.mapquest.com/plans).

### Registering for Map Tiles

The base project includes a MapQuest key but ideally you should register your own free account and use your own key
to avoid exhausting this base key if too many people start using or abusing it.

If using Mapquest then after registering for an account do the following:
* [Login to your developer account](https://developer.mapquest.com/user/login)
* Go to Keys and Reporting
* Register a new Application. Give it a name. The callback may not be necessary. Save the application
* Click on the "Application" you created and copy the "Consumer Key", NOT the secret.
* You'll use that key value in the index.html Sin the MapQuest ``<script>`` tag where it says =PASTEYOURMAPQUESTKEYHERE


If you register with CloudMade you can use their map tiles instead, as described by Leaflet 
example http://leafletjs.com/examples/geojson.html

### Testing in Cloud 9
* After [forking this base project and setup a C9 workspace](https://github.com/codeforgreenville/leaflet-google-sheets-template/blob/master/SC-CODES-README.md)
* Save your changes (File-> Save or Ctrl + S)
* Click Run Project and it will fire up an Apache web service running your environment
* On the command line at the bottom of C9 you'll see a message like ``Starting Apache httpd, serving https://leaflet-wi-fi-map-allella.c9users.io/`` (your public workspace URL)
* Visit your public workspace URL in a new browser tab and you should see your map with the points specified in the spreadsheet

### Multiple Layers and Advanced Leaflet
[Leaflet has a bunch of other plug-ins and options](http://leafletjs.com/reference-1.0.0.html), so the maps can be tweaked in all sorts of ways.

[Documentation for MapQuest and Leaflet](https://developer.mapquest.com/documentation/leaflet-plugins/maps/)

---
Original concept from [OpenData Day 2014 in Greenville SC](https://github.com/OpenUpstate/OpenDataDay2014)
